HDFS 在设计之初主要解决了两个问题，一是相对廉价的分布式解决了存储可扩展性的问题，二是分析性能比较均衡，支持用户在此基础上做很多创新来解决性能问题。

大数据技术栈总共包含有四层，分别是资源调度层、统一的分布式块存储管理层、统一的计算引擎层和统一的接口层，所以下一代的大数据技术一定是基于这四层进行改造，以适应新的应用场景和需求。

## 支持的文件格式

Hadoop中的文件格式大致上分为**面向行和面向列**两类：

面向行：同一行的数据存储在一起，即连续存储。SequenceFile、MapFile、Avro Datafile。采用这种方式，如果只需要访问行的一小部分数据，亦需要将整行读入内存，推迟序列化一定程度上可以缓解这个问题，但是从磁盘读取整行数据的开销却无法避免。面向行的存储适合于整行数据需要同时处理的情况。

面向列：整个文件被切割为若干列数据，每一列数据一起存储。Parquet 、 RCFile、ORCFile。面向列的格式使得读取数据时，可以跳过不需要的列，适合于只处于行的一小部分字段的情况。但是这种格式的读写需要更多的内存空间，因为需要缓存行在内存中（为了获取多行中的某一列）。同时不适合流式写入，因为一旦写入失败，当前文件无法恢复，而面向行的数据在写入失败时可以重新同步到最后一个同步点，所以Flume采用的是面向行的存储格式。

### sequencefile

考虑日志文件，其中每一行文本代表一条日志记录。纯文本不合适记录二进制类型的数据。在这种情况下，Hadoop的**SequenceFile**类非常合适， 为二进制键/值对提供了一个持久数据结构。将它作为日志文件的存储格式时，你可以自己选择键(比如**LongWritable**类型所表示的时间戳)，以及值可以是**Writable**类型(用于表示日志记录的数量)。

在hadoop中块大小默认是128m，小于该值的文件都属于小文件，通过**SequenceFile**类型将小文件包装起来，可以获得更高效率的存储和处理。

#### 优势

- 支持基于记录(Record)或块(Block)的数据压缩。
- 支持splitable，能够作为MapReduce的输入分片。
- 修改简单：主要负责修改相应的业务逻辑，而不用考虑具体的存储格式。
- spark支持该文件类型

#### 缺点

- 需要一个合并文件的过程，且合并后的文件不方便查看

### SequenceFile支持数据压缩

A.无压缩类型：如果没有启用压缩(默认设置)那么每个记录就由它的记录长度(字节数)、键的长度，键和值组成。长度字段为4字节。

B.记录压缩类型：记录压缩格式与无压缩格式基本相同，不同的是值字节是用定义在头部的编码器来压缩。注意：键是不压缩的。下图为记录压缩：

![img](https://ask.qcloudimg.com/http-save/yehe-1410343/nxgpfb5zqi.jpeg?imageView2/2/w/1620)

C.块压缩类型：块压缩一次压缩多个记录，因此它比记录压缩更紧凑，而且一般优先选择。当记录的字节数达到最小大小，才会添加到块。该最小值由io.seqfile.compress.blocksize中的属性定义。默认值是1000000字节。格式为记录数、键长度、键、值长度、值。下图为块压缩：

![img](https://ask.qcloudimg.com/http-save/yehe-1410343/h5vs0jztyf.jpeg?imageView2/2/w/1620)

### SequenceFile的排序和合并

MapReduce是对多个顺序文件进行排序(或合并)最有效的方法。MapReduce 本身是并行的，并且可由你指定要使用多少个reducer(该数决定着输出分区数)。除了通过MapReduce实现排序/归并，还有一种方法是使用SequenceFile.Sorter类中的sort()方法和merge()方法。它们比MapReduce更早出现，比MapReduce更底层(例如，为了实现并行，你需要手动对数据进行分区），所 以对顺序文件进行排序合并时MapReduce。

```shell
% hadoop jar $HADOOP_INSTALL/hadoop-*-examples.jar sort -r 1 \
-inFormat org.apache.hadoop.mapred.SequenceFilelnputFormat \
-outFormat org.apache.hadoop.mapred.SequenceFileOutputFormat \
-outKey org.apache.hadoop.io.IntWritable \
-outvalue org.apache.hadoop.io.Text \
numbers.seq sorted
```

### SequenceFile的写操作

```java
public class SequenceFileWriteDemo {
    private static final String[] DATA = {
        "One, two, buckle my shoe",
        "Three, four, shut the door",
        "Five, six, pick up sticks",
        "Seven, eight, lay them straight",
        "Nine, ten, a big fat hen"
    };
    public static void main(String[] args) throws IOException {
        String uri = args[0];
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(uri), conf);
        Path path = new Path(uri);
        IntWritable key = new IntWritable();
        Text value = new Text();
        SequenceFile.Writer writer = null;
        try {
            writer = SequenceFile.createWriter(fs, conf, path, key.getClass(), value.getClass());
            for (int i = 0; i < 100; i++) {
                key.set(100 - i);
                value.set(DATA[i % DATA.length]);
                System.out.printf("[%s]\t%s\t%s\n", writer.getLength(), key, value);
                writer.append(key, value);
            }
        } finally {
            IOUtils.closestream(writer);
        }
    }
}
```

结果：

```shell
% hadoop SequenceFileMriteDemo numbers.seq
[128] 100 One, two,buckle my shoe
[173] 99 Three, four, shut the door
[220] 98 Five, six, pick up sticks
[264] 97 Seven, eight, lay them straight
[314] 96 Nine, ten, a big fat hen
[359] 95 One, two, buckle my shoe
[404] 94 Three, four, shut the door
[451] 93 Five, six, pick up sticks
[495] 92 Seven, eight, lay them straight
[545] 91 Nine, ten, a big fat hen
...
[1976] 60 One, two, buckle my shoe
[2021] 59 Three, four, shut the door
[2088] 58 Five, six, pick up sticks
[2132] 57 Seven, eight, lay them straight
[2182] 56 Nine, ten, a big fat hen
...
[4557] 5 One, two, buckle my shoe
[4602] 4 Three, four, shut the door
[4649] 3 Five, six, pick up sticks
[4693] 2 Seven, eighty lay them straight
[4743] 1 Nine, ten, a big fat hen
```

### SequenceFile 的读操作

```java
    public static void main(String[] args) throws IOException {
        String uri = args[0];
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(uri), conf);
        Path path = new Path(uri);
        SequenceFile.Reader reader = null;
        try {
            reader = new SequenceFile.Reader(fs, path, conf);
            Writable key = (Writable)ReflectionUtils.newlnstance(reader.getKeyClass(), conf);
            Writable value = (Writable)ReflectionUtils.newlnstance(reader.getValueClass(), conf);
            long position = reader.getPosition();
            while (reader.next(key, value)) {
                String syncSeen = reader. syncSeen() ? "*":"";
                System.out.printf("[%s%s]\t%s\t%s\n", position, syncSeen, key,value);
                position = reader.getPosition(); // beginning of next record
            }
        } finally {
            IOUtils.closeStream(reader);
        }
    }
}
```

该程序的另一个特性是能够显示顺序文件中同步点的位置信息。所谓同步点， 是指数据读取的实例出错后能够再一次与记录边界同步的数据流中的一个位 置，例如，在数据流中搜索到任意位置后。同步点是由**SequenceFile**.**Writer**记录的，后者在顺序文件写入过程中插入一个特殊项以便每隔几个记录便有 一个同步标识。这样的特殊项非常小，因而只造成很小的存储开销，不到 1%。同步点始终位于记录的边界处。

在顺序文件中搜索给定位置有两种方法。第一种是调用**seek**()方法，该方法将读指针指向文件中指定的位置。例如，可以按如下方式搜査记录边界：

```java
reader.seek(359);
assertThat(reader.next(key, value), is(true));
assertThat(((IntWritable) key).get(), is(95));
```

但如果给定位置不是记录边界，调用**next**()方法时就会出错：

```java
reader.seek(360);
reader.next(key, value); // fails with IOException
```

第二种方法通过同步点査找记录边界。**SequenceFile**.**Reader**对象的 **sync(long** **position**)方法可以将读取位置定位到**position**之后的下一个同步点。如果**position**之后没有同步了，那么当前读取位置将指向文 件末尾。这样，我们对数据流中的任意位置调用**sync**()方法(例如非记录 边界)而且可以重新定位到下一个同步点并继续向后读取：

```java
reader.sync(360);
assertThat(reader.getPosition(), is(2021L));
assertThat(reader.next(key, value), is(true));
assertThat(((IntWritable) key).get(), is(59));
```

SequenceFile.Writer对象有一个sync()方法，该方法可以在数据流的当前位置插入一个同步点。不要把它和同名的Syncable接口中定义的sync()方法混为一谈，后者用于底层设备缓冲区的同步。

可以将加入同步点的顺序文件作为**MapReduce**的输入，因为该类顺序文件允许切分，由此该文件的不同部分可以由独立的**map**任务单独处理。

### 通过命令行接口显示SequenceFile

```shell
% hadoop fs -text numbers.seq | head
100 One, two, buckle my shoe
99 Three,four, shut the door
98 Five,six, pick up sticks
97 Seven,eight, lay them straight
96 Nine,ten^ a big fat hen
95 One, two, buckle my shoe
94 Three,four, shut the door
93 Five,six, pick up sticks
92 Seven,eight, lay them straight
91 Nine,ten, a big fat hen
```

### mapreduce合并小文件成sequencefile

```java
import java.io.IOException;
 
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
 
public class WholeFileInputFormat extends
		FileInputFormat<NullWritable, BytesWritable> {
	//设置每个小文件不可分片,保证一个小文件生成一个key-value键值对
	@Override
	protected boolean isSplitable(JobContext context, Path file) {
		return false;
	}
 
	@Override
	public RecordReader<NullWritable, BytesWritable> createRecordReader(
			InputSplit split, TaskAttemptContext context) throws IOException,
			InterruptedException {
		WholeFileRecordReader reader = new WholeFileRecordReader();
		reader.initialize(split, context);
		return reader;
	}

```

```java

import java.io.IOException;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
 
class WholeFileRecordReader extends RecordReader<NullWritable, BytesWritable> {
	private FileSplit fileSplit;
	private Configuration conf;
	private BytesWritable value = new BytesWritable();
	private boolean processed = false;
 
	@Override
	public void initialize(InputSplit split, TaskAttemptContext context)
			throws IOException, InterruptedException {
		this.fileSplit = (FileSplit) split;
		this.conf = context.getConfiguration();
	}
 
	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
		if (!processed) {
			byte[] contents = new byte[(int) fileSplit.getLength()];
			Path file = fileSplit.getPath();
			FileSystem fs = file.getFileSystem(conf);
			FSDataInputStream in = null;
			try {
				in = fs.open(file);
				IOUtils.readFully(in, contents, 0, contents.length);
				value.set(contents, 0, contents.length);
			} finally {
				IOUtils.closeStream(in);
			}
			processed = true;
			return true;
		}
		return false;
	}
 
	@Override
	public NullWritable getCurrentKey() throws IOException,
			InterruptedException {
		return NullWritable.get();
	}
 
	@Override
	public BytesWritable getCurrentValue() throws IOException,
			InterruptedException {
		return value;
	}
 
	@Override
	public float getProgress() throws IOException {
		return processed ? 1.0f : 0.0f;
	}
 
	@Override
	public void close() throws IOException {
		// do nothing
	}
}
```

```java
import java.io.IOException;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
 
public class SmallFilesToSequenceFileConverter extends Configured implements
		Tool {
	static class SequenceFileMapper extends
			Mapper<NullWritable, BytesWritable, Text, BytesWritable> {
		private Text filenameKey;
 
		@Override
		protected void setup(Context context) throws IOException,
				InterruptedException {
			InputSplit split = context.getInputSplit();
			Path path = ((FileSplit) split).getPath();
			filenameKey = new Text(path.toString());
		}
 
		@Override
		protected void map(NullWritable key, BytesWritable value,
				Context context) throws IOException, InterruptedException {
			context.write(filenameKey, value);
		}
	}
 
	@Override
	public int run(String[] args) throws Exception {
		Configuration conf = new Configuration();
		System.setProperty("HADOOP_USER_NAME", "hdfs");
		String[] otherArgs = new GenericOptionsParser(conf, args)
				.getRemainingArgs();
		if (otherArgs.length != 2) {
			System.err.println("Usage: combinefiles <in> <out>");
			System.exit(2);
		}
		Job job = new Job(conf, "combine small files to sequencefile");
		job.setInputFormatClass(WholeFileInputFormat.class);
		job.setOutputFormatClass(SequenceFileOutputFormat.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(BytesWritable.class);
		job.setMapperClass(SequenceFileMapper.class);
		return job.waitForCompletion(true) ? 0 : 1;
	}
 
	public static void main(String[] args) throws Exception {
		int exitCode = ToolRunner.run(new SmallFilesToSequenceFileConverter(),
				args);
		System.exit(exitCode);
		
	}
}
```

# 文件压缩

使用压缩的优点如下：

1. 节省数据占用的磁盘空间；
2. 加快数据在磁盘和网络中的传输速度，从而提高系统的处理速度。

### 压缩格式

| 压缩格式 | 工具  | 算法    | 文件扩展名 | 多文件 | 是否可切分       | HadoopCompressionCodec                     |
| -------- | ----- | ------- | ---------- | ------ | ---------------- | ------------------------------------------ |
| DEFLATE  | 无    | DEFLATE | .deflate   | 否     | 否               | org.apache.hadoop.io.compress.DefaultCodec |
| Gzip     | gzip  | DEFLATE | .gz        | 否     | 否               | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | bzip2 | bzip2   | .bz2       | 否     | 是               | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | lzop  | LZO     | .lzo       | 否     | 是（需要索引）   | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | N/A   | Snappy  | .Snappy    | 否     | 否               | org.apache.hadoop.io.compress.SnappyCodec  |
| LZ4      | N/A   | LZ4     | .lz4       | 否     | 否               | org.apache.hadoop.io.compress.Lz4Codec     |
| Zip      | zip   | DEFLATE | .zip       | 是     | 是，在文件范围内 | --                                         |

### 性能对比

| 压缩算法   | 原始文件大小 | 压缩文件大小 | 压缩速度   | 解压速度   |
| ---------- | ------------ | ------------ | ---------- | ---------- |
| `gzip`     | `8.3GB`      | `1.8GB`      | `17.5MB/s` | `58MB/s`   |
| `bzip2`    | `8.3GB`      | `1.1GB`      | `2.4MB/s`  | `9.5MB/s`  |
| `LZO-bset` | `8.3GB`      | `2GB`        | `4MB/s`    | `60.6MB/s` |
| `LZO`      | `8.3GB`      | `2.9GB`      | `49.3MB/s` | `74.6MB/s` |

## 参考文章

[关于 SequenceFile](http://xuexi.edu360.cn/714.html)

[Hadoop 日常实战 文件压缩](http://srcct.com/2014/12/28/2014/Hadoop%20%E6%97%A5%E5%B8%B8%E5%AE%9E%E6%88%98%20%E6%96%87%E4%BB%B6%E5%8E%8B%E7%BC%A9/)

[Hadoop 压缩实现分析](https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-compression-analysis/index.html)

[mapreduce合并小文件成sequencefile](https://blog.csdn.net/xiao_jun_0820/article/details/42747537)