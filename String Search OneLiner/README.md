## Motivation

Performing a string search on a big disk image can lead to a huge amount of findings.
As a result you usually get the byte offset and the line of strings for the hit. In
the next step you may like to analyze the context around each hit. For example, you
like to analyze all the strings found in the corresponding clusters.

To analyze all the strings in all this clusters manually one by one could become time
consuming and much work. The idea is to have a bash OneLiner, to support this activity.

## Example

In this example I will use a very small disk image to illustrate the problem. So I 
created a disk image ```sample.raw``` for investigation. I have to search the string 
*evidence* and analyze the context it is found. Here the steps how to proceed:

### 1. Read the partition table

I read the partition table to understand the partition layout of the disk.

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

### Do the sting search

Now I search the complete partition for our search string *evidence*. 
As a result I also like to get the offset of the hit in byte and store
the findings in the file ```evidence.txt```

Command:
~~~
$ blkls -e -o 2048 sample.raw | strings -a -td | grep -i evidence | tee -a evidence.txt
~~~
Output:
~~~
    82330 example test blah evidence blah blah example
    82438 example 12345 moreEvidence 67890 example
    82542 test text evidences test text
    83320     evidence
104222721  more more evidence more more evidence more more
120442931  more more evidence more more evidence more more
~~~

So I got 6 hits in total with different byte offsets.

### Further investigation

For doing the further investigation, for each byte offset of the 6 findings I have to do the following steps:

1. Read the file ```evidence.txt``` line by line
2. For each line extract the byte offset and calculate the corresponding cluster number.
3. Sort the cluster numbers and remove duplicates
4. Read the raw clusters from the disk image and extract all the strings

All this steps could be realized with the following command line:

Command:
~~~
$ cat evidence.txt | while read x; do echo $(( `echo $x | cut -f1 -d" "` / 4096 )); done | sort | uniq | xargs -I {} sh -c 'echo "----- Start Cluster {} -----"; blkcat -o 2048 sample.raw {}; echo "----- End Cluster {} -----"' | strings | less
~~~

Output (cut):
~~~
----- Start Cluster 20 -----
FILE0
11111111111111111111
22222222222222222222
example test blah evidence blah blah example
44444444444444444444
55555555555555555555
6666666666666
66666
example 12345 moreEvidence 67890 example
77777777777777777777
88888888888888888888
99999999999999999999
test text evidences test text
00000000000000000000 
FILE0
abc abc
    evidence
xyz xyz
FILE0
----- End Cluster 20 -----
----- Start Cluster 25445 -----
 more more evidence more more evidence more more
.....
.....
.....
~~~

Now I can review the strings surrounding my search string *evidence* within the 4096 bytes cluster 
and decide, if the cluster could be relevant for my investigation.


