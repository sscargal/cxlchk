# cxlchk

# Description 
`cxlchk` can be used to confirm a system with Compute Express Link (CXL) devices is healthy and the configuration looks sane. cxlchk has two main modules:
- Collector - Collects configuration and health data from the host 
- Analyzer - Runs tests against the collected data and reports whether the test Passes or Fails. If a test fails, there should be some information to explain why.

# Help

```
Usage: ./cxlchk [optional args]
 
Checks for known issues with Compute Express Link (CXL) configurations.
By default, cxlchk runs the collector and analyzer modules.
Run with root privilege.
 
Optional args:
   -A <Path to dataset>
      Specify the path to a dataset. Skips running the collector.
 
   -C
      Run the collector only. Skips running the analyzer.
 
   -c <Path to CXL executable>
      Specify the path to the CXL executable
 
   -h,-?
      Display this help
 
   -l
      List Analyzer Modules and Rules
 
   -v
      Print verbose output. Use -v, -vv, and -vvv to increase verbosity.
```

# Examples

Execute `cxlchk` which runs the Collector and Analyzer
```
$ sudo ./cxlchk
```

Only collect data from the host. Does not run the analyzer
```
$ sudo ./cxlchk -C
```

Runs the analyzer against a previously collected dataset
```
$ sudo ./cxlchk -A ./cxlchk.hostname.0113-1210
```

List the available Analyzer Modules and Rules
```
$ sudo ./cxlchk -l
Analyzer Modules
================
linux
cxl

Analyzer Rules
==============
linux/full_file_system
analyzer/linux/memmap
analyzer/cxl/find_devices
analyzer/cxl/offline_mem_blocks
analyzer/cxl/register_locator_dvsec
[...snip...]
```

# Example Output
The example output below shows what to expect when running the data collector and analysis
```
$ sudo ./cxlchk
=======================================================================
Starting CXL Checker
cxlchk Version 0.1.0
Started: Thu Apr 27 08:37:29 AM PDT 2023
=======================================================================
Operating System: Fedora Linux 36 (Server Edition)
Kernel Version : 6.3.0-rc6
CPU(s):                          208
Model name:                      Intel(R) Xeon(R) Platinum 8470 CPU @2.00GHz
Socket(s):                       2
NUMA node(s):                    3
NUMA node0 CPU(s):               0-51,104-155
NUMA node1 CPU(s):               52-103,156-207
NUMA node2 CPU(s):               
Using DAXCTL command: /usr/bin/daxctl
DAXCTL version: 76.1
Using CXL command: /usr/bin/cxl
CXL version: 76.1
=======================================================================
Starting data collector
=======================================================================
Collecting command outputs
100% (13 of 13)  [====================] 
Collecting files
100% (5 of 5)  [====================] 
Running CXL Collector
100% (3 of 3)  [====================] 
Running DAXCTL Collector
100% (5 of 5)  [====================] 
=======================================================================
Data collector completed
=======================================================================
=======================================================================
Starting analysis of the data
=======================================================================
[ PASSED   ] linux_check_full_file_systems : All file systems are <75% full.
[ PASSED   ] linux_check_memmap : No 'memmap' entries found in the Linux boot command line (/proc/cmdline)
[ PASSED   ] cxl_find_devices : One or more 'CXL' entries found in 'lspci'!
[ PASSED   ] cxl_find_devices : All memory blocks were ONLINE
=======================================================================
Data analysis completed
=======================================================================
=======================================================================
Analysis Report Summary
=======================================================================
[ PASSED   ] = 4
[ FAILED   ] = 0
[ SKIPPED  ] = 0
[ INFO     ] = 0
[ WARNING  ] = 0
[ CRITICAL ] = 0
[ UNKNOWN  ] = 0
=======================================================================
CXL Checker Complete
Ended: Thu Apr 27 08:37:30 AM PDT 2023
Duration: 1 seconds
Results: ./cxlchk.sr0.0427-0837
Logfile: ./cxlchk.sr0.0427-0837/cxlchk.log
=======================================================================
```
