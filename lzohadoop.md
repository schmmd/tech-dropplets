# LZO Compression in Hadoop
- Michael Schmitz
- schmmd
- 2011/09/19
- Hadoop
- published

While Hadoop does not provide an innovative way of looking at parallelizing
computation, it does provide a relatively easy-to-use and free-as-in-beer
platform to parallelize tasks across multiple machines. For me, Hadoop's
primary advantage is HDFS--the distributed file system. Many organizations want
to manage large files which exceed the storage capabilities of a single
machine.

Small organizations may need to process large volumes of data but are not able to buy a large cluster. Using compression on a cluster is a great way of extending its capabilities. Hadoop 0.20.x provides a few CODECs for compression. They are stored in `org.apache.hadoop.io.compress`.

* `BZip2Codec`
* `DefaultCodec`
* `GzipCodec`

`DefaultCodec` is an implementation of <a href="http://en.wikipedia.org/wiki/DEFLATE">`deflate`</a>, the other two should be self-explanatory.

I do not believe that any of these CODECs are splittable in Hadoop 0.20.x, although the <a href="https://issues.apache.org/jira/browse/HADOOP-4012">ability to split BZIP2 files was added in 0.21</a>. However, even if you are running 0.21, `BZIP2` compression is **slow**. If a compressed file is not splittable, the entire file must go to a single map process. This can cause a few problems.

1. If you don't have enough files, some of your mappers will be idle.
2. You lose advantages of data-locality. A file must move to a single node, instead of mapper processing local blocks.

An alternative, LZO, used to be packaged with Hadoop, but was decoupled due to licensing constraints. Now you need to install LZO separately, which you can do by following <a href="https://github.com/kevinweil/hadoop-lzo/blob/master/README.md">Kevin Weil's tutorial</a> for his <a href="https://github.com/kevinweil/hadoop-lzo">GitHub project</a>. Once installed and properly configured (which can take some work) you will have two new CODECs.

* `com.hadoop.compression.lzo.LzoCodec`
* `com.hadoop.compression.lzo.LzopCodec`

Why are there two different CODECs that look so similar? `LzoCodec` (no 'p') does not add headers, whereas the `LzopCodec` does. When storing files in HDFS, you will probably want to use `LzopCodec`--the only one compatible with the <a href="http://www.lzop.org/">command-line utility `lzop`</a>.

This seems like a lot of work. Why use LZO? It turns out LZO is <a href="http://aliver.wordpress.com/2010/06/22/huge-unix-file-compresser-shootout-with-tons-of-datagraphs/">**much faster** than competing compression algorithms</a>. In fact, LZO is fast to the point that it can speed up some Hadoop tasks by reducing the amount of I/O. While it is not a great compression algorithm in terms of compression ratio, it's real time speed makes it ideal for Hadoop. And the compression isn't bad. In my experience, files often get reduced to 1/3 the original size. On our small 6-node "cluster" this increases the disk space from 24 TB to 72 TB **AND** improves the speed of I/O intensive jobs, without paying an extra penny!
