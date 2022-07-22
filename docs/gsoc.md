---
layout: default
---

# Google summer  of code notes

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

[back](.././)
