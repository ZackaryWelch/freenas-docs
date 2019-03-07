.. _Command Line Utilities:

Command Line Utilities
======================

Several command line utilities which are provided with %brand% are
demonstrated in this section.

The following utilities can be used for benchmarking and performance
testing:

* :ref:`Iperf`: used for measuring maximum TCP and UDP bandwidth
  performance

* :ref:`Netperf`: a tool for measuring network performance

* :ref:`IOzone`: filesystem benchmark utility used to perform a broad
  filesystem analysis

* :ref:`arcstat`: used to gather ZFS ARC statistics

The following utilities are specific to RAID controllers:

* :ref:`tw_cli`:_used to monitor and maintain 3ware RAID controllers

* :ref:`MegaCli`: used to configure and manage Broadcom MegaRAID SAS
  family of RAID controllers

This section also describes these utilities:

* :ref:`freenas-debug`: the backend used to dump %brand% debugging
  information

* :ref:`tmux`: a terminal multiplexer similar to GNU screen

* :ref:`Dmidecode`: reports information about system hardware as
  described in the system's BIOS


.. index:: Iperf
.. _Iperf:

Iperf
-----

Iperf is a utility for measuring maximum TCP and UDP bandwidth
performance. It can be used to chart network throughput over time. For
example, it is used to test the speed of different types of shares
to determine which type performs best on the network.

%brand% includes the iperf server. To perform network testing,
install an iperf client on a desktop system that has
network access to the %brand% system. This section demonstrates
how to use the
`xjperf user interface client
<https://code.google.com/archive/p/xjperf/downloads>`__
as it works on Windows, macOS, Linux, and BSD systems.

Since this client is Java-based, the appropriate
`JRE
<http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_
must be installed on the client computer.

Linux and BSD users will need to install the iperf package using the
package management system for their operating system.

To start xjperf on Windows: unzip the downloaded file, start Command
Prompt in Run as administrator mode, :command:`cd` to the unzipped
folder, and run :command:`jperf.bat`.

To start xjperf on macOS, Linux, or BSD, unzip the downloaded file,
:command:`cd` to the unzipped directory, type
:command:`chmod u+x jperf.sh`, and run :command:`./jperf.sh`.

Start the iperf server on %brand% when the client is ready.

.. note:: Beginning with %brand% version 11.1, both `iperf2
   <https://sourceforge.net/projects/iperf2/>`_ and `iperf3
   <http://software.es.net/iperf/>`_ are pre-installed. To use iperf2,
   use :command:`iperf`. To use iperf3, instead type :command:`iperf3`.
   The examples below are for iperf2.

To see the available server options, open Shell and type:

.. code-block:: none

   iperf --help | more

or:

.. code-block:: none

   iperf3 --help | more

For example, to perform a TCP test and start the server in daemon mode
(to get the prompt back), type:

.. code-block:: none

   iperf -sD
   ------------------------------------------------------------
   Server listening on TCP port 5001
   TCP window size: 64.0 KByte (default)
   ------------------------------------------------------------
   Running Iperf Server as a daemon
   The Iperf daemon process ID: 4842


.. note:: The daemon process stops when :ref:`Shell` closes.
   Set up the environment with shares configured and started
   **before** starting the Iperf process.

From the desktop, open the client. Enter the IP of address of the
%brand% system, specify the running time for the test under
:menuselection:`Application layer options --> Transmit`
(the default test time is 10 seconds), and click the
:guilabel:`Run Iperf!` button.
:numref:`Figure %s <cli_view_iperf>`
shows an example of the client running on a
Windows system while an SFTP transfer is occurring on the network.


.. _cli_view_iperf:

.. figure:: images/iperf.png

   Viewing Bandwidth Statistics Using xjperf

Check the type of traffic before testing UPD or TCP.
The iperf server is used to get additional details for
services using TCP :command:`iperf -sD` or UDP :command:`iperf -sDu`.
The startup message indicates when the server is listening for TCP or UDP.
The :command:`sockstat -4 | more` command gives an overview of the services
running on the %brand% system. This helps to determine if the traffic
to test is UDP or TCP.

.. code-block:: none

   sockstat -4 | more
   USER     COMMAND PID     FD PROTO        LOCAL ADDRESS   FOREIGN ADDRESS
   root     iperf   4870    6  udp4         *:5001          *:*
   root     iperf   4842    6  tcp4         *:5001          *:*
   www      nginx   4827    3  tcp4         127.0.0.1:15956 127.0.0.1:9042
   www      nginx   4827    5  tcp4         192.168.2.11:80 192.168.2.26:56964
   www      nginx   4827    7  tcp4         *:80            *:*
   root     sshd    3852    5  tcp4         *:22            *:*
   root     python  2503    5  udp4         *:*             *:*
   root     mountd  2363    7  udp4         *:812           *:*
   root     mountd  2363    8  tcp4         *:812           *:*
   root     rpcbind 2359    9  udp4         *:111           *:*
   root     rpcbind 2359    10 udp4         *:886           *:*
   root     rpcbind 2359    11 tcp4         *:111           *:*
   root     nginx   2044    7  tcp4         *:80            *:*
   root     python  2029    3  udp4         *:*             *:*
   root     python  2029    4  tcp4         127.0.0.1:9042  *:*
   root     python  2029    7  tcp4         127.0.0.1:9042  127.0.0.1:15956
   root     ntpd    1548    20 udp4         *:123           *:*
   root     ntpd    1548    22 udp4         192.168.2.11:123*:*
   root     ntpd    1548    25 udp4         127.0.0.1:123   *:*
   root     syslogd 1089    6  udp4         127.0.0.1:514   *:*


When testing is finished, either type :command:`killall iperf` or
close Shell to terminate the iperf server process.

.. index:: Netperf
.. _Netperf:

Netperf
-------

Netperf is a benchmarking utility that can be used to measure the
performance of unidirectional throughput and end-to-end latency.

Before using the :command:`netperf` command, start its
server process with this command:

.. code-block:: none

   netserver
   Starting netserver with host 'IN(6)ADDR_ANY' port '12865' and family AF_UNSPEC

The following command displays the available options for
performing tests with the :command:`netperf` command. The
`Netperf Manual <https://hewlettpackard.github.io/netperf/>`__
describes each option in more detail and explains how to perform many
types of tests. It is the best reference for understanding how each
test works and how to interpret the results. When testing is
finished, type :command:`killall netserver` to stop the server
process.

.. code-block:: none

 netperf -h |more
 Usage: netperf [global options] -- [test options]
 Global options:
     -a send,recv	Set the local send,recv buffer alignment
     -A send,recv	Set the remote send,recv buffer alignment
     -B brandstr	Specify a string to be emitted with brief output
     -c [cpu_rate]	Report local CPU usage
     -C [cpu_rate]	Report remote CPU usage
     -d			Increase debugging output
     -D [secs,units] *  Display interim results at least every secs seconds
			using units as the initial guess for units per second
     -f G|M|K|g|m|k	Set the output units
     -F fill_file	Pre-fill buffers with data from fill_file
     -h			Display this text
     -H name|ip,fam *	Specify the target machine and/or local ip and family
     -i max,min		Specify the max and min number of iterations (15,1)
     -I lvl[,intvl]	Specify confidence level (95 or 99) (99)
			and confidence interval in percentage (10)
     -j			Keep additional timing statistics
     -l testlen		Specify test duration (>0 secs) (<0 bytes|trans)
     -L name|ip,fam *	Specify the local ip|name and address family
     -o send,recv	Set the local send,recv buffer offsets
     -O send,recv	Set the remote send,recv buffer offset
     -n numcpu		Set the number of processors for CPU util
     -N			Establish no control connection, do 'send' side only
     -p port,lport*	Specify netserver port number and/or local port
     -P 0|1		Don't/Do display test headers
     -r			Allow confidence to be hit on result only
     -s seconds		Wait seconds between test setup and test start
     -S			Set SO_KEEPALIVE on the data connection
     -t testname	Specify test to perform
     -T lcpu,rcpu	Request netperf/netserver be bound to local/remote cpu
     -v verbosity	Specify the verbosity level
     -W send,recv	Set the number of send,recv buffers
     -v level		Set the verbosity level (default 1, min 0)
     -V			Display the netperf version and exit


For those options taking two parms, at least one must be specified.
Specifying one value without a comma will set both parms to that
value, specifying a value with a leading comma will set just the
second parm, and specifying a value with a trailing comma will set the
first. To set each parm to unique values, specify both and separate them
with a comma.

For these options taking two parms, specifying one value with no comma
will only set the first parms and will leave the second at the default
value. To set the second value it must be preceded with a comma or be
a comma-separated pair. This is to retain previous netperf behavior.


.. index:: IOzone
.. _IOzone:

IOzone
------

IOzone is a disk and filesystem benchmarking tool. It can be used to
test file I/O performance for the following operations: read, write,
re-read, re-write, read backwards, read strided, fread, fwrite, random
read, pread, mmap, aio_read, and aio_write.

%brand% ships with IOzone so it can be run from Shell.
When using IOzone on %brand%, :command:`cd` to a directory in a
pool that you have permission to write to, otherwise an
error about being unable to write the temporary file will occur.

Before using IOzone, read through the `IOzone documentation PDF
<http://www.iozone.org/docs/IOzone_msword_98.pdf>`_ as it describes
the tests, the many command line switches, and how to interpret the
results.

These resources provide good
starting points on which tests to run, when to run them, and how to
interpret the results:

* `How To Measure Linux Filesystem I/O Performance With iozone
  <https://www.cyberciti.biz/tips/linux-filesystem-benchmarking-with-iozone.html>`__

* `Analyzing NFS Client Performance with IOzone
  <http://www.iozone.org/docs/NFSClientPerf_revised.pdf>`_

* `10 iozone Examples for Disk I/O Performance Measurement on Linux
  <https://www.thegeekstuff.com/2011/05/iozone-examples/>`_

Type the following command to receive a summary of the available
switches. IOzone is comprehensive so it may take some time
to learn how to use the tests effectively.

Starting with version 9.2.1, %brand% enables compression on newly
created ZFS pools by default. Since IOzone creates test data that is
compressible, this can skew test results. To configure IOzone to
generate incompressible test data, include the options
:samp:`-+w 1 -+y 1 -+C 1`.

Alternatively, consider temporarily disabling compression on the ZFS
pool or dataset when running IOzone benchmarks.

.. note:: If a visual representation of the collected data is
   preferred, scripts are available to render IOzone's output in
   `Gnuplot <http://www.gnuplot.info/>`__.

::

 iozone -h | more
 iozone: help mode
 Usage: iozone[-s filesize_Kb] [-r record_size_Kb] [-f [path]filename] [-h]
	      [-i test] [-E] [-p] [-a] [-A] [-z] [-Z] [-m] [-M] [-t children]
	      [-l min_number_procs] [-u max_number_procs] [-v] [-R] [-x] [-o]
	      [-d microseconds] [-F path1 path2...] [-V pattern] [-j stride]
	      [-T] [-C] [-B] [-D] [-G] [-I] [-H depth] [-k depth] [-U mount_point]
	      [-S cache_size] [-O] [-L cacheline_size] [-K] [-g maxfilesize_Kb]
	      [-n minfilesize_Kb] [-N] [-Q] [-P start_cpu] [-e] [-c] [-b Excel.xls]
	      [-J milliseconds] [-X write_telemetry_filename] [-w] [-W]
	      [-Y read_telemetry_filename] [-y minrecsize_Kb] [-q maxrecsize_Kb]
	      [-+u] [-+m cluster_filename] [-+d] [-+x multiplier] [-+p # ]
	      [-+r] [-+t] [-+X] [-+Z] [-+w percent dedupable] [-+y percent_interior_dedup]
	      [-+C percent_dedup_within]
	  -a  Auto mode
	  -A  Auto2 mode
	  -b Filename  Create Excel worksheet file
	  -B  Use mmap() files
	  -c  Include close in the timing calculations
	  -C  Show bytes transferred by each child in throughput testing
	  -d #  Microsecond delay out of barrier
	  -D  Use msync(MS_ASYNC) on mmap files
	  -e  Include flush (fsync,fflush) in the timing calculations
	  -E  Run extension tests
	  -f  filename to use
	  -F  filenames for each process/thread in throughput test
	  -g #  Set maximum file size (in Kbytes) for auto mode (or #m or #g)
	  -G  Use msync(MS_SYNC) on mmap files
	  -h  help
	  -H #  Use POSIX async I/O with # async operations
	  -i #  Test to run (0=write/rewrite, 1=read/re-read, 2=random-read/write
		3=Read-backwards, 4=Re-write-record, 5=stride-read, 6=fwrite/re-fwrite
		7=fread/Re-fread, 8=random_mix, 9=pwrite/Re-pwrite, 10=pread/Re-pread
		11=pwritev/Re-pwritev, 12=preadv/Re-preadv)
	  -I  Use VxFS VX_DIRECT, O_DIRECT,or O_DIRECTIO for all file operations
	  -j #  Set stride of file accesses to (# * record size)
	  -J #  milliseconds of compute cycle before each I/O operation
	  -k #  Use POSIX async I/O (no bcopy) with # async operations
	  -K  Create jitter in the access pattern for readers
	  -l #  Lower limit on number of processes to run
	  -L #  Set processor cache line size to value (in bytes)
	  -m  Use multiple buffers
	  -M  Report uname -a output
	  -n #  Set minimum file size (in Kbytes) for auto mode (or #m or #g)
	  -N  Report results in microseconds per operation
	  -o  Writes are synch (O_SYNC)
	  -O  Give results in ops/sec.
	  -p  Purge on
	  -P #  Bind processes/threads to processors, starting with this cpu
	  -q #  Set maximum record size (in Kbytes) for auto mode (or #m or #g)
	  -Q  Create offset/latency files
	  -r #  record size in Kb
	     or -r #k .. size in Kb
	     or -r #m .. size in Mb
	     or -r #g .. size in Gb
	  -R  Generate Excel report
	  -s #  file size in Kb
	     or -s #k .. size in Kb
	     or -s #m .. size in Mb
	     or -s #g .. size in Gb
	  -S #  Set processor cache size to value (in Kbytes)
	  -t #  Number of threads or processes to use in throughput test
	  -T  Use POSIX pthreads for throughput tests
	  -u #  Upper limit on number of processes to run
	  -U  Mount point to remount between tests
	  -v  version information
	  -V #  Verify data pattern write/read
	  -w  Do not unlink temporary file
	  -W  Lock file when reading or writing
	  -x  Turn off stone-walling
	  -X filename  Write telemetry file. Contains lines with (offset reclen compute_time) in ascii
	  -y #  Set minimum record size (in Kbytes) for auto mode (or #m or #g)
	  -Y filename  Read telemetry file. Contains lines with (offset reclen compute_time) in ascii
	  -z  Used in conjunction with -a to test all possible record sizes
	  -Z  Enable mixing of mmap I/O and file I/O
	  -+E Use existing non-Iozone file for read-only testing
	  -+K Sony special. Manual control of test 8.
	  -+m Cluster_filename  Enable Cluster testing
	  -+d File I/O diagnostic mode. (To troubleshoot a broken file I/O subsystem)
	  -+u Enable CPU utilization output (Experimental)
	  -+x # Multiplier to use for incrementing file and record sizes
	  -+p # Percentage of mix to be reads
	  -+r Enable O_RSYNC|O_SYNC for all testing.
	  -+t Enable network performance test. Requires -+m
	  -+n No retests selected.
	  -+k Use constant aggregate data set size.
	  -+q Delay in seconds between tests.
	  -+l Enable record locking mode.
	  -+L Enable record locking mode, with shared file.
	  -+B Sequential mixed workload.
	  -+A # Enable madvise. 0 = normal, 1=random, 2=sequential 3=dontneed, 4=willneed
	  -+N Do not truncate existing files on sequential writes.
	  -+S # Dedup-able data is limited to sharing within each numerically identified file set
	  -+V Enable shared file. No locking.
	  -+X Enable short circuit mode for filesystem testing ONLY
	      ALL Results are NOT valid in this mode.
	  -+Z Enable old data set compatibility mode. WARNING.. Published
	      hacks may invalidate these results and generate bogus, high values for results.
	  -+w ## Percent of dedup-able data in buffers.
	  -+y ## Percent of dedup-able within & across files in buffers.
	  -+C ## Percent of dedup-able within & not across files in buffers.
	  -+H Hostname  Hostname of the PIT server.
	  -+P Service  Service of the PIT server.
	  -+z Enable latency histogram logging.


.. index:: arcstat
.. _arcstat:

arcstat
-------

Arcstat is a script that prints out ZFS
`ARC <https://en.wikipedia.org/wiki/Adaptive_replacement_cache>`__
statistics. Originally it was a perl script created by Sun. That perl
script was ported to FreeBSD and then ported as a Python script
for use on %brand%.

Watching ARC hits/misses and percentages shows how well the ZFS pool is
fetching from the ARC rather than using disk I/O. Ideally, there will be
as many things fetching from cache as possible. Keep the load in mind
while reviewing the stats. For random reads, expect a miss and having to
go to disk to fetch the data. For cached reads, expect it to pull out of
the cache and have a hit.

Like all cache systems, the ARC takes time to fill with data. This
means that it will have a lot of misses until the pool has been in use
for a while. If there continues to be lots of misses and high disk I/O
on cached reads, there is cause to investigate further and tune the
system.

The
`FreeBSD ZFS Tuning Guide <https://wiki.freebsd.org/ZFSTuningGuide>`__
provides some suggestions for commonly tuned :command:`sysctl` values.
It should be noted that performance tuning is more of an art than a
science and that any changes made will probably require several
iterations of tune and test. Be aware that what needs to be tuned will
vary depending upon the type of workload and that what works for one
one network may not benefit another.

In particular, the value of pre-fetching depends upon the amount of
memory and the type of workload, as seen in
`Understanding ZFS: Prefetch
<http://cuddletech.com/?page_id=834&id=1040>`__

%brand% provides two command line scripts which can be manually run
from :ref:`Shell`:

* :command:`arc_summary.py`: provides a summary of the statistics

* :command:`arcstat.py`: used to watch the statistics in real time

The advantage of these scripts is that they provide
real time information, whereas the current |web-ui| reporting
mechanism is designed to only provide graphs charted over time.

This `forum post
<https://forums.freenas.org/index.php?threads/benchmarking-zfs.7928/>`__
demonstrates some examples of using these scripts with hints on how to
interpret the results.

To view the help for arcstat.py:

.. code-block:: none

    arcstat.py -h
    [-havxp] [-f fields] [-o file] [-s string] [interval [count]]

         -h : Print this help message
         -a : Print all possible stats
         -v : List all possible field headers and definitions
         -x : Print extended stats
         -f : Specify specific fields to print (see -v)
         -o : Redirect output to the specified file
         -s : Override default field separator with custom character or string
         -p : Disable auto-scaling of numerical fields

    Examples:
        arcstat -o /tmp/a.log 2 10
        arcstat -s "," -o /tmp/a.log 2 10
        arcstat -v
        arcstat -f time,hit%,dh%,ph%,mh% 1

To view ARC statistics in real time, specify an interval and a count.
This command will display every 1 second for a count of five.

.. code-block:: none

   arcstat.py 1 5
       time  read  miss  miss%  dmis  dm%  pmis  pm%  mmis  mm%  arcsz     c
   06:19:03     7     0      0     0    0     0    0     0    0   153M  6.6G
   06:19:04   257     0      0     0    0     0    0     0    0   153M  6.6G
   06:19:05   193     0      0     0    0     0    0     0    0   153M  6.6G
   06:19:06   193     0      0     0    0     0    0     0    0   153M  6.6G
   06:19:07   255     0      0     0    0     0    0     0    0   153M  6.6G


:numref:`Table %s <cli_arcstat_columns_tab>`
briefly describes the columns in the output.


.. tabularcolumns:: |>{\RaggedRight}p{\dimexpr 0.12\linewidth-2\tabcolsep}
                    |>{\RaggedRight}p{\dimexpr 0.33\linewidth-2\tabcolsep}|

.. _cli_arcstat_columns_tab:

.. table:: arcstat Column Descriptions
   :class: longtable

   +---------------------+------------------------------------------+
   | Column              | Description                              |
   |                     |                                          |
   +=====================+==========================================+
   | read                | total ARC accesses/second                |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | miss                | ARC misses/second                        |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | miss%               | ARC miss percentage                      |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | dmis                | demand data misses/second                |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | dm%                 | demand data miss percentage              |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | pmis                | prefetch misses per second               |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | pm%                 | prefetch miss percentage                 |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | mmis                | metadata misses/second                   |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | mm%                 | metadata miss percentage                 |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | arcsz               | arc size                                 |
   |                     |                                          |
   +---------------------+------------------------------------------+
   | c                   | arc target size                          |
   |                     |                                          |
   +---------------------+------------------------------------------+


To receive a summary of statistics, use:

.. code-block:: none

 arcsummary.py
 System Memory:
        2.36%   93.40   MiB Active,     8.95%   353.43  MiB Inact
        8.38%   330.89  MiB Wired,      0.15%   5.90    MiB Cache
        80.16%  3.09    GiB Free,       0.00%   0       Bytes Gap
        Real Installed:                         4.00    GiB
        Real Available:                 99.31%  3.97    GiB
        Real Managed:                   97.10%  3.86    GiB
        Logical Total:                          4.00    GiB
        Logical Used:                   13.93%  570.77  MiB
        Logical Free:                   86.07%  3.44    GiB
 Kernel Memory:                                 87.62   MiB
        Data:                           69.91%  61.25   MiB
        Text:                           30.09%  26.37   MiB
 Kernel Memory Map:                             3.86    GiB
        Size:                           5.11%   201.70  MiB
        Free:                           94.89%  3.66    GiB
 ARC Summary: (HEALTHY)
        Storage pool Version:                   5000
        Filesystem Version:                     5
        Memory Throttle Count:                  0
 ARC Misc:
        Deleted:                                8
        Mutex Misses:                           0
        Evict Skips:                            0
 ARC Size:                               5.83%   170.45  MiB
        Target Size: (Adaptive)         100.00% 2.86    GiB
        Min Size (Hard Limit):          12.50%  365.69  MiB
        Max Size (High Water):          8:1     2.86    GiB
 ARC Size Breakdown:
        Recently Used Cache Size:       50.00%  1.43    GiB
        Frequently Used Cache Size:     50.00%  1.43    GiB
 ARC Hash Breakdown:
        Elements Max:                           5.90k
        Elements Current:               100.00% 5.90k
        Collisions:                             72
        Chain Max:                              1
        Chains:                                 23
 ARC Total accesses:                                    954.06k
        Cache Hit Ratio:                99.18%  946.25k
        Cache Miss Ratio:               0.82%   7.81k
        Actual Hit Ratio:               98.84%  943.00k
        Data Demand Efficiency:         99.20%  458.77k
        CACHE HITS BY CACHE LIST:
          Anonymously Used:             0.34%   3.25k
          Most Recently Used:           3.73%   35.33k
          Most Frequently Used:         95.92%  907.67k
          Most Recently Used Ghost:     0.00%   0
          Most Frequently Used Ghost:   0.00%   0
        CACHE HITS BY DATA TYPE:
          Demand Data:                  48.10%  455.10k
          Prefetch Data:                0.00%   0
          Demand Metadata:              51.56%  487.90k
          Prefetch Metadata:            0.34%   3.25k
        CACHE MISSES BY DATA TYPE:
          Demand Data:                  46.93%  3.66k
          Prefetch Data:                0.00%   0
          Demand Metadata:              49.76%  3.88k
          Prefetch Metadata:            3.30%   258
 ZFS Tunable (sysctl):
        kern.maxusers                           590
        vm.kmem_size                            4141375488
        vm.kmem_size_scale                      1
        vm.kmem_size_min                        0
        vm.kmem_size_max                        1319413950874
        vfs.zfs.vol.unmap_enabled               1
        vfs.zfs.vol.mode                        2
        vfs.zfs.sync_pass_rewrite               2
        vfs.zfs.sync_pass_dont_compress         5
        vfs.zfs.sync_pass_deferred_free         2
        vfs.zfs.zio.exclude_metadata            0
        vfs.zfs.zio.use_uma                     1
        vfs.zfs.cache_flush_disable             0
        vfs.zfs.zil_replay_disable              0
        vfs.zfs.version.zpl                     5
        vfs.zfs.version.spa                     5000
        vfs.zfs.version.acl                     1
        vfs.zfs.version.ioctl                   5
        vfs.zfs.debug                           0
        vfs.zfs.super_owner                     0
        vfs.zfs.min_auto_ashift                 9
        vfs.zfs.max_auto_ashift                 13
        vfs.zfs.vdev.write_gap_limit            4096
        vfs.zfs.vdev.read_gap_limit             32768
        vfs.zfs.vdev.aggregation_limit          131072
        vfs.zfs.vdev.trim_max_active            64
        vfs.zfs.vdev.trim_min_active            1
        vfs.zfs.vdev.scrub_max_active           2
        vfs.zfs.vdev.scrub_min_active           1
        vfs.zfs.vdev.async_write_max_active     10
        vfs.zfs.vdev.async_write_min_active     1
        vfs.zfs.vdev.async_read_max_active      3
        vfs.zfs.vdev.async_read_min_active      1
        vfs.zfs.vdev.sync_write_max_active      10
        vfs.zfs.vdev.sync_write_min_active      10
        vfs.zfs.vdev.sync_read_max_active       10
        vfs.zfs.vdev.sync_read_min_active       10
        vfs.zfs.vdev.max_active                 1000
        vfs.zfs.vdev.async_write_active_max_dirty_percent60
        vfs.zfs.vdev.async_write_active_min_dirty_percent30
        vfs.zfs.vdev.mirror.non_rotating_seek_inc1
        vfs.zfs.vdev.mirror.non_rotating_inc    0
        vfs.zfs.vdev.mirror.rotating_seek_offset1048576
        vfs.zfs.vdev.mirror.rotating_seek_inc   5
        vfs.zfs.vdev.mirror.rotating_inc        0
        vfs.zfs.vdev.trim_on_init               1
        vfs.zfs.vdev.larger_ashift_minimal      0
        vfs.zfs.vdev.bio_delete_disable         0
        vfs.zfs.vdev.bio_flush_disable          0
        vfs.zfs.vdev.cache.bshift               16
        vfs.zfs.vdev.cache.size                 0
        vfs.zfs.vdev.cache.max                  16384
        vfs.zfs.vdev.metaslabs_per_vdev         200
        vfs.zfs.vdev.trim_max_pending           10000
        vfs.zfs.txg.timeout                     5
        vfs.zfs.trim.enabled                    1
        vfs.zfs.trim.max_interval               1
        vfs.zfs.trim.timeout                    30
        vfs.zfs.trim.txg_delay                  32
        vfs.zfs.space_map_blksz                 4096
        vfs.zfs.spa_slop_shift                  5
        vfs.zfs.spa_asize_inflation             24
        vfs.zfs.deadman_enabled                 1
        vfs.zfs.deadman_checktime_ms            5000
        vfs.zfs.deadman_synctime_ms             1000000
        vfs.zfs.recover                         0
        vfs.zfs.spa_load_verify_data            1
        vfs.zfs.spa_load_verify_metadata        1
        vfs.zfs.spa_load_verify_maxinflight     10000
        vfs.zfs.check_hostid                    1
        vfs.zfs.mg_fragmentation_threshold      85
        vfs.zfs.mg_noalloc_threshold            0
        vfs.zfs.condense_pct                    200
        vfs.zfs.metaslab.bias_enabled           1
        vfs.zfs.metaslab.lba_weighting_enabled  1
        vfs.zfs.metaslab.fragmentation_factor_enabled1
        vfs.zfs.metaslab.preload_enabled        1
        vfs.zfs.metaslab.preload_limit          3
        vfs.zfs.metaslab.unload_delay           8
        vfs.zfs.metaslab.load_pct               50
        vfs.zfs.metaslab.min_alloc_size         33554432
        vfs.zfs.metaslab.df_free_pct            4
        vfs.zfs.metaslab.df_alloc_threshold     131072
        vfs.zfs.metaslab.debug_unload           0
        vfs.zfs.metaslab.debug_load             0
        vfs.zfs.metaslab.fragmentation_threshold70
        vfs.zfs.metaslab.gang_bang              16777217
        vfs.zfs.free_bpobj_enabled              1
        vfs.zfs.free_max_blocks                 18446744073709551615
        vfs.zfs.no_scrub_prefetch               0
        vfs.zfs.no_scrub_io                     0
        vfs.zfs.resilver_min_time_ms            3000
        vfs.zfs.free_min_time_ms                1000
        vfs.zfs.scan_min_time_ms                1000
        vfs.zfs.scan_idle                       50
        vfs.zfs.scrub_delay                     4
        vfs.zfs.resilver_delay                  2
        vfs.zfs.top_maxinflight                 32
        vfs.zfs.delay_scale                     500000
        vfs.zfs.delay_min_dirty_percent         60
        vfs.zfs.dirty_data_sync                 67108864
        vfs.zfs.dirty_data_max_percent          10
        vfs.zfs.dirty_data_max_max              4294967296
        vfs.zfs.dirty_data_max                  426512793
        vfs.zfs.max_recordsize                  1048576
        vfs.zfs.zfetch.array_rd_sz              1048576
        vfs.zfs.zfetch.max_distance             8388608
        vfs.zfs.zfetch.min_sec_reap             2
        vfs.zfs.zfetch.max_streams              8
        vfs.zfs.prefetch_disable                1
        vfs.zfs.mdcomp_disable                  0
        vfs.zfs.nopwrite_enabled                1
        vfs.zfs.dedup.prefetch                  1
        vfs.zfs.l2c_only_size                   0
        vfs.zfs.mfu_ghost_data_lsize            0
        vfs.zfs.mfu_ghost_metadata_lsize        0
        vfs.zfs.mfu_ghost_size                  0
        vfs.zfs.mfu_data_lsize                  26300416
        vfs.zfs.mfu_metadata_lsize              1780736
        vfs.zfs.mfu_size                        29428736
        vfs.zfs.mru_ghost_data_lsize            0
        vfs.zfs.mru_ghost_metadata_lsize        0
        vfs.zfs.mru_ghost_size                  0
        vfs.zfs.mru_data_lsize                  122090496
        vfs.zfs.mru_metadata_lsize              2235904
        vfs.zfs.mru_size                        139389440
        vfs.zfs.anon_data_lsize                 0
        vfs.zfs.anon_metadata_lsize             0
        vfs.zfs.anon_size                       163840
        vfs.zfs.l2arc_norw                      1
        vfs.zfs.l2arc_feed_again                1
        vfs.zfs.l2arc_noprefetch                1
        vfs.zfs.l2arc_feed_min_ms               200
        vfs.zfs.l2arc_feed_secs                 1
        vfs.zfs.l2arc_headroom                  2
        vfs.zfs.l2arc_write_boost               8388608
        vfs.zfs.l2arc_write_max                 8388608
        vfs.zfs.arc_meta_limit                  766908416
        vfs.zfs.arc_free_target                 7062
        vfs.zfs.arc_shrink_shift                7
        vfs.zfs.arc_average_blocksize           8192
        vfs.zfs.arc_min                         383454208
        vfs.zfs.arc_max                         3067633664


When reading the tunable values, 0 means no, 1 typically means yes,
and any other number represents a value. To receive a brief
description of a "sysctl" value, use :command:`sysctl -d`. For
example:

.. code-block:: none

   sysctl -d vfs.zfs.zio.use_uma
   vfs.zfs.zio.use_uma: Use uma(9) for ZIO allocations


The ZFS tunables require a fair understanding of how ZFS works,
meaning that reading man pages and searching for the
meaning of unfamiliar acronyms is required.
**Do not change a tunable's value without researching it first.**
If the tunable takes a numeric value (rather than 0 for no or 1 for
yes), do not make one up. Instead, research examples of beneficial
values that match the workload.

If any of the ZFS tunables are changed, continue to monitor
the system to determine the effect of the change. It is recommended
that the changes are tested first at the command line using
:command:`sysctl`. For example, to disable pre-fetch (i.e. change
disable to *1* or yes):

.. code-block:: none

   sysctl vfs.zfs.prefetch_disable=1
   vfs.zfs.prefetch_disable: 0 -> 1


The output will indicate the old value followed by the new value. If
the change is not beneficial, change it back to the original value. If
the change turns out to be beneficial, it can be made permanent by
creating a *sysctl* using the instructions in :ref:`Tunables`.


.. index:: tw_cli
.. _tw_cli:

tw_cli
------

%brand% includes the :command:`tw_cli` command line utility for
providing controller, logical unit, and drive management for
AMCC/3ware ATA RAID Controllers. The supported models are listed in
the man pages for the
`twe(4) <https://www.freebsd.org/cgi/man.cgi?query=twe>`__
and
`twa(4) <https://www.freebsd.org/cgi/man.cgi?query=twa>`__
drivers.

Before using this command, read its
`man page <https://www.cyberciti.biz/files/tw_cli.8.html>`__
as it describes the terminology and provides some usage examples.

When :command:`tw_cli` in Shell is entered, the prompt will change,
indicating that interactive mode is enabled where
all sorts of maintenance commands on the controller and its arrays
can be run.

Alternately, one command can be specified to run. For example, to view
the disks in the array:

.. code-block:: none

 tw_cli /c0 show
 Unit	UnitType	Status	%RCmpl	%V/I/M	Stripe	Size(GB)	Cache   AVrfy
 ------------------------------------------------------------------------------
 u0	RAID-6		OK	-	-	256K	5587.88		RiW	ON
 u1	SPARE		OK	-	-	-	931.505		-	OFF
 u2	RAID-10		OK	-	-	256K	1862.62		RiW	ON

 VPort Status	Unit    Size		Type	Phy Encl-Slot	Model
 ------------------------------------------------------------------------------
 p8	OK	u0	931.51 GB SAS	-	/c0/e0/slt0	SEAGATE ST31000640SS
 p9	OK	u0	931.51 GB SAS	-	/c0/e0/slt1	SEAGATE ST31000640SS
 p10	OK	u0	931.51 GB SAS	-	/c0/e0/slt2	SEAGATE ST31000640SS
 p11	OK	u0	931.51 GB SAS	-	/c0/e0/slt3	SEAGATE ST31000640SS
 p12	OK	u0	931.51 GB SAS	-	/c0/e0/slt4	SEAGATE ST31000640SS
 p13	OK	u0	931.51 GB SAS	-	/c0/e0/slt5	SEAGATE ST31000640SS
 p14	OK	u0	931.51 GB SAS	-	/c0/e0/slt6	SEAGATE ST31000640SS
 p15	OK	u0	931.51 GB SAS	-	/c0/e0/slt7	SEAGATE ST31000640SS
 p16	OK	u1	931.51 GB SAS	-	/c0/e0/slt8	SEAGATE ST31000640SS
 p17	OK	u2	931.51 GB SATA	-	/c0/e0/slt9	ST31000340NS
 p18	OK	u2	931.51 GB SATA	-	/c0/e0/slt10    ST31000340NS
 p19	OK	u2	931.51 GB SATA	-	/c0/e0/slt11    ST31000340NS
 p20	OK	u2	931.51 GB SATA	-	/c0/e0/slt15    ST31000340NS

 Name	OnlineState	BBUReady	Status	Volt	Temp	Hours   LastCapTest
 ---------------------------------------------------------------------------
 bbu	On		Yes		OK	OK	OK	212	03-Jan-2012


Or, to review the event log:

.. code-block:: none

 tw_cli /c0 show events
 Ctl	Date				Severity	AEN Message
 ------------------------------------------------------------------------------
 c0	[Thu Feb 23 2012 14:01:15]	INFO		Battery charging started
 c0	[Thu Feb 23 2012 14:03:02]	INFO		Battery charging completed
 c0	[Sat Feb 25 2012 00:02:18]	INFO		Verify started: unit=0
 c0	[Sat Feb 25 2012 00:02:18]	INFO		Verify started: unit=2,subunit=0
 c0	[Sat Feb 25 2012 00:02:18]	INFO		Verify started: unit=2,subunit=1
 c0	[Sat Feb 25 2012 03:49:35]	INFO		Verify completed: unit=2,subunit=0
 c0	[Sat Feb 25 2012 03:51:39]	INFO		Verify completed: unit=2,subunit=1
 c0	[Sat Feb 25 2012 21:55:59]	INFO		Verify completed: unit=0
 c0	[Thu Mar 01 2012 13:51:09]	INFO		Battery health check started
 c0	[Thu Mar 01 2012 13:51:09]	INFO		Battery health check completed
 c0	[Thu Mar 01 2012 13:51:09]	INFO		Battery charging started
 c0	[Thu Mar 01 2012 13:53:03]	INFO		Battery charging completed
 c0	[Sat Mar 03 2012 00:01:24]	INFO		Verify started: unit=0
 c0	[Sat Mar 03 2012 00:01:24]	INFO		Verify started: unit=2,subunit=0
 c0	[Sat Mar 03 2012 00:01:24]	INFO		Verify started: unit=2,subunit=1
 c0	[Sat Mar 03 2012 04:04:27]	INFO		Verify completed: unit=2,subunit=0
 c0	[Sat Mar 03 2012 04:06:25]	INFO		Verify completed: unit=2,subunit=1
 c0	[Sat Mar 03 2012 16:22:05]	INFO		Verify completed: unit=0
 c0	[Thu Mar 08 2012 13:41:39]	INFO		Battery charging started
 c0	[Thu Mar 08 2012 13:43:42]	INFO		Battery charging completed
 c0	[Sat Mar 10 2012 00:01:30]	INFO		Verify started: unit=0
 c0	[Sat Mar 10 2012 00:01:30]	INFO		Verify started: unit=2,subunit=0
 c0	[Sat Mar 10 2012 00:01:30]	INFO		Verify started: unit=2,subunit=1
 c0	[Sat Mar 10 2012 05:06:38]	INFO		Verify completed: unit=2,subunit=0
 c0	[Sat Mar 10 2012 05:08:57]	INFO		Verify completed: unit=2,subunit=1
 c0	[Sat Mar 10 2012 15:58:15]	INFO		Verify completed: unit=0


If the disks added to the array do not appear in the
|web-ui|, try running this command:

.. code-block:: none

   tw_cli /c0 rescan

Use the drives to create units and export them to the operating
system. When finished, run :command:`camcontrol rescan all` to make
them available in the %brand% |web-ui|.

This `forum post
<https://forums.freenas.org/index.php?threads/3ware-drive-monitoring.13835/>`__
contains a handy wrapper script that will give error notifications.


.. index:: MegaCli
.. _MegaCli:

MegaCli
-------

:command:`MegaCli` is the command line interface for the Broadcom
:MegaRAID SAS family of RAID controllers. %brand% also includes the
`mfiutil(8) <https://www.freebsd.org/cgi/man.cgi?query=mfiutil>`__
utility which can be used to configure and manage connected storage
devices.

The :command:`MegaCli` command is quite complex with several dozen
options. The commands demonstrated in the `Emergency Cheat Sheet
<http://tools.rapidsoft.de/perc/perc-cheat-sheet.html>`_ can get you
started.


.. index:: freenas-debug
.. _freenas-debug:

freenas-debug
-------------

The %brand% |web-ui| provides an option to save debugging information to a
text file using :menuselection:`System --> Advanced --> Save Debug`.
This debugging information is created by the :command:`freenas-debug`
command line utility and a copy of the information is saved to
:file:`/var/tmp/fndebug`.

This command can be run manually from :ref:`Shell` to gather specific
debugging information. To see a usage explanation listing all options,
run the command without any options:


.. code-block:: none

   freenas-debug
   Usage: /usr/local/bin/freenas-debug <options>
   Where options are:

    -A  Dump all debug information
    -B  Dump System Configuration Database
    -C  Dump SMB Configuration
    -D  Dump Domain Controller Configuration
    -I  Dump IPMI Configuration
    -M  Dump SATA DOMs Information
    -N  Dump NFS Configuration
    -S  Dump SMART Information
    -T  Loader Configuration Information
    -Z  Remove old debug information
    -a  Dump Active Directory Configuration
    -c  Dump (AD|LDAP) Cache
    -e  Email debug log to this comma-delimited list of email addresses
    -f  Dump AFP Configuration
    -g  Dump GEOM Configuration
    -h  Dump Hardware Configuration
    -i  Dump iSCSI Configuration
    -j  Dump Jail Information
    -l  Dump LDAP Configuration
    -n  Dump Network Configuration
    -s  Dump SSL Configuration
    -t  Dump System Information
    -v  Dump Boot System File Verification Status and Inconsistencies
    -y  Dump Sysctl Configuration
    -z  Dump ZFS Configuration


Individual tests can be run alone. For example, when troubleshooting
an Active Directory configuration, use:

.. code-block:: none

   freenas-debug -a


To collect the output of every module, use :literal:`-A`:

.. code-block:: none

   freenas-debug -A

For collecting debug information about a single volume, use
:command:`zdb` with :literal:`-U /data/zfs/zpool.cache`
followed by the name of the volume (ZFS pool):

.. code-block:: none

    zdb -U /data/zfs/zpool.cache volume1

See the 
`zdb(8) manual page <https://www.freebsd.org/cgi/man.cgi?query=zdb>`__
for more information.

.. index:: tmux
.. _tmux:

tmux
----

:command:`tmux` is a terminal multiplexer which enables a number of
:terminals to be created, accessed, and controlled from a single
:screen. :command:`tmux` is an alternative to GNU :command:`screen`.
Similar to screen, :command:`tmux` can be detached from a screen and
continue running in the background, then later reattached. Unlike
:ref:`Shell`, :command:`tmux` provides access to a command
prompt while still giving access to the graphical administration
screens.

To start a session, simply type :command:`tmux`. As seen in
:numref:`Figure %s <cli_tmux_fig>`,
a new session with a single window opens with a status line at the
bottom of the screen. This line shows information on the current
session and is used to enter interactive commands.


.. _cli_tmux_fig:

.. figure:: images/shell-tmux.png

   tmux Session


To create a second window, press :kbd:`Ctrl+b` then :kbd:`"`. To close
a window, type :command:`exit` within the window.

`tmux(1)
<http://man.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man1/tmux.1?query=tmux>`__
lists all of the key bindings and commands for interacting with
:command:`tmux` windows and sessions.

If :ref:`Shell` is closed while :command:`tmux` is running, it will
detach its session. The next time Shell is open, run
:command:`tmux attach` to return to the previous session. To leave the
:command:`tmux` session entirely, type :command:`exit`. If
multiple windows are running, it is required to :command:`exit` out
of each first.

These resources provide more information about using :command:`tmux`:

* `A tmux Crash Course
  <https://robots.thoughtbot.com/a-tmux-crash-course>`__

* `TMUX - The Terminal Multiplexer
  <http://blog.hawkhost.com/2010/06/28/tmux-the-terminal-multiplexer/>`__


.. index:: Dmidecode
.. _Dmidecode:

Dmidecode
---------

Dmidecode reports hardware information as reported by the system BIOS.
Dmidecode does not scan the hardware, it only reports what the BIOS
told it to. A sample output can be seen
`here <http://www.nongnu.org/dmidecode/sample/dmidecode.txt>`__.

To view the BIOS report, type the command with no arguments:

.. code-block:: none

   dmidecode | more


`dmidecode(8) <https://linux.die.net/man/8/dmidecode>`__
describes the supported strings and types.


.. index:: Midnight Commander
.. _Midnight_Commander:

Midnight Commander
------------------

Midnight Commander is a program used to manage files from the shell.
Open the application by running :command:`mc`.
The arrow keys are used to navigate and select files. Function keys are
used to perform operations such as renaming, editing, and copying files.
These resources provide more information about using Midnight Commander:

* `Midnight Commander wikipedia page <https://en.wikipedia.org/wiki/Midnight_Commander>`__

* `Midnight Commander website <https://midnight-commander.org/>`__

* `mc(1) <https://www.freebsd.org/cgi/man.cgi?query=mc>`__

* `Basic Tutorial <http://linuxcommand.org/lc3_adv_mc.php>`__
