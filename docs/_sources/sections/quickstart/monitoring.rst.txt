##########
Monitoring
##########

There are several options available for monitoring a DIAS experiment in real-time :

- SQL queries
- R-Studio plots
- Redash dashboards


SQL queries
***********

For fast and detailed understanding of the aggregation outcomes, it is easiest to execute queries on the Postgres database.


**Show the last aggregation state per peer**

.. code:: sql

    WITH with_last_peer_seq_id AS
    (
        SELECT
            MAX(seq_id) AS last_peer_seq_id
            ,peer
        FROM
            aggregation
        GROUP BY
            peer
    )
    SELECT
        agg.*
    FROM

        aggregation agg
    INNER JOIN
        with_last_peer_seq_id last_records
        ON
        last_records.last_peer_seq_id = agg.seq_id
    ORDER BY
        peer ASC
    ;

**Show raw logs for a selected peer over a given epoch range**

.. code:: sql

    SELECT * FROM rawlog WHERE peer = 13 AND epoch BETWEEN (1551766218 - 1) AND (1551766218 + 10)ORDER BY dt, seq_id ASC;

**Using the event log we can observe the frequency of function calls**

.. code:: sql

    SELECT classname,func,key,COUNT(*) FROM eventlog GROUP BY classname,func,key ORDER BY classname,func,key ASC;

**Memory usage evolution for a selected peer**

.. code:: sql

    SELECT
        *
    FROM
        memlog
    WHERE
        peer = 1
    ORDER BY seq_id ASC;


There are many more examples here (:ref:`label_dias_logging_system`).


R-Studio plots
**************

One way to plot the logged data is R.
R can be launched with |Rstudio|, or from the |R_command| itself.

.. |Rstudio| raw:: html

   <a href="https://rstudio.com/" target="_blank">RStudio</a>

.. |R_command| raw:: html

  <a href="https://www.datacamp.com/community/tutorials/installing-R-windows-mac-ubuntu" target="_blank">command line</a>

When using Rstudio, one can use multiple different scripts, as the state and all variables are stored between two consecutive executions.
For this reason, the following files are split into different parts. Further down there are also examples for combined scripts for the commandline.

Example plot for GDELT
======================

This sequence of queries creates a plot for the last 50000 entries of the database.

It shows the following 3 data sets:

1. The true sum of GDELT Events. Those values are the ground truth and desired by the peers.
2. The sum of selected states. This is the actual sum of the selected state over all peers. If the peer behave correctly, the true sum should be equal to this.
3. The DIAS sum. This value is the average over all local aggregates of type SUM.

.. figure:: monitoring/ExampleEposPlot.png
  :width: 30%

  Example plot

Reading the inferred values of all peers from sql database
----------------------------------------------------------

This script reads the inferred individual aggregates of all peers from a (possibly remote) sql database.

.. code-block:: R

  #one of both lines need to be executed
  #db.host <- <remote ip address>
  #db.host <- 'localhost'

  db.schema <- 'dias'

  diasNetworkId <- 0

  source_table <- 'aggregation_event'

  stopifnot(sum(ls() == 'db.host'      ) == 1)
  stopifnot(sum(ls() == 'db.schema'    ) == 1)
  stopifnot(sum(ls() == 'diasNetworkId') == 1)
  stopifnot(sum(ls() == 'source_table' ) == 1)

  print(sprintf('db.host : %s', db.host))
  print(sprintf('diasNetworkId : %s', diasNetworkId))
  print(sprintf('source_table : %s', source_table))

  # include
  library(dplyr)         # install.packages('dplyPerr')
  library("RPostgreSQL") # install.packages("RPostgreSQL")

  # constants
  now <- Sys.time()

  # database settings
  db.port <- <port on which the database can be reached>
  db.user <- <sql username>
  db.pwd  <- <password for sql username>

  # dataset settings
  last.epoch <- -1 # no restriction on epoch

  # plot settings
  db.rows <- 50000 # number of (most recent) rows to retrieve from database (-1 for all rows)

  # create output dataframe
  df.dias.all <- data.frame()

  # loads the PostgreSQL driver
  psql.drv <- dbDriver("PostgreSQL")

  # creates a connection to the postgres database
  # note that "con" will be used later in each connection to the database
  db.con <- dbConnect(psql.drv, dbname = db.schema
                     ,host = db.host, port = db.port
                     ,user = db.user, password = db.pwd)

  # mod eag 2018-09-11 - allow user to set a limit
  sql <- ''
  if(last.epoch == -1) {
    if(db.rows == -1) {
      sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'ORDER BY seq_id ASC');
    }
    else {
      sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'AND seq_id >= (SELECT MAX(seq_id) FROM',source_table, ') -', db.rows - 1, 'ORDER BY seq_id ASC');
    }
  }else {
    sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'AND seq_id >= (SELECT MAX(seq_id) FROM',source_table, ' WHERE epoch <=', last.epoch,') -', db.rows - 1, 'ORDER BY seq_id ASC');
  }

  print(sql)

  print('reading data')
  df.dias.all <- dbGetQuery(db.con, sql)
  nrows <- nrow(df.dias.all)
  print(sprintf('#rows : %s', nrows))
  print(sprintf('epoch range : %s - %s', min(df.dias.all$epoch), max(df.dias.all$epoch)))
  print('completed')

  # important to disconnect as a maximum of 16 open connections
  dbDisconnect(db.con)

Reading the true sum from sql database
--------------------------------------

With the "true" sum, we mean the acutal sum, which should be calculated by the peers.

.. code-block:: R

  #one of both lines need to be executed
  #db.host <- <remote ip address>
  #db.host <- 'localhost'

  db.schema <- 'dias'

  diasNetworkId <- 0

  stopifnot(sum(ls() == 'db.host'      ) == 1)
  stopifnot(sum(ls() == 'db.schema'    ) == 1)
  stopifnot(sum(ls() == 'diasNetworkId') == 1)

  print(sprintf('db.host : %s', db.host))
  print(sprintf('diasNetworkId : %s', diasNetworkId))

  # include
  library(dplyr)         # install.packages('dplyr')
  library("RPostgreSQL") # install.packages("RPostgreSQL")

  # constants
  now <- Sys.time()

  # database settings
  db.port <- <port on which the database can be reached>
  db.user <- <sql username>
  db.pwd  <- <password for sql username>

  # plot settings
  db.rows <- 50000 # number of (most recent) rows to retrieve from database

  # create output dataframe
  df.true.gdelt.sum <- data.frame()

  # loads the PostgreSQL driver
  psql.drv <- dbDriver("PostgreSQL")

  # creates a connection to the postgres database
  # note that "con" will be used later in each connection to the database
  db.con <- dbConnect(psql.drv      , dbname = db.schema
                     ,host = db.host, port = db.port
                     ,user = db.user, password = db.pwd)

  # read sum of events for all peers
  sql <- 'SELECT epoch,SUM(eventcount) AS true_sum_events FROM gdeltv2c WHERE epoch is NOT NULL GROUP BY epoch ORDER BY epoch'


  print('reading data')
  df.true.gdelt.sum <- dbGetQuery(db.con, sql)
  nrows <- nrow(df.true.gdelt.sum)
  print(sprintf('#rows : %s', nrows))
  print('completed')

  # important to disconnect as a maximum of 16 open connections
  dbDisconnect(db.con)

Plotting the read data
----------------------

.. code-block:: R

  # verify source data frame exists
  stopifnot(sum(ls() == 'df.dias.all'      ) == 1)
  stopifnot(sum(ls() == 'df.true.gdelt.sum') == 1)

  # include
  library(dplyr) # install.packages('dplyr')
  library(tidyr) # spread

  # constants
  now    <- Sys.time()
  series <- 'Sum'

  # check there is data in the input data.frame
  nrows <- nrow(df.dias.all)
  print(sprintf('#rows : %s', nrows))
  stopifnot(nrows > 0 )

  # get data range
  dt.range     <- range(df.dias.all$dt)
  dt.range.str <- sprintf('%s to %s', dt.range[1], dt.range[2])
  print(sprintf('dt.range : %s', dt.range.str))

  # get peers in the sample
  peers     <- sort(unique(df.dias.all$peer))
  num.peers <- length(peers)
  print(sprintf('#num.peers : %s', num.peers))

  # align all measurements to the same grid, since some
  # peers leave the network and don't generate measurements when they have left
  # scaffolding: seq
  min.epoch  <- min(df.dias.all$epoch)
  max.epoch  <- max(df.dias.all$epoch)
  num.epochs <- max.epoch - min.epoch + 1

  # need to show complete epochs! therefore don't show the very last one
  # this still does assume that each peer has provided an update for MAX(epoch) - 1
  unique.epochs <- sort(unique(df.dias.all$epoch))

  len.scaffolding <- length(unique.epochs)
  print(sprintf('len.scaffolding : %s', len.scaffolding))

  df.scaffolding <- data.frame(epoch = unique.epochs, epoch.rebase = seq(1, len.scaffolding))

  # compute average value of each peer at each epoch
  # this is because in the event table, there can be many updates per epoch
  df.peers <-
    df.dias.all %>%
    filter (active == TRUE) %>%
    group_by(epoch, peer) %>%
    summarize(state = mean(state)
             ,avg = mean(avg)
             ,sum = mean(sum)
             ,cnt_obs = n()
             ) %>%
    arrange(epoch, peer)

  # prepare baseline
  df.baseline <- data.frame(epoch = unique.epochs
                           ,epoch.rebase = seq(1, len.scaffolding)
                           ,state = numeric(len.scaffolding)
                           ,sum = numeric(len.scaffolding)
                           ,cnt = numeric(len.scaffolding)
                           )

  # ylim: range for the y-axis
  # show exact range
  ylim <- range(df.peers$sum, na.rm = TRUE)

  # prepare output for website
  # ouptut table contains 1 column per peer; to facilitate the construction of the output data.frame
  # we created an unpivoted view first, and then unpivot(spread) before saving to database
  # edward | 2018-09-24

  # plot each peer, one at a time
  for(peer.id in peers) {
    # get observations for this peer only
    df.this.peer <-
      df.peers %>%
      filter(peer == peer.id)

    # align to scaffolding
    df.plot <-
      df.scaffolding %>%
      left_join(df.this.peer, by = 'epoch')

    if( peer.id == 1 ){
      plot(df.plot$epoch.rebase
          ,df.plot$sum
          ,xlab = paste( 'Epoch (rebased, starts at ', min.epoch,')', sep = '' )
          ,ylab = series
          ,ylim = ylim
          ,main = paste( paste(db.host,'|', 'network',diasNetworkId,'|',series,'over',num.peers,'peers'),paste('last',nrows,'of',db.rows,'rows'),dt.range.str, sep = '\n')
          ,pch = 16
          ,cex = 0.5
          ,col = 4
          )
    }
    else {
      points(df.plot$epoch.rebase
            ,df.plot$sum
            ,pch = 16
            ,cex = 0.5
            ,col = 4
            )
    }

    # update baseline
    # first, fill missing values
    df.this.peer.fill <- df.plot
    for(i in 1:len.scaffolding) {
      if(is.na(df.this.peer.fill$state[i])){
        prev_value = 0.0
        if( i >= 2 ) {
          prev_value = if_else( is.na(df.this.peer.fill$state[i-1]), 0.0, df.this.peer.fill$state[i-1])
        }
        df.this.peer.fill$state[i] = prev_value
      }

      if(is.na(df.this.peer.fill$sum[i])) {
        prev_value = 0.0
        if(i >= 2) {
          prev_value = if_else(is.na(df.this.peer.fill$sum[i-1]), 0.0, df.this.peer.fill$sum[i-1])
        }
        df.this.peer.fill$sum[i] = prev_value
      }
    }

    # update baseline
    df.baseline$state = df.baseline$state + df.this.peer.fill$state
    df.baseline$sum   = df.baseline$sum + df.this.peer.fill$sum
    df.baseline$cnt   = df.baseline$cnt + 1
  }

  # compte the true sum of raw values
  r <- 0
  print('computing true sum of raw values')
  for(epoch_loop in df.baseline$epoch) {
    r <- r + 1

    true.gdelt.sum <-
      df.true.gdelt.sum %>%
      filter(epoch <= epoch_loop) %>%
      arrange(desc(epoch)) %>%
      head(n=1) %>%
      select(true_sum_events) %>%
      as.numeric()

    # save
    df.baseline$true_sum_events[r] <- true.gdelt.sum
  }

  # -----------------------
  # plot true sum of events
  # -----------------------

  lines(df.baseline$epoch.rebase
       ,df.baseline$true_sum_events
       ,col = 'green'
       ,lwd = 1.5
  )

  # -------------------------------
  # plot the true sum of the states
  # -------------------------------

  lines(df.baseline$epoch.rebase
       ,df.baseline$state
       ,col = 'blue'
       ,lwd = 1.5
       )

  # ---------------------------------
  # plot the average aggregated value
  # ---------------------------------

  df.baseline$agg_sum <- df.baseline$sum / df.baseline$cnt
  lines(df.baseline$epoch.rebase
       ,df.baseline$agg_sum
       ,col = 'red'
       ,lwd = 1.5
       )

  # legend
  legend("bottomleft", c('True sum of GDELT events', 'Sum of selected states', 'DIAS sum'), col=c('green', 'blue', 'red'), lwd=1.5)

Combined plot for the command line
----------------------------------

.. code-block:: R

  #------------------------------------------------------------- step 1

  #one of both lines need to be executed
  #db.host <- <remote ip address>
  #db.host <- 'localhost'

  db.schema <- 'dias'

  diasNetworkId <- 0

  source_table <- 'aggregation_event'

  stopifnot(sum(ls() == 'db.host'      ) == 1)
  stopifnot(sum(ls() == 'db.schema'    ) == 1)
  stopifnot(sum(ls() == 'diasNetworkId') == 1)
  stopifnot(sum(ls() == 'source_table' ) == 1)

  print(sprintf('db.host : %s', db.host))
  print(sprintf('diasNetworkId : %s', diasNetworkId))
  print(sprintf('source_table : %s', source_table))

  # include
  library(dplyr) # install.packages( 'dplyPerr')
  library("RPostgreSQL")#install.packages("RPostgreSQL")

  # constants
  now <- Sys.time()

  # database settings
  db.port <- <port on which the database can be reached>
  db.user <- <sql username>
  db.pwd  <- <password for sql username>

  # dataset settings
  last.epoch <- -1 # no restriction on epoch

  # plot settings
  db.rows <- 50000 # number of (most recent) rows to retrieve from database (-1 for all rows)

  # create output dataframe
  df.dias.all <- data.frame()

  # loads the PostgreSQL driver
  psql.drv <- dbDriver("PostgreSQL")

  # creates a connection to the postgres database
  # note that "con" will be used later in each connection to the database
  db.con <- dbConnect(psql.drv, dbname = db.schema
                     ,host = db.host, port = db.port
                     ,user = db.user, password = db.pwd)

  # mod eag 2018-09-11 - allow user to set a limit
  sql <- ''
  if(last.epoch == -1) {
    if(db.rows == -1) {
      sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'ORDER BY seq_id ASC');
    }
    else {
      sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'AND seq_id >= (SELECT MAX(seq_id) FROM',source_table, ') -', db.rows - 1, 'ORDER BY seq_id ASC');
    }
  }else {
    sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'AND seq_id >= (SELECT MAX(seq_id) FROM',source_table, ' WHERE epoch <=', last.epoch,') -', db.rows - 1, 'ORDER BY seq_id ASC');
  }

  print(sql)

  print('reading data')
  df.dias.all <- dbGetQuery(db.con, sql)
  nrows <- nrow(df.dias.all)
  print(sprintf('#rows : %s', nrows))
  print(sprintf('epoch range : %s - %s', min(df.dias.all$epoch), max(df.dias.all$epoch)))
  print('completed')

  # important to disconnect as a maximum of 16 open connections
  dbDisconnect(db.con)


  #------------------------------------------------------------- step 2

  stopifnot(sum(ls() == 'db.host'      ) == 1)
  stopifnot(sum(ls() == 'db.schema'    ) == 1)
  stopifnot(sum(ls() == 'diasNetworkId') == 1)

  print(sprintf('db.host : %s', db.host))
  print(sprintf('diasNetworkId : %s', diasNetworkId))

  # include
  library(dplyr) # install.packages( 'dplyr')
  library("RPostgreSQL")#install.packages("RPostgreSQL")

  # constants
  now <- Sys.time()

  # database settings
  db.port <- <port on which the database can be reached>
  db.user <- <sql username>
  db.pwd  <- <password for sql username>

  # create output dataframe
  df.true.gdelt.sum <- data.frame()

  # loads the PostgreSQL driver
  psql.drv <- dbDriver("PostgreSQL")

  # creates a connection to the postgres database
  # note that "con" will be used later in each connection to the database
  db.con <- dbConnect(psql.drv      , dbname = db.schema
                     ,host = db.host, port = db.port
                     ,user = db.user, password = db.pwd)


  # read sum of events for all peers
  sql <- 'SELECT epoch,SUM(eventcount) AS true_sum_events FROM gdeltv2c WHERE epoch is NOT NULL GROUP BY epoch ORDER BY epoch'

  print('reading data')
  df.true.gdelt.sum <- dbGetQuery(db.con, sql)
  nrows <- nrow(df.true.gdelt.sum)
  print(sprintf('#rows : %s', nrows))
  print('completed')

  # important to disconnect as a maximum of 16 open connections
  dbDisconnect(db.con)

  #------------------------------------------------------------- step 3

  # verify source data frame exists
  stopifnot(sum(ls() == 'df.dias.all'      ) == 1)
  stopifnot(sum(ls() == 'df.true.gdelt.sum') == 1)

  # include
  library(dplyr) # install.packages('dplyr')
  library(tidyr) # spread

  # constants
  now    <- Sys.time()
  series <- 'Sum'

  # check there is data in the input data.frame
  nrows <- nrow(df.dias.all)
  print(sprintf('#rows : %s', nrows))
  stopifnot(nrows > 0 )

  # get data range
  dt.range     <- range(df.dias.all$dt)
  dt.range.str <- sprintf('%s to %s', dt.range[1], dt.range[2])
  print(sprintf('dt.range : %s', dt.range.str))

  # get peers in the sample
  peers     <- sort(unique(df.dias.all$peer))
  num.peers <- length(peers)
  print(sprintf('#num.peers : %s', num.peers))

  # align all measurements to the same grid, since some
  # peers leave the network and don't generate measurements when they have left
  # scaffolding: seq
  min.epoch  <- min(df.dias.all$epoch)
  max.epoch  <- max(df.dias.all$epoch)
  num.epochs <- max.epoch - min.epoch + 1

  # need to show complete epochs! therefore don't show the very last one
  # this still does assume that each peer has provided an update for MAX(epoch) - 1
  unique.epochs <- sort(unique(df.dias.all$epoch))

  len.scaffolding <- length(unique.epochs)
  print(sprintf('len.scaffolding : %s', len.scaffolding))

  df.scaffolding <- data.frame(epoch = unique.epochs, epoch.rebase = seq(1, len.scaffolding))

  # compute average value of each peer at each epoch
  # this is because in the event table, there can be many updates per epoch
  df.peers <-
    df.dias.all %>%
    filter (active == TRUE) %>%
    group_by(epoch, peer) %>%
    summarize(state = mean(state)
             ,avg = mean(avg)
             ,sum = mean(sum)
             ,cnt_obs = n()
             ) %>%
    arrange(epoch, peer)

  # prepare baseline
  df.baseline <- data.frame(epoch = unique.epochs
                           ,epoch.rebase = seq(1, len.scaffolding)
                           ,state = numeric(len.scaffolding)
                           ,sum = numeric(len.scaffolding)
                           ,cnt = numeric(len.scaffolding)
                           )

  # ylim: range for the y-axis
  # show exact range
  ylim <- range(df.peers$sum, na.rm = TRUE)

  # prepare output for website
  # ouptut table contains 1 column per peer; to facilitate the construction of the output data.frame
  # we created an unpivoted view first, and then unpivot(spread) before saving to database
  # edward | 2018-09-24

  # plot each peer, one at a time
  for(peer.id in peers) {
    # get observations for this peer only
    df.this.peer <-
      df.peers %>%
      filter(peer == peer.id)

    # align to scaffolding
    df.plot <-
      df.scaffolding %>%
      left_join(df.this.peer, by = 'epoch')

    if( peer.id == 1 ){
      plot(df.plot$epoch.rebase
          ,df.plot$sum
          ,xlab = paste( 'Epoch (rebased, starts at ', min.epoch,')', sep = '' )
          ,ylab = series
          ,ylim = ylim
          ,main = paste( paste(db.host,'|', 'network',diasNetworkId,'|',series,'over',num.peers,'peers'),paste('last',nrows,'of',db.rows,'rows'),dt.range.str, sep = '\n')
          ,pch = 16
          ,cex = 0.5
          ,col = 4
          )
    }
    else {
      points(df.plot$epoch.rebase
            ,df.plot$sum
            ,pch = 16
            ,cex = 0.5
            ,col = 4
            )
    }

    # update baseline
    # first, fill missing values
    df.this.peer.fill <- df.plot
    for(i in 1:len.scaffolding) {
      if(is.na(df.this.peer.fill$state[i])){
        prev_value = 0.0
        if( i >= 2 ) {
          prev_value = if_else( is.na(df.this.peer.fill$state[i-1]), 0.0, df.this.peer.fill$state[i-1])
        }
        df.this.peer.fill$state[i] = prev_value
      }

      if(is.na(df.this.peer.fill$sum[i])) {
        prev_value = 0.0
        if(i >= 2) {
          prev_value = if_else(is.na(df.this.peer.fill$sum[i-1]), 0.0, df.this.peer.fill$sum[i-1])
        }
        df.this.peer.fill$sum[i] = prev_value
      }
    }

    # update baseline
    df.baseline$state = df.baseline$state + df.this.peer.fill$state
    df.baseline$sum   = df.baseline$sum + df.this.peer.fill$sum
    df.baseline$cnt   = df.baseline$cnt + 1
  }

  # compte the true sum of raw values
  r <- 0
  print('computing true sum of raw values')
  for(epoch_loop in df.baseline$epoch) {
    r <- r + 1

    true.gdelt.sum <-
      df.true.gdelt.sum %>%
      filter(epoch <= epoch_loop) %>%
      arrange(desc(epoch)) %>%
      head(n=1) %>%
      select(true_sum_events) %>%
      as.numeric()

    # save
    df.baseline$true_sum_events[r] <- true.gdelt.sum
  }

  # -----------------------
  # plot true sum of events
  # -----------------------

  lines(df.baseline$epoch.rebase
       ,df.baseline$true_sum_events
       ,col = 'green'
       ,lwd = 1.5
  )

  # -------------------------------
  # plot the true sum of the states
  # -------------------------------

  lines(df.baseline$epoch.rebase
       ,df.baseline$state
       ,col = 'blue'
       ,lwd = 1.5
       )

  # ---------------------------------
  # plot the average aggregated value
  # ---------------------------------

  df.baseline$agg_sum <- df.baseline$sum / df.baseline$cnt
  lines(df.baseline$epoch.rebase
       ,df.baseline$agg_sum
       ,col = 'red'
       ,lwd = 1.5
       )

  # legend
  legend("bottomleft", c('True sum of GDELT events', 'Sum of selected states', 'DIAS sum'), col=c('green', 'blue', 'red'), lwd=1.5)


Example plots for General DIAS
==============================

The following queries generate two different plots for checking the liveness of the system.

1. A plot with the avg of the selected states of all peers. With this plot one can see if the peers are working and changing their values.
2. A plot with the count of how many peers are currently active and did thus not leave.

For both plots there are some assumptions, which have to be met, in order to get some sense out of the plots.

1. For the first plot, the local selected states need to be randomised in a deterministic set, which changes over time.
2. For the second plot, peers need to join/leave the network eventually and repeatedly.

.. figure:: monitoring/ExampleDIASPlotAvg.png
  :width: 30%

  Example plot for the average selected states with selected states adapting their values according to the time of day

.. figure:: monitoring/ExampleDIASPlotCount.png
  :width: 30%

  Example plot for the number of online peers in a constantly changing network with different intensity levels

Reading the DIAS node aggregates
--------------------------------

.. code-block:: R

  #one of both has to be executed
  #db.host <- <remote ip>
  #db.host <- 'localhost'

  db.schema <- 'dias'

  diasNetworkId <- 0

  source_table <- 'aggregation_event'

  stopifnot(sum(ls() == 'db.host'      ) == 1)
  stopifnot(sum(ls() == 'db.schema'    ) == 1)
  stopifnot(sum(ls() == 'diasNetworkId') == 1)
  stopifnot(sum(ls() == 'source_table' ) == 1)

  print(sprintf('db.host : %s', db.host))
  print(sprintf('diasNetworkId : %s', diasNetworkId))
  print(sprintf('source_table : %s', source_table))

  # include
  library(dplyr)         # install.packages( 'dplyPerr')
  library("RPostgreSQL") # install.packages("RPostgreSQL")

  # constants
  now <- Sys.time()

  # database settings
  db.port <- <port on which the database can be reached>
  db.user <- <sql username>
  db.pwd  <- <password for sql username>

  # dataset settings
  last.epoch <- -1 # no restriction on epoch

  # plot settings
  db.rows <- 50000 # number of (most recent) rows to retrieve from database (-1 for all rows)

  # create output dataframe
  df.dias.all <- data.frame()

  # loads the PostgreSQL driver
  psql.drv <- dbDriver("PostgreSQL")
  # creates a connection to the postgres database
  # note that "con" will be used later in each connection to the database
  db.con <- dbConnect(psql.drv, dbname = db.schema
                     ,host = db.host, port = db.port
                     ,user = db.user, password = db.pwd)

  # mod eag 2018-09-11 - allow user to set a limit
  sql <- ''
  if( last.epoch == -1 ) {
    if( db.rows == -1 ) {
      sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'ORDER BY seq_id ASC');
    }
    else {
      sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'AND seq_id >= (SELECT MAX(seq_id) FROM',source_table, ') -', db.rows - 1, 'ORDER BY seq_id ASC');
    }
  }
  else {
    sql <- paste('SELECT * FROM',source_table,'WHERE network =', diasNetworkId, 'AND seq_id >= (SELECT MAX(seq_id) FROM',source_table, ' WHERE epoch <=', last.epoch,') -', db.rows - 1, 'ORDER BY seq_id ASC');
  }

  print(sql)

  print('reading data')
  df.dias.all <- dbGetQuery(db.con, sql)
  nrows       <- nrow(df.dias.all)
  print(sprintf('#rows : %s', nrows))
  print(sprintf('epoch range : %s - %s', min(df.dias.all$epoch), max(df.dias.all$epoch)))
  print('completed')

  # important to disconnect as a maximum of 16 open connections
  dbDisconnect(db.con)


Plotting the average plot
-------------------------

.. code-block:: R

  # verify source data frame exists
  stopifnot(sum(ls() == 'df.dias.all') == 1)

  # include
  library(dplyr) # install.packages( 'dplyr')

  # constants
  now    <- Sys.time()
  series <- 'Average'

  # check there is data in the input data.frame
  nrows <- nrow(df.dias.all)
  print(sprintf( '#rows : %s', nrows))
  stopifnot(nrows > 0)

  # get data range
  dt.range     <- range(df.dias.all$dt)
  dt.range.str <- sprintf('%s to %s', dt.range[1], dt.range[2])
  print(sprintf('dt.range : %s', dt.range.str))

  # get peers in the sample
  peers <- sort(unique(df.dias.all$peer))
  num.peers <- length(peers)
  print(sprintf( '#num.peers : %s', num.peers))


  # align all measurements to the same grid, since some
  # peers leave the network and don't generate measurements when they have left
  # scaffolding: seq
  min.epoch  <- min( df.dias.all$epoch )
  max.epoch  <- max( df.dias.all$epoch )
  num.epochs <- max.epoch - min.epoch + 1

  # need to show complete epochs! therefore don't show the very last one
  # this still does assume that each peer has provided an update for MAX(epoch) - 1
  epoch.rebase   <- seq( 1, num.epochs - 2 )

  df.scaffolding <- data.frame(epoch = seq( min.epoch + 1, max.epoch - 1), epoch.rebase = epoch.rebase)


  # compute baseline; this is the true mean for each epoch
  # some epochs may not contain an observation for all peers, so the sum used as the baseline will be incorrect
  # simply ignore these rows
  # modified eag 2018-01-15
  df.dias.all %>%
    filter (active == TRUE) %>%
    group_by(epoch) %>%
    summarize( baseline = mean(state) , cnt_obs = n()) -> df.baseline

  # plot baseline
  df.scaffolding %>% left_join(df.baseline, by = 'epoch') -> df.scaffolding.baseline

  # ylim: range for the y-axis
  # show exact range
  ylim <- range(df.dias.all$avg, na.rm = TRUE)

  plot(df.scaffolding.baseline$epoch.rebase
      ,df.scaffolding.baseline$baseline
      ,xlab = paste( 'Epoch (rebased, starts at ', min.epoch,')', sep = '' )
      ,ylab = series
      ,ylim = ylim
      ,main = paste( paste(db.host,'|', 'network',diasNetworkId,'|',series,'over',num.peers,'peers'),paste('last',nrows,'of',db.rows,'rows'),dt.range.str, sep = '\n')
      )

  # add peers
  for(peer.id in peers) {
    df.dias.all %>% filter(peer == peer.id) -> df.this.peer
    df.scaffolding %>% left_join(df.this.peer, by = 'epoch') -> df.scaffolding.peer

    points(df.scaffolding.peer$epoch.rebase
          ,df.scaffolding.peer$avg
          ,pch = 16
          ,cex = 0.5
          ,col = 4
          )
  }

  # replot the baseline, this time using a line
  lines(df.scaffolding.baseline$epoch.rebase
       ,df.scaffolding.baseline$baseline
       ,col = 'red'
       ,lwd = 1.5
       )


  # legend
  legend("bottomleft", c('baseline'), col=c('red'), lwd=1.5)

Plotting the active peer count
------------------------------

.. code-block:: R

  # verify source data frame exists
  stopifnot(sum(ls() == 'df.dias.all') == 1)

  # include
  library(dplyr) # install.packages( 'dplyr')

  # constants
  now    <- Sys.time()
  series <- 'Count'

  # check there is data in the input data.frame
  nrows <- nrow(df.dias.all)
  print(sprintf('#rows : %s', nrows))
  stopifnot(nrows > 0)

  # get data range
  dt.range     <- range(df.dias.all$dt)
  dt.range.str <- sprintf('%s to %s', dt.range[1], dt.range[2])
  print(sprintf('dt.range : %s', dt.range.str))

  #View(df.dias.all)

  # get peers in the sample
  peers     <- sort(unique(df.dias.all$peer))
  num.peers <- length(peers)
  print(sprintf('#num.peers : %s', num.peers))


  # align all measurements to the same grid, since some
  # peers leave the network and don't generate measurements when they have left
  # scaffolding: seq
  min.epoch  <- min(df.dias.all$epoch)
  max.epoch  <- max(df.dias.all$epoch)
  num.epochs <- max.epoch - min.epoch + 1

  # need to show complete epochs! therefore don't show the very last one
  # this still does assume that each peer has provided an update for MAX(epoch) - 1
  epoch.rebase <- seq(1, num.epochs - 2)

  df.scaffolding <- data.frame(epoch = seq( min.epoch + 1, max.epoch - 1), epoch.rebase = epoch.rebase)



  # compute baseline; this is the true number for each epoch

  # some epochs may not contain an observation for all peers, so the sum used as the baseline will be incorrect
  # simply ignore these rows
  df.dias.all %>%
    group_by(epoch) %>%
    summarize( baseline = sum(active), cnt_obs = n() ) %>%
    select( epoch,baseline ) -> df.baseline


  # plot baseline
  df.scaffolding %>% left_join(df.baseline, by = 'epoch') -> df.scaffolding.baseline

  # ylim
  ylim <- range(df.dias.all$count, na.rm = TRUE)

  plot(df.scaffolding.baseline$epoch.rebase
      ,df.scaffolding.baseline$baseline
      ,xlab = paste( 'Epoch (rebased, starts at ', min.epoch,')', sep = '' )
      ,ylab = series
      ,ylim = ylim
      ,main = paste(  paste(db.host,'|', 'network',diasNetworkId,'|',series,'over',num.peers,'peers'),paste('last',nrows,'of',db.rows,'rows'),dt.range.str, sep = '\n')
      )

  # add peers
  for(peer.id in peers) {
    df.dias.all %>% filter(peer == peer.id) -> df.this.peer
    df.scaffolding %>% left_join(df.this.peer, by = 'epoch') -> df.scaffolding.peer

    points(df.scaffolding.peer$epoch.rebase
          ,df.scaffolding.peer$count
          ,pch = 16
          ,cex = 0.5
          ,col = 4
          )
  }

  # replot the baseline, this time using a line
  lines(df.scaffolding.baseline$epoch.rebase
       ,df.scaffolding.baseline$baseline
       ,col = 'red'
       ,lwd = 1.5
       )

  # legend
  legend("bottomleft", c('baseline'), col=c('red'), lwd=1.5)

Combined plotting
-----------------

The combination in this case is straight forward, as one simply has to combine the :ref:`Reading the DIAS node aggregates` query with either one of the plotting queries.

Notes
=====

Note that if the sql database is on a remote server, postgres needs to whitelist the requesting comupter's ip by editign the |pg_hba| file.

.. |pg_hba| raw:: html

   <a href="https://www.postgresql.org/docs/8.2/auth-pg-hba-conf.html" target="_blank">pg_hba.conf</a>

Everywhere, where there is the following annotation: <something>, the whole statement must be replaced by some dynamic value explained by the statements inside the brackets.

Redash dashboards
*****************
