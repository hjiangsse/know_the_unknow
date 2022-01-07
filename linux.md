# Use dd and hdparm to measure device read/write performance:
## Write speed test
``` bash
dd if=/dev/zero of=/tmp/test1.img bs=1G count=1 oflag=dsync
```
* if: the input file, /dev/zero is a special file in Unix-like operating systems that provides as many null characters (ASCII NUL, 0x00) as are read from it.
* if: the output file.
* bs: the size of the block you want *dd* to use(make sure your have that space in RAM).
* count: the number of block you want *dd* to read.
* oflag=dsync: use synchronized I/O for the data copy.(Skip the copying and give you the right answer.)

Run the command:
```
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 2.32698 s, 461 MB/s
```

## Read speed test
``` bash
 sudo hdparm -Ttv /dev/sda1
```
run the command and get the result:
```
/dev/sda1:
SG_IO: bad/missing sense data, sb[]:  70 00 05 00 00 00 00 0d 00 00 00 00 20 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
 multcount     =  0 (off)
 readonly      =  0 (off)
 readahead     = 256 (on)
 geometry      = 13992/255/63, sectors = 1048576, start = 2048
 Timing cached reads:   19038 MB in  1.99 seconds = 9562.72 MB/sec
 Timing buffered disk reads: 512 MB in  0.67 seconds = 760.66 MB/sec
```
For meaningful results, this operation should be repeated 2-3 times on an otherwise 
inactive system (no other active processes) with at least a couple of megabytes of free memory.


