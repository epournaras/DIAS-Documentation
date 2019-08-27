Monitoring
**********

There are several options available for monitoring a DIAS experiment in real-time :

- SQL queries
- R-Studio plots
- Redash dashboards


SQL queries
-----------

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
--------------



Redash dashboards
-----------------
