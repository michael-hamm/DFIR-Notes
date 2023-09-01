## Motivation

Performing a string search on a big disk image can lead to a huge amount of findings.
As a result you usually get the byte offset and the line of the hit. To investigate the
context surrounding the hit manually one by one is time complex and time consuming.
The idea is to have a bash oneliner, to support this task.

## Example

In this example I will use a very small disk image to illustrate the problem. So I
receive a disk image "sample.raw" for investigation. Here the steps how to proceed:

### 1. Read the partition table

I read the partition table to understand the partition level layout of the disk.

Command:
~~~
$ mmls sample.raw
~~~
Output:
~~~
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000511999   0000509952   NTFS / exFAT (0x07)
~~~

So there seems to be a NTFS partition starting at sector 2048.

### 2. Identify cluster size of the NTFS file system

In the next step I have to identify the cluster size of the file system.

Command:
~~~
$ fsstat -o 2048 sample.raw
~~~
Output (cut):
~~~
Sector Size: 512
Cluster Size: 4096
Total Cluster Range: 0 - 63742
Total Sector Range: 0 - 509950
~~~

The cluster size is obviously 4096 bytes per cluster.


