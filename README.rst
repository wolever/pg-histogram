pg-histogram: PostgreSQL functions for generating text-based histograms.
========================================================================

``histogram.sql`` provides functions which make it easy to generate text-based
histograms from within PostgreSQL!

By David `@wolever`__ Wolever

__ https://twitter.com/wolever

For example::

    psql# WITH email_lengths AS (
       -#    SELECT length(email) AS length
       -#    FROM auth_user
       -# )
       -# SELECT * FROM show_histogram((SELECT histogram(length, 12, 32, 10) FROM email_lengths))
     bucket |  range  | count | overflow |                 bar                  |             cumbar             | cumsum |      cumpct       
    --------+---------+-------+----------+--------------------------------------+--------------------------------+--------+-------------------
          0 | [12,14) |    17 |       -4 | =======--                            | ==                             |     21 |             0.056
          1 | [14,16) |    83 |        0 | ==================================== | ========                       |    104 | 0.277333333333333
          2 | [16,18) |    18 |        0 | ========                             | ==========                     |    122 | 0.325333333333333
          3 | [18,20) |    34 |        0 | ===============                      | ============                   |    156 |             0.416
          4 | [20,22) |    46 |        0 | ====================                 | ================               |    202 | 0.538666666666667
          5 | [22,24) |    44 |        0 | ===================                  | ====================           |    246 |             0.656
          6 | [24,26) |    61 |        0 | ==========================           | =========================      |    307 | 0.818666666666667
          7 | [26,28) |    26 |        0 | ===========                          | ===========================    |    333 |             0.888
          8 | [28,30) |    13 |        0 | ======                               | ============================   |    346 | 0.922666666666667
          9 | [30,32) |    11 |       18 | =====++++++++                        | ============================== |    375 |                 1
    (10 rows)


Installation
------------

Install with::

    $ curl https://raw.githubusercontent.com/wolever/pg-histogram/master/histogram.sql | psql


Column Meanings
---------------

The ``count`` column is the number of values which were within the bucket's
range. For example, if the ``range`` is ``[10, 12)``, ``10`` and ``11`` would
fall inside the bucket's range.

The ``overflow`` column is the number of values which fell outside the bucket's
range. It is always be zero except for the first bucket, where it may be
negative, and the last bucket, where it may be positive. For example, if the
first bucket's range is ``[10, 12)`` and the values are ``9``, ``10``, ``11``,
then the first bucket's ``count`` will be ``2`` (for ``10`` and ``11``), and
the first bucket's ``overflow`` will be ``-1`` (for ``9``).

The ``bar`` column is a bar graph showing the count and overflow for each
bucket. The overflow, if present, will be shown with ``-`` for the first
bucket, and ``+`` on the last bucket.

The ``cumbar`` column is a bar graph showing the cumulative sum sum (ie, where
the last bar will always be 100%).

The ``cumsum`` and ``cumpct`` are the cumulative sum and cumulative percent.


The Functions
-------------

The ``histogram`` aggregate function accepts three arguments::

    histogram(value, minimum_value, maximum_value, number_of_buckets)

And returns ``histogram_result[]``, which can be passed to ``show_histogram``::

    psql# SELECT UNNEST(histogram(generate_series, 10, 20, 10)) from generate_series(5, 25);
            unnest        
    ----------------------
     (1,-5,6,0,"[10,11)")
     (1,0,1,1,"[11,12)")
     (1,0,1,2,"[12,13)")
     (1,0,1,3,"[13,14)")
     ...
    (10 rows)
    psql# SELECT * FROM show_histogram((SELECT histogram(generate_series, 10, 20, 10) from generate_series(5, 25)));
     bucket |  range  | count | overflow |                 bar                 |             cumbar             | cumsum |      cumpct       
    --------+---------+-------+----------+-------------------------------------+--------------------------------+--------+-------------------
          0 | [10,11) |     1 |       -5 | =====-------------------------      | =========                      |      6 | 0.285714285714286
          1 | [11,12) |     1 |        0 | =====                               | ==========                     |      7 | 0.333333333333333
     ...
          8 | [18,19) |     1 |        0 | =====                               | ====================           |     14 | 0.666666666666667
          9 | [19,20) |     1 |        6 | =====++++++++++++++++++++++++++++++ | ============================== |     21 |                 1
    (10 rows)


The ``histogram_version`` function returns the current version::

    psql# SELECT histogram_version();
     histogram_version 
    -------------------
     0.1.0
    (1 row)
