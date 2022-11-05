---
layout: default
---

# Google summer  of code notes

Performance Co-Pilot timeseries are series of time-stamped values gathered from hosts making performance
data available. With pmseries query, users can obtain various information about performance
metrics in real time or from historical data. The existing pmseries query language implementation is
maturing. However, there are still some important query grammar and functions needed to be added. This
proposal aims to extend some time series functions in the `libpcp_web` module, including extending the
grammar to support scalar operands in expressions, extending the cross-domain operations, implementing
new statistical functions, and implementing new functions for time series sample matching.

## Pull Request
* Time domain operation for max [PR #1611](https://github.com/performancecopilot/pcp/pull/1611)
* libpcp_web: add pmseries time domain functions [PR #1623](https://github.com/performancecopilot/pcp/pull/1623)
* libpcp_web: solve memory leak [PR #1628](https://github.com/performancecopilot/pcp/pull/1628)
* libpcp_web: modify libpcp_web make file [PR #1630](https://github.com/performancecopilot/pcp/pull/1630)
* libpcp_web: Pmseries nth percentile [PR #1637](https://github.com/performancecopilot/pcp/pull/1637)
* libpcp_web: pmseries language extensions with topk functions [PR #1638](https://github.com/performancecopilot/pcp/pull/1638)
* pmseries: scalar multiplication [PR #1681](https://github.com/performancecopilot/pcp/pull/1681)

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
* Remember to update 
```
np->value_set.series_values[i].series_desc.type
np->value_set.series_values[i].series_desc.semantics
np->value_set.series_values[i].series_desc.units
```
if any operation is done to the original redis data.
Checkout 
[pmSemStr](https://man7.org/linux/man-pages/man3/pmsemstr.3.html) and
[pmUnitsStr](https://man7.org/linux/man-pages/man3/pmUnitsStr.3.html) for more information.
* Try to test on multi-host environment: 
[Record metrics from a remote system](https://pcp.readthedocs.io/en/latest/QG/RecordMetricsFromRemoteSystem.html)

## Early July (Week 5 & 6)
1. Implemented operations for `topk_inst()` and `topk_sample()`.
### Notes
1. use `./new` to create a new qa tests under the qa folder.

## Late July (Week 7 & 8)
1. Implemented `nth_percentile_inst()` and `nth_percentile_inst()`.
### Notes
* [HdrHistogram_c](https://github.com/HdrHistogram/HdrHistogram_c) provides examples for histogram function.
HDR stands for high dynamic range.
* [bpftrace](https://bpftrace.org/) provides examples of histogram output.

## Early August (Week 9)
1. Implemented scalar multiplication and its tests
* Note: does not have overflow handling. Current method is to report error when overflow occurs.

## Early September (Week 10 & 11)
1. Understood callback for creating histogram bar chart.
* pcp/src/include/pcp/pmwebapi.h: added a structure to store histogram values.
* pcp/src/pmseries/pmseries.c: added call back for on_histogram_value.
### Notes
* Try to use HdrHistogram and it can be one of the vendors for pcp.

## Late September (12 & 13)


## Special notes
### What if qa fails
Before running `./check ...`, run `pmseries --load "{source.path: \"PATH/pcp/qa/archives/proc\"}"`.
* If unable to connect to redis server with the error msg 'Segmentation fault (core dumped)', try to run `sudo make clean` and rebuild the project. This should solve the segmentation fault problem.


## Early October (14 & 15)
1. Implemented histogram() function: created a new callback structure to send histogram data.
2. Understand the timeseries sample matching problem, and create graphs to see metric data trends.

## Special notes
Change time period of a metric:
1. go to /var/lib/pcp/config/pmlogger/config.default
2. add a section for the metric with new period, such as
```
log advisory on 2sec {
    disk.all.read
}
```
By doing so, the report period of `disk.all.read` will be change to 2 seconds instead of 10 seconds.

## Late October (16 & 17)
1. Designed algorithm to do upsampling and interpolations of vector operands to match timeseries samples with other vector operands.


## Special notes
1. Figured out the usages of two `timing_t` in both `node_t` and `series_t` structures.
    * `timing_t` in `node_t` is the time periods (`delta\interval` in the pcp time series query expression) for the series root node.
    * `timing_t` in `series_t` is the time intervals for each child roots.

## TODO: After GSOC
1. Keep implementing the Timeseries Sample Matching Function.
2. Solve some memory leak problems.

[back](.././)
