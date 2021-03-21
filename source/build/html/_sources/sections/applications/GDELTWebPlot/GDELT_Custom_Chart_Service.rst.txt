Database Helper
***************

The file "GDELT_Custom_Chart_Service.php" returns a json string in the following format:

.. code-block:: json

    {
        "r1": result_list_of_query1,
        "r2": result_list_of_query2,
        "r3": result_list_of_query3
    }

The queries and their results are described below.

Query1
======

Query1 queries the entries of a special table called "gdelt_web_plot" and returns the following values with the speciefied keys as a list of jsons:

.. code-block:: json

    [
        {
            "seq_id"                  : sequence_id of entry                                 [INTEGER],
            "dt"                      : timestamp of entry                                   [TIMESTAMP WITHOUT TIMEZONE],
            "epoch"                   : epoch of experiment (usually in milliseconds)        [BIGINT],
            "true_sum_gdelt_events"   : The sum of all injected values                       [DOUBLE],
            "sum_selected_states"     : The externally calculated sum of all selected states [DOUBLE],
            "dias_sum_selected_states": The sum of all node estimations                      [DOUBLE],
            "peer1"                   : the actual aggregation of peer1                      [DOUBLE],
            "peer2"                   : the actual aggregation of peer2                      [DOUBLE],
            "peer3"                   : the actual aggregation of peer3                      [DOUBLE],
            "peer4"                   : the actual aggregation of peer4                      [DOUBLE],
            "peer5"                   : the actual aggregation of peer5                      [DOUBLE],
            "peer6"                   : the actual aggregation of peer6                      [DOUBLE],
            "peer7"                   : the actual aggregation of peer7                      [DOUBLE],
            "peer8"                   : the actual aggregation of peer8                      [DOUBLE],
            "peer9"                   : the actual aggregation of peer9                      [DOUBLE],
            "peer10"                  : the actual aggregation of peer10                     [DOUBLE],
            "peer11"                  : the actual aggregation of peer11                     [DOUBLE],
            "peer12"                  : the actual aggregation of peer12                     [DOUBLE],
            "peer13"                  : the actual aggregation of peer13                     [DOUBLE],
            "peer14"                  : the actual aggregation of peer14                     [DOUBLE],
            "peer15"                  : the actual aggregation of peer15                     [DOUBLE],
            "peer16"                  : the actual aggregation of peer16                     [DOUBLE],
            "peer17"                  : the actual aggregation of peer17                     [DOUBLE],
            "peer18"                  : the actual aggregation of peer18                     [DOUBLE],
            "peer19"                  : the actual aggregation of peer19                     [DOUBLE],
            "peer20"                  : the actual aggregation of peer20                     [DOUBLE],
            "peer21"                  : the actual aggregation of peer21                     [DOUBLE],
            "peer22"                  : the actual aggregation of peer22                     [DOUBLE],
            "peer23"                  : the actual aggregation of peer23                     [DOUBLE],
            "peer24"                  : the actual aggregation of peer24                     [DOUBLE],
            "peer25"                  : the actual aggregation of peer25                     [DOUBLE],
            "peer26"                  : the actual aggregation of peer26                     [DOUBLE],
            "peer27"                  : the actual aggregation of peer27                     [DOUBLE],
            "peer28"                  : the actual aggregation of peer28                     [DOUBLE],
        }, ...
    ]

Query2
======

Query2 queries the entries of the table vw_gdelt_map_peer_country. It returns a list with the mappings between the peers and the corresponding country as a two digit code (eg. AU for Austria)

The values are returned as a list of jsons in the following format. The countries should always stay the same, but the matching can differ:

.. code-block:: json

    [
        {
            "peer":1,                     [INTEGER]
            "actiongeo_countrycode":"AU", [STRING]
        },
        {
            "peer":2,                     [INTEGER]
            "actiongeo_countrycode":"BE", [STRING]
        },
        {
            "peer":3,                     [INTEGER]
            "actiongeo_countrycode":"BU", [STRING]
        },
        {
            "peer":4,                     [INTEGER]
            "actiongeo_countrycode":"CY", [STRING]
        },
        {
            "peer":5,                     [INTEGER]
            "actiongeo_countrycode":"EI", [STRING]
        },
        {
            "peer":6,                     [INTEGER]
            "actiongeo_countrycode":"EN", [STRING]
        },
        {
            "peer":7,                     [INTEGER]
            "actiongeo_countrycode":"EZ", [STRING]
        },
        {
            "peer":8,                     [INTEGER]
            "actiongeo_countrycode":"FI", [STRING]
        },
        {
            "peer":9,                     [INTEGER]
            "actiongeo_countrycode":"FR", [STRING]
        },
        {
            "peer":10,                    [INTEGER]
            "actiongeo_countrycode":"GM", [STRING]
        },
        {
            "peer":11,                    [INTEGER]
            "actiongeo_countrycode":"GR", [STRING]
        },
        {
            "peer":12,                    [INTEGER]
            "actiongeo_countrycode":"HR", [STRING]
        },
        {
            "peer":13,                    [INTEGER]
            "actiongeo_countrycode":"HU", [STRING]
        },
        {
            "peer":14,                    [INTEGER]
            "actiongeo_countrycode":"IT", [STRING]
        },
        {
            "peer":15,                    [INTEGER]
            "actiongeo_countrycode":"LG", [STRING]
        },
        {
            "peer":16,                    [INTEGER]
            "actiongeo_countrycode":"LH", [STRING]
        },
        {
            "peer":17,                    [INTEGER]
            "actiongeo_countrycode":"LO", [STRING]
        },
        {
            "peer":18,                    [INTEGER]
            "actiongeo_countrycode":"LU", [STRING]
        },
        {
            "peer":19,                    [INTEGER]
            "actiongeo_countrycode":"MT", [STRING]
        },
        {
            "peer":20,                    [INTEGER]
            "actiongeo_countrycode":"NL", [STRING]
        },
        {
            "peer":21,                    [INTEGER]
            "actiongeo_countrycode":"PL", [STRING]
        },
        {
            "peer":22,                    [INTEGER]
            "actiongeo_countrycode":"PO", [STRING]
        },
        {
            "peer":23,                    [INTEGER]
            "actiongeo_countrycode":"RO", [STRING]
        },
        {
            "peer":24,                    [INTEGER]
            "actiongeo_countrycode":"SI", [STRING]
        },
        {
            "peer":25,                    [INTEGER]
            "actiongeo_countrycode":"SP", [STRING]
        },
        {
            "peer":26,                    [INTEGER]
            "actiongeo_countrycode":"SW", [STRING]
        },
        {
            "peer":27,                    [INTEGER]
            "actiongeo_countrycode":"SZ", [STRING]
        },
        {
            "peer":28,                    [INTEGER]
            "actiongeo_countrycode":"UK", [STRING]
        }
    ]

Query3
======

Query3 queries the gdelt_web_plot_history for the earliest entry and returnst that in a json list in the following way (the date is only an example and can differ):

.. code-block:: json

    [
        {
            "min": "2019-01-20 10:40:37.758339", [TIMESTAMP WITHOUT TIMEZONE]
        }
    ]
