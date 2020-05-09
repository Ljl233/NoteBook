# Java IO
在Java中如果要进行输出和输入操作，就需要使用到IO流，例如第一次写的语句System.out.println("hello,world")就是一个典型的输出流。IO体系中涉及到的类很多，但核心体系就是由File、 InputStream 、OutputStream、Reader、Writer和Serializable(接口)组成的。

## I/O 基本概念
按照流的方向可以分为输入流（InputStream）与输出流(OuputStream)：

- 输入流：只能读取数据，不能写入数据。
- 输出流：只能写入数据，不能读取数据。

这是对于使用来说的，程序是运行在内存中的，如果把内存中的数据写入硬盘中就是输出，反之则是输入。

![io流](https://img-blog.csdn.net/20170409144144824)

流:在 Java 中引入了 “流” 的概念，它表示任何有能力产生数据源或有能力接收数据源的对象。数据源可以想象成水源，海水、河水、湖水、一杯水等等。数据传输可以想象为水的运输，古代有用桶运水，用竹管运水的，现在有钢管运水，不同的运输方式对应不同的运输特性。

#### 按照处理的数据单位可以分为字节流和字符流：

- 字节流：操作的数据单元是8位的字节。以InputStream、OutputStream作为抽象基类。
- 字符流：操作的数据单元是字符。以Writer、Reader作为抽象基类。
- 字节流可以处理所有数据文件，如果处理的是纯文本数据，就使用字符流。


#### 按照流的作用可分为节点流和处理流：

- 节点流：程序直接与数据源连接，和实际的输入/输出节点连接。
- 处理流：对节点流进行包装，扩展原来的功能，通过处理流执行输入/输出操作。这涉及到装饰者模式。


#### IO流中的数据源有三类：

- 基于磁盘文件：FileInputStream、FileOutputSteam、FileReader、FileWriter
- 基于内存：ByteArrayInputStream ByteArrayOutputStream（ps:字节数组都是在内存中产生的）
- 基于网络：SocketInputStream、SocketOutputStream（ps:网络通信时传输数据）

#### 处理流的作用和分类：

处理流可以隐藏底层设备上节点流的差异，无需关心数据源的来源，程序只需要通过处理流执行输入/输出操作。处理流是以一个存在的节点流作为构造参数的。通常情况下，都推荐使用处理流来完成输入/输出操作。

- 缓冲流：提供一个缓冲区，能够提高输入/输出的执行效率，减少同节点的频繁操作。

BufferedInputStream/BufferedOutputStream、BufferedReader/BufferWriter

- 转换流：将字节流转成字符流。字节流使用范围广，但字符流使用更方便。例如一个字节流的数据源是文本内容的话，转成字符流来处理会更加方便。

InputStreamReader/OutputStreamWriter

- 打印输出流：把指定内容打印输出，根据构造参数中的节点流来决定输出到何处。

PrintStream ：打印输出字节数据，使用该类。

PrintWriter ： 打印输出文本数据，使用该类。

```java

    private static Object toObject(byte[] data) {

        ByteArrayInputStream bais = null;
        ObjectInputStream ois = null;
        try {
            bais = new ByteArrayInputStream(data);
            ois = new ObjectInputStream(bais);
            return ois.readObject();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                if (bais!=null){
                    bais.close();
                }
                if (ois!=null){
                    ois.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return null;
    }


    private static <T> byte[] toByteArray(T body) {

        ByteArrayOutputStream baos = null;
        ObjectOutputStream oos = null;

        try {
            baos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(baos);
            oos.writeObject(body);
            oos.flush();
            return baos.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (baos != null) {
                    baos.close();
                }
                if (oos != null) {
                    oos.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return new byte[0];
    }
```

```java
System.out.println("hello world");

public final static PrintStream out;
FileOutputStream fdOut = new FileOutputStream(FileDescriptor.out);
out = newPrintStream(fdOut, props.getProperty("sun.stdout.encoding"));

```
## 代码示例
（一）使用字节流读取本地文件内容：
```java
	//使用File对象来获取数据源，并读取其中内容,将其输出到标准控制台
	public static void getContent(File file) throws IOException {
		//创建文件输入流，数据源是file
		BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));
		byte[] buf = new byte[1024];//可以缓存50字节
		int len = 0; //读取后返回的长度
		while((len = bis.read(buf)) != -1) { //长度为-1则读取完毕，结束循环读取
			//打印缓存区的内容，从0开始
			System.out.println(new String(buf,0,len));		
		}
		bis.close();
	}
使用File类获取数据源，使用文件字节输入流来读取该数据源。如果数据源是纯文本数据的话，使用字符流会更方便，而且效率更高。
```
（二）使用字符处理流读取本地文件内容：
```java
	public static void getContent(String path) throws IOException {
		File f = new File(path);
		//判断f指向的数据源是否存在
		if(f.exists()) { 
			//判断是不是文件
			if(f.isFile()) {
				//不要向上转型成Reader,readLine()是子类独有方法
				BufferedReader br = new BufferedReader(new FileReader(path));
				String s = null;
				while((s = br.readLine()) != null) { //readLine()每次读取一行
					System.out.println(s);
				}
			}
		}		
	}
该方法比上一个增加了文件判断，提高了程序的健壮性。使用了BufferedReader处理流来处理纯文本数据，比字节流更加简洁方便。如果处理的是纯文本数据，强烈推荐字符处理流！！！
```
（三）使用字符流写入数据到指定文件：
```java
public static void main(String[] args) throws IOException {
		//以标准输入作为扫描来源
		Scanner sc = new Scanner(System.in);
		File f = new File("D:\\reviewIO\\WRITERTest.txt");
		BufferedWriter bw = new BufferedWriter(new FileWriter(f));
		if(!f.exists()) {
			f.createNewFile();
		}
		while(true) {
			String s = sc.nextLine();
			bw.write(s);
			bw.flush();
			if(s.equals("结束") || s.equals("")) {
				System.out.println("写入数据结束！");
				return;
			}
		}
	}
上面程序使用字符流简单地实现了写入文件的操作，字节流也差不多，可以自行更改。关于Scanner类，该类是一个扫描类，以System.in作为数据源。System.in是标准输入（键盘输入数据），这里只是先了解一下。
```
（四）使用转换流（InputStreamReader/OutputStreamWriter），对写入数据进行改进：
```java
public static void testConvert(File f) throws IOException {
		if(!f.exists()) {
			f.createNewFile();
		}
		//以System.in作为读取的数据源
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		BufferedWriter bw = new BufferedWriter(new FileWriter(f,true)); //允许添加内容，不会清除原来的数据源内容。
		String s = null;
		while(!(s = br.readLine()).equals("")) {
			bw.write(s);
			bw.newLine();//空一行
		}
		bw.flush();		
		bw.close();
		br.close();
	} 
因为System.in是一个InputStream对象，字符处理流无法直接使用，所以需要使用转换流将字节流转成字符流。然后使用字符输入处理流的readLine()每次读取一行，使用newLine()完成换行。
```
注意点：通常使用IO流写入文件时，写入的数据总会覆盖原来的数据，这是因为文件输出流默认不允许追加内容，所以需要为FileOuputStream、FileWriter的构造参数boolean append 传入true。
（五）使用字节流完成文件拷贝：
```java
//字节流实现文件拷贝
	public static String copyFile(String src, String dest) throws IOException, ClassNotFoundException {
		File srcFile = new File(src);//源文件数据源
		File desFile = new File(dest);//写入到目标数据源
		//数据源不存在
		if(!srcFile.exists() || !desFile.exists()) {
			throw new ClassNotFoundException("源文件或者拷贝目标文件地址不存在！");
		}
		//非文件类型
		if(!srcFile.isFile() || !desFile.isFile()) {
			return "源文件或者目标文件不是文件类型!";
		}
		InputStream is = null;
		OutputStream os = null;
		byte[] buf = new byte[1024];//缓存区
		int len = 0;//读取长度
		try {
			is = new BufferedInputStream(new FileInputStream(srcFile));//读取数据源
			os = new BufferedOutputStream(new FileOutputStream(desFile));//写入到数据源			
			while((len = is.read(buf)) != -1) { //读取长度不为-1，继续读取
				os.write(buf); //读取内容之后马上写入目标数据源
			}
			os.flush();//输出
			return "文件拷贝成功！查看拷贝文件路径：" + desFile.getPath();						
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}finally {
			if(is != null)
				is.close();
			if(os != null)
				os.close();
		}
		return "文件拷贝失败";
	}
如果是对文件进行复制，一般采用字节流来完成，因为可以作用于一切类型数据。但如果操作的是纯文本数据，应该使用字符流。
```
（六）使用打印流来完成写入数据操作：
```java
		//输出内容的文件数据源
		File f = new File("D:\\reviewIO\\PW.java");
		PrintWriter pw = new PrintWriter(f);
		//把指定内容打印至数据源中
		pw.println("AAAAAAAAA");
		pw.println("BBBBBBBBB");
		pw.println("CCCCCCCCC");
		pw.flush();
		System.out.println("使用PrintWriter写入数据完成");
		System.out.println("==========读取写入的数据==========");
		BufferedReader br = new BufferedReader(new FileReader(f));
		String s = null;
		StringBuilder sb = new StringBuilder();//一个可变字符串
		while((s = br.readLine()) != null) {
			sb.append(s); //把读取的字符串组合起来
		}
		System.out.println(sb);
		br.close();
		pw.close();
如上面代码所示，打印流可以完成写入数据操作。而且打印流比常规输出流功能更加强大，例如PrintWriter可以指定输出文本使用何种字符集。也可以在构造参数中指定自动刷新。如果不想覆盖原来的数据，使用该类的append()方法，就会在文件尾部添加内容。
```

（七）使用打印流来完成文本拷贝：
```java
//使用打印流PrintStream来完成文件拷贝
	public static void copyFile(File src, File dest) throws Exception {
		BufferedInputStream bis = new BufferedInputStream(new FileInputStream(src));
		BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(dest));
		PrintStream ps = new PrintStream(bos,true);
		byte[] buf = new byte[1024];
		int len = 0;
		while((len = bis.read(buf)) != -1) {
			ps.write(buf);
		}
		ps.close();
		bos.close();
		bis.close();
	}	
上面代码使用打印流来简单实现了文件拷贝操作，利用了打印流构造函数的自动刷新。使用打印流还有一个好处就是不需要检查异常。

```


参考：https://blog.csdn.net/fengwei4618/article/details/69808702