---
layout: default
---

# Google summer  of code notes


## Multi-hosts monitoring set up
Follow [Record metrics from a remote system](https://pcp.readthedocs.io/en/latest/QG/RecordMetricsFromRemoteSystem.html)
to set up the remote system (can be a virtual machine on the same computer).

On your local host, 
1. use `sudo vi /etc/hosts` to edit the hosts on your machine and add the new system mapping to the file.

2. go to `/etc/pcp/pmlogger`,
    * The control `file` contains one line per host to be logged.

    * The file `control.d` stores the config of each host

3. Copy the local file and give it a name for your remote system. set n to primary option, which means this remote system is not primary, and your local machine should be primary system.
The arguments for the hosts are 
* `-r`: creates the local config
* `-T`: terminating cycle
* `-c`: config file for pmlogger 
* `-v`: volume size. Once the archieve meets the set volume, a new archieve will be created.

4. Use `sudo systemctl restart pmlogger` to restart pmlogger, and use
`sudo systemctl status pmlogger` to check its status.

5. Go to the dir `cd /var/log/pcp/pmlogger/shiyao_fedora` where stores the config of your second system to see details.
* Files 20220726.15.27.* have all the metrics sent from redis. Once this file meets the set volume (the -v option), a new one with the same prefix will be created, and new data will be stored to the new files. and the old files will be compressed. 
* File 20220726.15.27.index is a lookup table for the previous files. It's used for a quicker data query.
* File 20220726.15.27.meta is the metadata from redis, it stores metric names, descriptors, labels and so on.
* File Latest is a PCP archive folio
* File pmlogger.log is created by -r option. It stores the query frequency for each metric.
6. We can use `sudo systemctl stop pmproxy` to stop the new messages from redis, and use `sudo systemctl restart pmproxy` to restart the pmproxy and allow new messages from redis server.

7. We can use `pmseries -a 805f4cdf368337dd564c365909543cc86a39275e` to see where the data come from (local or remote).

## Special notes
### What if qa fails
Before running `./check ...`, run `pmseries --load "{source.path: \"PATH/pcp/qa/archives/proc\"}"`.
* If unable to connect to redis server with the error msg 'Segmentation fault (core dumped)', try to run `sudo make clean` and rebuild the project. This should solve the segmentation fault problem.

## Early June (Week 1 & Week2)
1. Set up testing environment
* Use the following commands to check which .so is linked
```
which pmseries
ls -l /usr/local/lib
ls -l /usr/lib/libpcp*
ldd /usr/local/bin/pmseries
```
and use
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/libpcp_web.so.1
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/libpcp_web.so.1
```
to link to the correct .so.1 file
2. finsed time domain operations for `min()` and `max()` functions
### Notes
1. Use `sudo ./check -g pmseries` to run all pmseries tests, or use `sudo ./check 1886`to run a specific test.

2. If a function is implemented, remember to 
* run tests,
* update man pages, and
* run valgrind --leak-check=full
3. To update man pages, go to `/pcp/man/man1/pmseries.1` file and run `man ./pmseries.1` once new function descriptions are added
4. Use `gdb --args pmseries "..."` to debug

## Late June (Week 3 & Week4)
1. Implemented time domain operation: `sum_sample()` and `avg_sample()`
2. Implemented time domain and instance domain operations for standard deviation, i.e. `stdev_inst()` and `stdev_sample()`.
### Notes
1. Remember to update 
```
np->value_set.series_values[i].series_desc.type
np->value_set.series_values[i].series_desc.semantics
np->value_set.series_values[i].series_desc.units
```
if any operation is done to the original redis data.
Checkout 
[pmSemStr](https://man7.org/linux/man-pages/man3/pmsemstr.3.html) and
[pmUnitsStr](https://man7.org/linux/man-pages/man3/pmUnitsStr.3.html) for more information.
2. Try to test on multi-host environment: 
[Record metrics from a remote system](https://pcp.readthedocs.io/en/latest/QG/RecordMetricsFromRemoteSystem.html)

## Early July (Week 5 & 6)
1. Implemented operations for `topk_inst()` and `topk_sample()`.
### Notes
1. use `./new` to create a new qa tests under the qa folder.

## Late July (Week 7 & 8)
1. Implemented `nth_percentile_inst()` and `nth_percentile_inst()`.
### Notes
1. [HdrHistogram_c](https://github.com/HdrHistogram/HdrHistogram_c) provides examples for histogram function.
HDR stands for high dynamic range.
2. [bpftrace](https://bpftrace.org/) provides examples of histogram output.

## Early August (Week 9)
1. Implemented scalar multiplication and its tests
* Note: does not have overflow handling. Current method is to report error when overflow occurs.

## Early September (Week 10 & 11)
1. Understood callback for creating histogram bar chart.
* pcp/src/include/pcp/pmwebapi.h: add a structure to store histogram values
* pcp/src/pmseries/pmseries.c: add call back for on_histogram_value
### Notes
1. Try to use HdrHistogram and it can be one of the vendors for pcp.

## Late September (12 & 13)


[back](.././)
