========
Overview
========

High Dynamic Range Histogram python implementation

.. image:: https://badges.gitter.im/Join Chat.svg
   :target: https://gitter.im/HdrHistogram/HdrHistogram

.. image:: https://travis-ci.org/HdrHistogram/HdrHistogram_py.svg?branch=master
   :target: https://travis-ci.org/HdrHistogram/HdrHistogram_py


This repository contains a port to python of most of the original Java HDR Histogram
library:

- Basic histogram value recording
    - record value
    - record value with correction for coordinated omission
- Supports 16-bit, 32-bit and 64-bit counters
- All histogram basic query APIs
    - get value at percentile
    - get total count
    - get min value, max value, mean, standard deviation
- All iterators are implemented: all values, recorded, percentile, linear, logarithmic
- Text file histogram log writer and log reader
- Histogram encoding and decoding (HdrHistogram V2 format only, V1 and V0 not supported)
- supports python 2.7 and python 3

Histogram V2 format encoding inter-operability with Java and C versions verified through unit test code.

Python API
----------
Users of this library can generally play 2 roles (often both):

- record values into 1 or more histograms (histogram provisioning)
- analyze and display histogram content and characteristics (histogram query)

In distributed cases, histogram provisioning can be be done remotely (and possibly in multiple locations) then
aggregated in a central place for analysis.

A histogram instance can be created using the HdrHistogram class and specifying the
minimum and maximum trackable value and the number of precision digits desired.
For example to create a histogram that can count values in the [1..3600000] range and
1% precision (this could be for example to track latencies in the range [1 msec..1 hour]):

.. code::

     histogram = HdrHistogram(1, 60 * 60 * 1000, 2)

By default counters are 64-bit while 16 or 32-bit counters can be specified (word_size
option set to 2 or 4 bytes).
Note that counter overflow is not tested in this version so be careful when using
smaller counter sizes.

Once created it is easy to add values to a histogram:

.. code::

     histogram.record_value(latency)

If the code that generates the values is subject to Coordinated Omission,
use the corrected version of that method (example when the expected interval is
10 msec):

.. code::

     histogram.record_corrected_value(latency, 10)

At any time, the histogram can be queried to return any property, such as getting
the total number of values recorded or the value at a given percentile:

.. code::

     count = histogram.get_total_count()
     value = histogram.get_value_at_percentile(99.9)

Recorded values can be iterated over using the recorded iterator:

.. code::

    for item in histogram.get_recorded_iterator():
        print('value=%f count=%d percentile=%f' %
                item.value_iterated_to,
                item.count_added_in_this_iter_step,
                item.percentile)


An encoded/compressed histogram can be generated by calling the compress method:

.. code::

     encoded_histogram = histogram.encode()

And on reception, a compressed histogram can be decoded from the encoded string:

.. code::

     decoded_histogram = HdrHistogram.decode(encoded_histogram)
     count = decoded_histogram.get_total_count()

In the case of aggregation, the decode_and_add method can be used:

.. code::

     aggregation_histogram.decode_and_add(encoded_histogram)

If you want to print the histogram in the standard tabular format:

.. code::

    histogram.output_percentile_distribution(file, scaling_ratio)
    
For additional help on how to use the API:

- browse through the python code and check the API documentation in the comment section for each method (where available)
- the best documentation is by looking at the test code under the test directory

The test code (https://github.com/HdrHistogram/HdrHistogram_py/blob/master/test/test_hdrhistogram.py) pretty much covers every API.

Installation
------------
Pre-requisites:

Make sure you have python 2.7 or 3, and pip installed

Binary installation
^^^^^^^^^^^^^^^^^^^
This is the preferred method for most installations where you only need to use this library.
Use a python virtual environment if needed.

.. code::

    pip install hdrhistogram

Source code installation and Unit Testing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is the method to use for any development work with this library or if you
want to read or run the test code.

Install the unit test automation harness tox and hdrhistogram from github:

.. code::

    pip install tox
    # cd to the proper location to clone the repository
    git clone https://github.com/HdrHistogram/HdrHistogram_py.git
    cd hdrhistogram

Running tox will execute the following targets:

- pep8/flake8 for syntax and indentation checking
- python unit test code (python 2.7 and 3)
- pylint

Just run tox without any argument (the first run will take more time as tox will setup the execution environment and download the necessary packages):

.. code::

    $ tox
    GLOB sdist-make: /openstack/pyhdr/HdrHistogram_py/setup.py
    py27 inst-nodeps: /openstack/pyhdr/HdrHistogram_py/.tox/dist/hdrhistogram-0.5.2.zip
    py27 installed: astroid==1.5.3,backports.functools-lru-cache==1.4,configparser==3.5.0,enum34==1.1.6,flake8==3.3.0,future==0.16.0,hdrhistogram==0.5.2,isort==4.2.15,lazy-object-proxy==1.3.1,mccabe==0.6.1,pbr==3.1.1,py==1.4.34,pycodestyle==2.3.1,pyflakes==1.5.0,pylint==1.7.1,pytest==3.1.2,singledispatch==3.4.0.3,six==1.10.0,wrapt==1.10.10
    py27 runtests: PYTHONHASHSEED='4015036329'
    py27 runtests: commands[0] | py.test -q -s --basetemp=/openstack/pyhdr/HdrHistogram_py/.tox/py27/tmp
    ......................ss.........
    31 passed, 2 skipped in 5.14 seconds
    py3 inst-nodeps: /openstack/pyhdr/HdrHistogram_py/.tox/dist/hdrhistogram-0.5.2.zip
    py3 installed: You are using pip version 8.1.1, however version 9.0.1 is available.,You should consider upgrading via the 'pip install --upgrade pip' command.,flake8==2.5.4,future==0.15.2,hdrhistogram==0.5.2,mccabe==0.4.0,pbr==1.9.1,pep8==1.7.0,py==1.4.31,pyflakes==1.0.0,pytest==2.9.1
    py3 runtests: PYTHONHASHSEED='4015036329'
    py3 runtests: commands[0] | py.test -q -s --basetemp=/openstack/pyhdr/HdrHistogram_py/.tox/py3/tmp
    s......................ss.........
    31 passed, 3 skipped in 5.11 seconds
    pep8 inst-nodeps: /openstack/pyhdr/HdrHistogram_py/.tox/dist/hdrhistogram-0.5.2.zip
    pep8 installed: You are using pip version 8.1.1, however version 9.0.1 is available.,You should consider upgrading via the 'pip install --upgrade pip' command.,flake8==2.5.4,future==0.15.2,hdrhistogram==0.5.2,mccabe==0.4.0,pbr==1.9.1,pep8==1.7.0,py==1.4.31,pyflakes==1.0.0,pytest==2.9.1
    pep8 runtests: PYTHONHASHSEED='4015036329'
    pep8 runtests: commands[0] | flake8 hdrh test
    lint inst-nodeps: /openstack/pyhdr/HdrHistogram_py/.tox/dist/hdrhistogram-0.5.2.zip
    lint installed: astroid==1.5.3,backports.functools-lru-cache==1.4,configparser==3.5.0,enum34==1.1.6,flake8==3.3.0,future==0.16.0,hdrhistogram==0.5.2,isort==4.2.15,lazy-object-proxy==1.3.1,mccabe==0.6.1,pbr==3.1.1,py==1.4.34,pycodestyle==2.3.1,pyflakes==1.5.0,pylint==1.7.1,pytest==3.1.2,singledispatch==3.4.0.3,six==1.10.0,wrapt==1.10.10
    lint runtests: PYTHONHASHSEED='4015036329'
    lint runtests: commands[0] | pylint --rcfile pylint.rc hdrh test

    --------------------------------------------------------------------
    Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)

    ________________________________________________________________ summary ________________________________________________________________
      py27: commands succeeded
      py3: commands succeeded
      pep8: commands succeeded
      lint: commands succeeded
      congratulations :)

Utility to dump an encoded histogram string (dump_hdrh)
-------------------------------------------------------

You can dump any encoded histogram using the dump_hdrh tool (installed along with the package).

.. code::

   $ dump_hdrh

   Usage: dump_hdrh [<string encoded hdr histogram>]*

You can pass one or more histogram strings to the tools:

.. code::

   $ dump_hdrh 'HISTFAAAACl4nJNpmSzMwMDAxQABzFCaEUzOmNZg/wEi0NzIyPSYlWmpGBMAh4gG4A=='

   Dumping histogram: HISTFAAAACl4nJNpmSzMwMDAxQABzFCaEUzOmNZg/wEi0NzIyPSYlWmpGBMAh4gG4A==

         Value     Percentile TotalCount 1/(1-Percentile)

   139647.000 0.000000000000          1           1.00
   139647.000 0.100000000000          1           1.11
   139647.000 0.190000000000          1           1.23
   139647.000 0.271000000000          1           1.37
   187135.000 0.343900000000          2           1.52
   187135.000 0.409510000000          2           1.69
   187135.000 0.468559000000          2           1.88
   187135.000 0.521703100000          2           2.09
   187135.000 0.569532790000          2           2.32
   187135.000 0.612579511000          2           2.58
   187135.000 0.651321559900          2           2.87
   477695.000 0.686189403910          3           3.19
   477695.000 1.000000000000          3
   #[Mean    =   268074.667, StdDeviation   =   149397.390]
   #[Max     =   477695.000, TotalCount     =        3.000]
   #[Buckets =           14, SubBuckets     =         2048]


Aggregation of Distributed Histograms
-------------------------------------

Aggregation of multiple histograms into 1 is useful in cases where tools
that generate these individual histograms have to run in a distributed way in
order to scale sufficiently.
As an example, the wrk2 tool (https://github.com/giltene/wrk2.git) is a great
tool for measuring the latency of HTTP requests with a large number of
connections. Although this tool can support thousands of connections per
process, some setups require massive scale in the order of hundreds of
thousands of connections which require running a large number of instances of
wrk processes, possibly on a large number of servers.
Given that each instance of wrk can generate a separate histogram, assessing
the scale of the entire system requires aggregating all these histograms
into 1 in a way that does not impact the accuracy of the results.
So there are 2 problems to solve:

- find a way to properly aggregate multiple histograms without losing any detail

- find a way to transport all these histograms into a central place

This library provides a solution for the aggregation part of the problem:

- reuse the HDR histogram compression format version 1 to encode and compress a complete histogram that can be sent over the wire to the aggregator

- provide python APIs to easily and efficiently:

  * compress an histogram instance into a transportable string
  * decompress a compressed histogram and add it to an existing histogram

Refer to the unit test code (test/test_hdrhistogram.py) to see how these APIs can be used.

Histogram wire encoding and size
--------------------------------
Histograms are encoded using the HdrHistogram V2 format which is based on an adapted ZigZag LEB128 encoding where:

- consecutive zero counters are encoded as a negative number representing the count of consecutive zeros
- non zero counter values are encoded as a positive number

An empty histogram (all zeros counters) is encoded in exactly 48 bytes regardless of the counter size.
A typical histogram (2 digits precision 1 usec to 1 day range) can be encoded in less than the typical MTU size of 1500 bytes.

This format is compatible with the HdrHistogram Java and C implementations.

Performance
-----------
Histogram value recording has the same cost characteristics than the original Java version
since it is a direct port (fixed cost for CPU and reduced memory usage).
Encoding and decoding in the python version is very fast and close to native performance thanks to the use of:

- integrated C extensions (native C code called from python) that have been developed to handle the low-level byte encoding/decoding/addition work at native speed
- native compression library (zlib and base64)

On a macbook pro (Intel Core i7 @ 2.3GHz) and Linux server (Intel(R) Xeon(R) CPU E5-2665 @ 2.40GHz):

+---------------------------+-----------+--------+
| Operation Time in usec    |   Macbook |  Linux |
+===========================+===========+========+
| record a value            |        2  |    1.5 |
+---------------------------+-----------+--------+
| encode typical histogram  |      100  |   90   |
+---------------------------+-----------+--------+
| decode and add            |      150  |  125   |
+---------------------------+-----------+--------+


The typical histogram is defined as one that has 30% of 64-bit buckets filled with
sequential values starting at 20% of the array, for a range of 1 usec to 24 hours
and 2 digits precision. This represents a total of 3968 buckets, of which
the first 793 are zeros, the next 1190 buckets have a sequential/unique value and all
remaining buckets are zeros, for an encoded length of 3116 bytes. Most real-world histograms
have a much sparser pattern that will yield a lower encoding and decoding time.
Decode and add will decode the encoded histogram and add its content to an existing histogram.

To measure the performance of encoding and decoding and get the profiling, use the
--runperf option. The 2 profiling functions will provide the profiling information
for encoding and decoding the typical histogram 1000 times (so the time values shown
are seconds for 1000 decodes/decodes).

Example of run on the same macbook pro:

.. code::

    $ tox -e py27 '-k test_cod_perf --runperf'
    GLOB sdist-make: /openstack/pyhdr/hdrhistogram/setup.py
    py27 inst-nodeps: /openstack/pyhdr/hdrhistogram/.tox/dist/hdrhistogram-0.2.3.dev1.zip
    py27 installed: flake8==2.4.1,hdrhistogram==0.2.3.dev1,mccabe==0.3.1,numpy==1.9.2,pbr==1.7.0,pep8==1.5.7,py==1.4.30,pyflakes==0.8.1,pytest==2.7.2,wsgiref==0.1.2
    py27 runtests: PYTHONHASHSEED='4078653554'
    py27 runtests: commands[0] | py.test -q -s --basetemp=/openstack/pyhdr/hdrhistogram/.tox/py27/tmp -k test_cod_perf --runperf
    0:00:00.095722
             36303 function calls in 0.107 seconds

       Ordered by: standard name

       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.000    0.000    0.107    0.107 <string>:1(<module>)
         2000    0.004    0.000    0.004    0.000 __init__.py:505(string_at)
         1000    0.001    0.000    0.007    0.000 base64.py:42(b64encode)
            1    0.000    0.000    0.000    0.000 codec.py:109(__init__)
            1    0.000    0.000    0.000    0.000 codec.py:144(_init_counts)
            1    0.000    0.000    0.000    0.000 codec.py:162(get_counts)
         1000    0.008    0.000    0.074    0.000 codec.py:204(compress)
            1    0.000    0.000    0.000    0.000 codec.py:246(__init__)
            1    0.000    0.000    0.000    0.000 codec.py:275(get_counts)
         1000    0.005    0.000    0.094    0.000 codec.py:284(encode)
            1    0.000    0.000    0.000    0.000 codec.py:59(get_encoding_cookie)
            1    0.000    0.000    0.000    0.000 codec.py:63(get_compression_cookie)
         2190    0.002    0.000    0.003    0.000 histogram.py:139(_clz)
         2190    0.003    0.000    0.006    0.000 histogram.py:150(_get_bucket_index)
         2190    0.001    0.000    0.001    0.000 histogram.py:156(_get_sub_bucket_index)
         1190    0.001    0.000    0.001    0.000 histogram.py:159(_counts_index)
         1190    0.001    0.000    0.006    0.000 histogram.py:169(_counts_index_for)
         1190    0.003    0.000    0.009    0.000 histogram.py:174(record_value)
         1190    0.000    0.000    0.000    0.000 histogram.py:228(get_value_from_sub_bucket)
         1190    0.001    0.000    0.001    0.000 histogram.py:231(get_value_from_index)
            1    0.000    0.000    0.000    0.000 histogram.py:31(get_bucket_count)
         1000    0.001    0.000    0.095    0.000 histogram.py:413(encode)
         1000    0.001    0.000    0.005    0.000 histogram.py:456(get_counts_array_index)
            1    0.000    0.000    0.000    0.000 histogram.py:62(__init__)
            1    0.001    0.001    0.012    0.012 test_hdrhistogram.py:374(fill_hist_counts)
            1    0.001    0.001    0.107    0.107 test_hdrhistogram.py:489(check_cod_perf)
         5000    0.001    0.000    0.001    0.000 {_ctypes.addressof}
         1000    0.005    0.000    0.005    0.000 {binascii.b2a_base64}
         2190    0.001    0.000    0.001    0.000 {bin}
            2    0.000    0.000    0.000    0.000 {built-in method now}
         3190    0.000    0.000    0.000    0.000 {len}
            1    0.000    0.000    0.000    0.000 {math.ceil}
            1    0.000    0.000    0.000    0.000 {math.floor}
            4    0.000    0.000    0.000    0.000 {math.log}
            2    0.000    0.000    0.000    0.000 {math.pow}
         1190    0.000    0.000    0.000    0.000 {max}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
         1000    0.001    0.000    0.001    0.000 {method 'join' of 'str' objects}
         1190    0.000    0.000    0.000    0.000 {min}
         1000    0.008    0.000    0.008    0.000 {pyhdrh.encode}
         1000    0.056    0.000    0.056    0.000 {zlib.compress}

And for decoding:

.. code::

    $ tox -e py27 '-k test_dec_perf --runperf'
    GLOB sdist-make: /openstack/pyhdr/hdrhistogram/setup.py
    py27 inst-nodeps: /openstack/pyhdr/hdrhistogram/.tox/dist/hdrhistogram-0.2.3.dev1.zip
    py27 installed: flake8==2.4.1,hdrhistogram==0.2.3.dev1,mccabe==0.3.1,numpy==1.9.2,pbr==1.7.0,pep8==1.5.7,py==1.4.30,pyflakes==0.8.1,pytest==2.7.2,wsgiref==0.1.2
    py27 runtests: PYTHONHASHSEED='2608914940'
    py27 runtests: commands[0] | py.test -q -s --basetemp=/openstack/pyhdr/hdrhistogram/.tox/py27/tmp -k test_dec_perf --runperf
    0:00:00.149938
             115325 function calls in 0.160 seconds

       Ordered by: standard name

       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.000    0.000    0.160    0.160 <string>:1(<module>)
            2    0.000    0.000    0.000    0.000 __init__.py:505(string_at)
            1    0.000    0.000    0.000    0.000 base64.py:42(b64encode)
         1000    0.001    0.000    0.012    0.000 base64.py:59(b64decode)
         1001    0.001    0.000    0.023    0.000 codec.py:109(__init__)
         1001    0.009    0.000    0.009    0.000 codec.py:144(_init_counts)
         1000    0.002    0.000    0.022    0.000 codec.py:147(init_counts)
         3001    0.001    0.000    0.001    0.000 codec.py:162(get_counts)
         1000    0.004    0.000    0.022    0.000 codec.py:165(_decompress)
            1    0.000    0.000    0.000    0.000 codec.py:204(compress)
         1001    0.002    0.000    0.003    0.000 codec.py:246(__init__)
         3001    0.001    0.000    0.002    0.000 codec.py:275(get_counts)
            1    0.000    0.000    0.000    0.000 codec.py:284(encode)
         1000    0.005    0.000    0.041    0.000 codec.py:306(decode)
         1000    0.002    0.000    0.010    0.000 codec.py:352(add)
         3000    0.001    0.000    0.001    0.000 codec.py:50(get_cookie_base)
         1000    0.001    0.000    0.001    0.000 codec.py:53(get_word_size_in_bytes_from_cookie)
            1    0.000    0.000    0.000    0.000 codec.py:59(get_encoding_cookie)
         1001    0.000    0.000    0.000    0.000 codec.py:63(get_compression_cookie)
         7191    0.004    0.000    0.008    0.000 histogram.py:139(_clz)
         7191    0.009    0.000    0.017    0.000 histogram.py:150(_get_bucket_index)
         7191    0.003    0.000    0.003    0.000 histogram.py:156(_get_sub_bucket_index)
         1190    0.000    0.000    0.000    0.000 histogram.py:159(_counts_index)
         1190    0.001    0.000    0.005    0.000 histogram.py:169(_counts_index_for)
         1190    0.002    0.000    0.008    0.000 histogram.py:174(record_value)
        10190    0.003    0.000    0.003    0.000 histogram.py:228(get_value_from_sub_bucket)
         4190    0.004    0.000    0.005    0.000 histogram.py:231(get_value_from_index)
         2000    0.002    0.000    0.008    0.000 histogram.py:240(get_lowest_equivalent_value)
         4000    0.006    0.000    0.019    0.000 histogram.py:248(get_highest_equivalent_value)
         1001    0.011    0.000    0.011    0.000 histogram.py:31(get_bucket_count)
         1000    0.000    0.000    0.000    0.000 histogram.py:326(get_total_count)
         2000    0.001    0.000    0.010    0.000 histogram.py:342(get_max_value)
         2000    0.003    0.000    0.011    0.000 histogram.py:347(get_min_value)
            1    0.000    0.000    0.000    0.000 histogram.py:413(encode)
         1000    0.002    0.000    0.010    0.000 histogram.py:439(set_internal_tacking_values)
            1    0.000    0.000    0.000    0.000 histogram.py:456(get_counts_array_index)
         1000    0.006    0.000    0.044    0.000 histogram.py:495(add)
         1000    0.001    0.000    0.149    0.000 histogram.py:526(decode_and_add)
         1000    0.003    0.000    0.104    0.000 histogram.py:545(decode)
         1001    0.012    0.000    0.060    0.000 histogram.py:62(__init__)
            1    0.001    0.001    0.010    0.010 test_hdrhistogram.py:374(fill_hist_counts)
            1    0.001    0.001    0.160    0.160 test_hdrhistogram.py:502(check_dec_perf)
         3005    0.000    0.000    0.000    0.000 {_ctypes.addressof}
         1000    0.011    0.000    0.011    0.000 {binascii.a2b_base64}
            1    0.000    0.000    0.000    0.000 {binascii.b2a_base64}
         7191    0.003    0.000    0.003    0.000 {bin}
            2    0.000    0.000    0.000    0.000 {built-in method now}
         9192    0.001    0.000    0.001    0.000 {len}
         1001    0.000    0.000    0.000    0.000 {math.ceil}
         1001    0.000    0.000    0.000    0.000 {math.floor}
         4004    0.001    0.000    0.001    0.000 {math.log}
         2002    0.001    0.000    0.001    0.000 {math.pow}
         3190    0.001    0.000    0.001    0.000 {max}
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
         2000    0.003    0.000    0.003    0.000 {method 'from_buffer_copy' of '_ctypes.PyCStructType' objects}
            1    0.000    0.000    0.000    0.000 {method 'join' of 'str' objects}
         3190    0.001    0.000    0.001    0.000 {min}
         1000    0.007    0.000    0.007    0.000 {pyhdrh.add_array}
         1000    0.011    0.000    0.011    0.000 {pyhdrh.decode}
            1    0.000    0.000    0.000    0.000 {pyhdrh.encode}
            1    0.000    0.000    0.000    0.000 {zlib.compress}
         1000    0.015    0.000    0.015    0.000 {zlib.decompress}
    
    .
    ==================================== 30 tests deselected by '-ktest_dec_perf' ====================================
    1 passed, 30 deselected in 0.35 seconds
    ____________________________________________________ summary _____________________________________________________
      py27: commands succeeded
      congratulations :)

Finally, example of profiling when recording a large number of values (record_value
shows 0.313 seconds for 172032 calls):

.. code::

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    1.936    1.936 <string>:1(<module>)
   172044    0.090    0.000    0.189    0.000 histogram.py:137(_clz)
   172044    0.191    0.000    0.379    0.000 histogram.py:148(_get_bucket_index)
   172044    0.066    0.000    0.066    0.000 histogram.py:154(_get_sub_bucket_index)
   172032    0.066    0.000    0.066    0.000 histogram.py:157(_counts_index)
   172032    0.182    0.000    0.693    0.000 histogram.py:167(_counts_index_for)
   172032    0.313    0.000    1.078    0.000 histogram.py:172(record_value)
   344064    0.158    0.000    0.158    0.000 histogram.py:206(get_count_at_index)
   172050    0.038    0.000    0.038    0.000 histogram.py:226(get_value_from_sub_bucket)
   172044    0.139    0.000    0.177    0.000 histogram.py:229(get_value_from_index)
       12    0.103    0.009    0.103    0.009 histogram.py:552(add_counts)
        6    0.122    0.020    1.376    0.229 test_hdrhistogram.py:605(fill_hist_counts)
       12    0.193    0.016    0.351    0.029 test_hdrhistogram.py:612(check_hist_counts)
      
Limitations, Caveats and Known Issues
-------------------------------------

The latest features and bug fixes of the original HDR histogram library may not be available in this python port.
Examples of notable features/APIs not implemented:

- concurrency support (AtomicHistogram, ConcurrentHistogram...)
- DoubleHistogram
- histogram auto-resize
- recorder function

This implementation has byte endianess encoding issues when used with PyPy
due to a limitation of the PyPy code
(see https://github.com/HdrHistogram/HdrHistogram_py/issues/13).

The current implementation has issues running on Windows 32-bit systems (library crashing during decode).

Dependencies
------------
The only dependency (outside of using pytest and tox for the unit testing) is the
small pbr python package which takes care of the versioning (among other things).

Licensing
---------

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

Contribution
------------
External contribution, forks and GitHub pull requests are welcome.


Acknowledgements
----------------

The python code was directly ported from the original HDR Histogram Java and C libraries:

* https://github.com/HdrHistogram/HdrHistogram.git
* https://github.com/HdrHistogram/HdrHistogram_c.git


Links
-----

* Source: https://github.com/HdrHistogram/HdrHistogram_py.git

