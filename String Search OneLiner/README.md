## Motivation

Performing a string search on a big disk image can lead to a huge amount of findings.
As a result you usually get the byte offset and the line of strings for the hit. In
the next step you may like to analyze the context arround the hit, like all the strings
found in the corresponding cluster.

To investigate the strings of this all this clusters manually one by one could become
time consuming and much work. The idea is to have a bash oneliner, to support this activity.

## Example

In this example I will use a very small disk image to illustrate the problem. So I receive 
a disk image "sample.raw" for investigation. I have to search the string _evidence_ and
the context it is found. Here the steps how to proceed:

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

Now I search the complete partition for our search string _evidence_. 
As a resut I also like to get the offset of the hit in byte and store
the findings in the file ```evidence.txt```

Command:
~~~
blkls -e -o 2048 sample.raw | strings -a -td | grep -i evidence | tee -a evidence.txt
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

Doing the further investigation manually, for each of the 6 byte offsets,
I have to investigate like this:
1. Calculate the cluster number for the byte offset
2. Is this a new cluster number? Did I got it already before, than ignore it
3. Display all strings of this cluster

### OneLiner do do all this work

The OneLiner do the following:
1. Read the resultat out of the file: ```evidence.txt```
2. For each line extract the byte offset and calculate the cluster number.
3. Sort the cluster numbers and remove duplicates
4. Read the clusters and search the strings

Command:
~~~
cat evidence.txt | while read x; do echo $(( `echo $x | cut -f1 -d" "` / 4096 )); done | sort | uniq | xargs -I {} blkcat -o 2048 sample.raw {} | strings | less
~~~







