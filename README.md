# NanoLog
* Low Latency C++11 Logging Library. 
* It's fast. Very fast. See [Latency benchmark](#latency-benchmark-of-guaranteed-logger)
* NanoLog only uses standard headers so it should work with any C++11 compliant compiler.
* Supports typical logger features namely multiple log levels, log file rolling and asynchronous writing to file.

# Design highlights
* Zero copying of string literals.
* Lazy conversion of integers and doubles to ascii. 
* No heap memory allocation for log lines representable in less than ~256 bytes.
* Minimalistic header includes. Avoids common pattern of header only library. Helps in compilation times of projects.

# Guaranteed and Non Guaranteed logging
* Nanolog supports Guaranteed logging i.e. log messages are never dropped even at extreme logging rates.
* Nanolog also supports Non Guaranteed logging. Uses a ring buffer to hold log lines. When the ring gets full, the previous log line in the slot will be dropped. Does not block producer even if the ring buffer is full.

# Usage
```c++
#include "NanoLog.hpp"

int main()
{
  // Ensure initialize is called once prior to logging.
  // This will create log files like /tmp/nanolog1.txt, /tmp/nanolog2.txt etc.
  // Log will roll to the next file after every 1MB.
  // This will initialize the guaranteed logger.
  nanolog::initialize(nanolog::GuaranteedLogger(), "/tmp/", "nanolog", 1);
  
  // Or if you want to use the non guaranteed logger -
  // ring_buffer_size_mb - LogLines are pushed into a mpsc ring buffer whose size
  // is determined by this parameter. Since each LogLine is 256 bytes,
  // ring_buffer_size = ring_buffer_size_mb * 1024 * 1024 / 256
  // In this example ring_buffer_size_mb = 3.
  // nanolog::initialize(nanolog::NonGuaranteedLogger(3), "/tmp/", "nanolog", 1);
  
  for (int i = 0; i < 5; ++i)
  {
    LOG_INFO << "Sample NanoLog: " << i;
  }
  
  // Change log level at run-time.
  nanolog::set_log_level(nanolog::LogLevel::CRIT);
  LOG_WARN << "This log line will not be logged since we are at loglevel = CRIT";
  
  return 0;
}
```
# Latency benchmark of Guaranteed logger
* A google search for fast logger C++ gives the first result [spdlog](https://github.com/gabime/spdlog)
* Theres another interesting [article](https://kjellkod.wordpress.com/2015/06/30/the-worlds-fastest-logger-vs-g3log/) on worst case latency by the author of [g3log](https://github.com/KjellKod/g3log)
* So let's benchmark NanoLog vs [spdlog](https://github.com/gabime/spdlog) vs [g3log](https://github.com/KjellKod/g3log).
* Take a look at [nano_vs_spdlog_vs_g3log.cpp](https://github.com/Iyengar111/NanoLog/blob/master/nano_vs_spdlog_vs_g3log.cpp)
* Benchmark was compiled with g++ 4.8.4 running Linux Mint 17 on Intel(R) Core(TM) i7-2630QM CPU @ 2.00GHz
```
Thread count: 1

nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        1|        4|        8|       68| 0.347930|

spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        3|        3|        3|        5|       11|      129| 2.588590|

g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        5|        6|        6|       10|       19|      186| 5.206230|




Thread count: 2

nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        1|        2|        5|       55| 0.457240|
nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        1|        2|        5|       81| 0.459090|

spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        2|        3|        3|        3|        5|       25| 2.449580|
spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        2|        3|        3|        3|        6|       21| 2.457150|

g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        4|        5|        6|       12|       18|       64| 4.574850|
g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        4|        5|        6|       12|       20|       84| 4.586590|





Thread count: 3

nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        1|        3|        6|       91| 0.450700|
nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        2|        3|        7|       90| 0.676050|
nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        2|        3|        7|      262| 0.680430|

spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        2|        2|        2|        4|        6|     6729| 1.803570|
spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        3|        3|        3|        5|        8|       25| 2.679420|
spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        3|        3|        3|        5|       10|       50| 2.685230|

g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        4|        4|        6|       17|       27|       53| 4.385530|
g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        4|        4|        6|       16|       26|       55| 4.435680|
g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        6|        7|        8|       19|       29|     1031| 5.896250|





Thread count: 4

nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        2|        3|        6|       53| 0.582140|
nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        2|        3|        7|       70| 0.608980|
nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        2|        3|        7|       62| 0.803630|
nanolog_guaranteed percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        0|        1|        2|        3|        7|       61| 0.797270|

spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        2|        2|        2|        3|        5|       40| 1.767930|
spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        2|        2|        2|        3|        6|       21| 1.768640|
spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        3|        3|        3|        4|        8|       24| 2.676170|
spdlog percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        3|        3|        3|        5|       10|       31| 2.698580|

g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        4|        4|        5|       17|       30|     7766| 4.620760|
g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        6|        7|        9|       21|       35|     8478| 6.368940|
g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        6|        7|        8|       22|       32|     1327| 7.023880|
g3log percentile latency numbers in microseconds
     50th|     75th|     90th|     99th|   99.9th|    Worst|  Average|
        7|        8|        9|       23|       36|     8470| 7.831750|

```

# Latency benchmark of Non guaranteed logger
* Take a look at [non_guaranteed_nanolog_benchmark.cpp](https://github.com/Iyengar111/NanoLog/blob/master/non_guaranteed_nanolog_benchmark.cpp) for the code used to generate the latency numbers.
* Benchmark was compiled with g++ 4.8.4 running Linux Mint 17 on Intel(R) Core(TM) i7-2630QM CPU @ 2.00GHz
```
Thread count: 1
	Average NanoLog Latency = 131 nanoseconds
Thread count: 2
	Average NanoLog Latency = 182 nanoseconds
	Average NanoLog Latency = 272 nanoseconds
Thread count: 3
	Average NanoLog Latency = 216 nanoseconds
	Average NanoLog Latency = 209 nanoseconds
	Average NanoLog Latency = 315 nanoseconds
Thread count: 4
	Average NanoLog Latency = 229 nanoseconds
	Average NanoLog Latency = 221 nanoseconds
	Average NanoLog Latency = 233 nanoseconds
	Average NanoLog Latency = 332 nanoseconds
Thread count: 5
	Average NanoLog Latency = 247 nanoseconds
	Average NanoLog Latency = 240 nanoseconds
	Average NanoLog Latency = 320 nanoseconds
	Average NanoLog Latency = 345 nanoseconds
	Average NanoLog Latency = 383 nanoseconds
```
# Crash handling
* [g3log](https://github.com/KjellKod/g3log) has support for crash handling. I do not see the point in re-inventing the wheel. Have a look at that what's done there and if it works for you, give Kjell credit and use his crash handling code.

# Tips to make it faster!
* NanoLog uses standard library chrono timestamps. Your platform / os may have non-standard but faster timestamps. Use them!
