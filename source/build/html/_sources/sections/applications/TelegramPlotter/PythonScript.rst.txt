Python Script
*************

Once you created a Telegram Bot, you are all set up.

The script is designed for monitoring two databases on two different servers, one specifically for gdelt, and one for DIAS.

GDELT has to be monitored locally, and is represented as C0

DIAS can be locally or remotely (represented by C1), but should not be ran locally concurrently to any GDELT application.



Setup
=====

In order to be able to run the script, python3 must be installed.

Python packages
---------------

To install the python packages, one needs to run the following command in the command line:

.. code:: bash

    pip3 install python-telegram-bot Wand pexpect configparser

Code Setup
----------

In order to run the script, some infos must be entered into the config file "config.conf":

- **telegram_token:** the token of your telegram bot to use (See previous section). 
- **chat_id:** the chat id of your chat with the bot (see previous section).
- **ip:** the ip address of the remote server. If left empty, only the local server will be checked.
- **ssh_username:** the username to use when connecting to the remote server
- **ssh_password:** the password to use when connection to the remote server
- **run_c0:** wheter the monitoring for c0 should be run or not
- **run_c1:** wheter the monitoring for c1 shoule be run or not
- **hour:** the hour of the day the automatic check should fire
- **minute:** the minute of the hour the automatic check should fire
- **second:** the second of the minute the automatic check should fire
- **microsecond:** the microsecond of the second the automatic check should fire
- **interval_in_days:** in what interval (in days) the autamatic check should be repeated

You also need to enter some lines in the R scripts, located in the folders 'C1' and 'C0':

**read.and.plot.aggregates.and.true.C0.R**:

- Line 44: enter the postgres database username
- Line 45: enter the postgres database password

**read.and.plot.aggregates.and.true.C1.avg.R**:

- Line 7: enter the remote IP address (can be 127.0.0.1)
- Line 36: enter the postgres database username
- Line 37: enter the postgres database password

**read.and.plot.aggregates.and.true.C1.count.R**:

- Line 7: enter the remote IP address (can be 127.0.0.1)
- Line 36: enter the postgres database username
- Line 37: enter the postgres database password



Launching the script
====================

If you launch the script on a server, it is advised to wrap the whole setup around a unix or tmux screen.

.. code:: bash

    tmux new -t python-monitoring

Inside the tmux (or screen) session, launch the script by first traversing to the correct directory.
Once there, launch it by entering the following command:

.. code:: bash 

    python3 DocumentSender.py

That's it, you can escape the session by pressing ctrl+b, and then d (or crly+a then d with unix screen).

To reattach later, type in:

.. code:: bash

    tmux a -t python-monitoring