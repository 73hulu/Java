# BufferedReader
直接继承`Reader`，目标媒介是缓冲。从字符输出流中读取文本，缓冲字符，以提供字符、数组和行有效读，为其他字符输入流添加一些缓冲功能。结构定义如下：

![BufferedReader](http://ovn0i3kdg.bkt.clouddn.com/BufferedReader.png)

我们平常都是用如下的方式来使用这个类：
```java
BufferedReader in = new BufferedReader(new FileReader("foo.in"));
```
为什么`FileReader`已经是一个`Reader`了，为什么还要在它外面包裹一层？为了缓冲。为什么需要缓冲呢？原因很简单，效率问题！缓冲中的数据实际上是保存在内存中，而原始数据可能是保存在硬盘或NandFlash中；而我们知道，从内存中读取数据的速度比从硬盘读取数据的速度至少快10倍以上。

那干嘛不干脆一次性将全部数据都读取到缓冲中呢？第一，读取全部的数据所需要的时间可能会很长。第二，内存价格很贵，容量不像硬盘那么大。


## 构造方法
定义了两个构造方法。
### public BufferedReader(Reader in, int sz){...}
定义了一个字符输入流，指定缓冲区大小。定义如下：
```java
public BufferedReader(Reader in, int sz) {
    super(in);
    if (sz <= 0)
        throw new IllegalArgumentException("Buffer size <= 0");
    this.in = in;
    cb = new char[sz];
    nextChar = nChars = 0;
}
```
首先执行了父类`Reader`的构造方法，即把字符输出缓冲区作为锁。定义了输入流`in`，`cb`是字符数组，是字符缓冲区，`nextChar`是下一个要读取的字符在cb缓冲区中的位置  ，`nChars`是cb缓冲区中字符的总的个数。

### public BufferedReader(Reader in){..}
只接受一个输入流作为参数，缓冲区大小默认为8192，即8k，定义如下：
```java
public BufferedReader(Reader in) {
    this(in, defaultCharBufferSize);
}
```

### private void ensureOpen() throws IOException{...}
该方法用来确保流未关闭。定义如下：
```java
private void ensureOpen() throws IOException {
    if (in == null)
        throw new IOException("Stream closed");
}
```


### private void fill() throws IOException{...}
该函数用来填充缓冲区，这个函数会在两种情况下被调用：
1.  缓冲区没有数据时，通过`fill()`可以向缓冲区填充数据。   
2. 缓冲区数据被读完，需更新时，通过fill()可以更新缓冲区的数据。

定义如下：
```java
private void fill() throws IOException {
    int dst; // dst表示“cb中填充数据的起始位置”。
    if (markedChar <= UNMARKED) { // 没有标记的情况，则设dst=0
        /* No mark */
        dst = 0;
    } else {
        /* Marked */
        int delta = nextChar - markedChar; // delta表示“当前标记的长度”，它等于“下一个被读取字符的位置”减去“标记的位置”的差值；
        if (delta >= readAheadLimit) {
            /* Gone past read-ahead limit: Invalidate mark */

            // 若“当前标记的长度”超过了“标记上限(readAheadLimit)”，    
            // 则丢弃标记！    
            markedChar = INVALIDATED;
            readAheadLimit = 0;
            dst = 0;
        } else {
            if (readAheadLimit <= cb.length) {
                /* Shuffle in the current buffer */

                // 若“当前标记的长度”没有超过了“标记上限(readAheadLimit)”，    
                // 并且“标记上限(readAheadLimit)”小于/等于“缓冲的长度”；    
                // 则先将“下一个要被读取的位置，距离我们标记的置符的距离”间的字符保存到cb中。
                System.arraycopy(cb, markedChar, cb, 0, delta);
                markedChar = 0;
                dst = delta;
            } else {
                /* Reallocate buffer to accommodate read-ahead limit */
                // 若“当前标记的长度”没有超过了“标记上限(readAheadLimit)”，    
                // 并且“标记上限(readAheadLimit)”大于“缓冲的长度”；     
                // 则重新设置缓冲区大小，并将“下一个要被读取的位置，距离我们标记的置符的距离”间的字符保存到cb中。
                char ncb[] = new char[readAheadLimit];
                System.arraycopy(cb, markedChar, ncb, 0, delta);
                cb = ncb;
                markedChar = 0;
                dst = delta;
            }
            // 更新nextChar和nChars    
            nextChar = nChars = delta;
        }
    }

    int n;
    do {
      // 从“in”中读取数据，并存储到字符数组cb中；    
     // 从cb的dst位置开始存储，读取的字符个数是cb.length - dst    
     // n是实际读取的字符个数；若n==0(即一个也没读到)，则继续读取！   
        n = in.read(cb, dst, cb.length - dst);
    } while (n == 0);

    // 如果从“in”中读到了数据，则设置nChars(cb中字符的数目)=dst+n，    
    // 并且nextChar(下一个被读取的字符的位置)=dst。    
    if (n > 0) {
        nChars = dst + n;
        nextChar = dst;
    }
}
```
其中一些变量定义如下：
```java
private int nChars, nextChar;
private static final int INVALIDATED = -2;
private static final int UNMARKED = -1;
private int markedChar = UNMARKED;
private int readAheadLimit = 0; /* Valid only when markedChar > 0 */
```
其中`INVALIDATED`表示“标记无效”，它与`UNMARKED`的区别是：(01)`UNMARKED`是压根就没有设置过标记;(02)而`INVALIDATED`是设置了标记，但是被标记位置太长，导致标记无效！其中`UNMARKED`表示没有设置“标记” 。`markedChar`表示“标记” 。`readAheadLimit`表示“标记”能标记位置的最大长度。

> `fill`方法的详细解释可以参考 http://blog.csdn.net/asivy/article/details/18704449

### read方法
read方法用来实现从缓存中读取字符，有2个重载方法，其中一个是重写了父类的read的方法，一个实现了父类的抽象方法。
### public int read() throws IOException{..}
该方法重写了父类方法，用来从缓存中读取单个字符，最后以int的形式返回。定义如下：
```java
public int read() throws IOException {
    synchronized (lock) {
        ensureOpen();
        for (;;) {
            if (nextChar >= nChars) {
                fill();
                if (nextChar >= nChars)
                    return -1;
            }
            if (skipLF) {
                skipLF = false;
                if (cb[nextChar] == '\n') {
                    nextChar++;
                    continue;
                }
            }
            return cb[nextChar++];
        }
    }
}
```
该方法线程安全，首先保证流正处理打开状态。如果要读取的位置已经超过了最大字符数，说明缓冲区的数据已经被读完，那么需要调用`fill`方法读入新的数据；如果读完还是大于，那么说明之前就已经到达了input的结尾，所以返回-1。`skipLF`即skip Line Feed，是“是否忽略换行符”标记。若要“忽略换行符”， 则对下一个字符是否是换行符进行处理。最后返回下一个字符。

### public int read(char cbuf[], int off, int len) throws IOException{...}
实现了父抽象类`Reader`的抽象方法，这个方法将缓冲区的数据写入到`cbuf`数组中，其中`off`是数组的偏移量，len是写入的长度。
```java
public int read(char cbuf[], int off, int len) throws IOException {
    synchronized (lock) {
        ensureOpen();
        if ((off < 0) || (off > cbuf.length) || (len < 0) ||
            ((off + len) > cbuf.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        }

        int n = read1(cbuf, off, len);
        if (n <= 0) return n;
        while ((n < len) && in.ready()) {
            int n1 = read1(cbuf, off + n, len - n);
            if (n1 <= 0) break;
            n += n1;
        }
        return n;
    }
}
```
这几个实际上是对`read1`方法的包装，添加了"同步处理"和“阻塞式读取”的功能，定义如下：
```java
private int read1(char[] cbuf, int off, int len) throws IOException {
    if (nextChar >= nChars) {
        /* If the requested length is at least as large as the buffer, and
           if there is no mark/reset activity, and if line feeds are not
           being skipped, do not bother to copy the characters into the
           local buffer.  In this way buffered streams will cascade
           harmlessly. */
        if (len >= cb.length && markedChar <= UNMARKED && !skipLF) {
            return in.read(cbuf, off, len);
        }
        fill();
    }
    if (nextChar >= nChars) return -1;
    if (skipLF) {
        skipLF = false;
        if (cb[nextChar] == '\n') {
            nextChar++;
            if (nextChar >= nChars)
                fill();
            if (nextChar >= nChars)
                return -1;
        }
    }
    int n = Math.min(len, nChars - nextChar);
    System.arraycopy(cb, nextChar, cbuf, off, n);
    nextChar += n;
    return n;
}
```
首先，比较nextChar和nChars的值，如果前者大于等于或者，说明缓冲区的数据已经被读完，则更新缓冲区数据。如果已经读完仍然大于，说明input已经被读取完了，此时返回-1。如果要“忽略换行符”，则进行相应的处理。之后进行字符串拷贝。


## readLine方法
readLine提供了读取一行的方法。重载了两个方法。
### public String readLine() throws IOException{...}
无参构造方法，定义如下：
```java
public String readLine() throws IOException {
    return readLine(false);
}
```
调用的是有参构造方法，默认参数false。
### String readLine(boolean ignoreLF) throws IOException{...}
其中的参数`ignoreLF`表示是否忽略换行符。
```java
String readLine(boolean ignoreLF) throws IOException {
   StringBuffer s = null;
   int startChar;

   synchronized (lock) {
     // 确保底层的inputStream未被关闭  
       ensureOpen();
     // ignoreLF在BufferedReader中始终为false  
     // 而skipLF则会在读到"\r"字符之后被设置为true，通过这样来忽略掉之后的"\n"  
       boolean omitLF = ignoreLF || skipLF;

       // 接下来是两个循环，第一层bufferLoop主要是用来往底层的数组里填充字符，这个底层数组就相当于一个缓冲区  
       // 而这个缓冲区的大小可以通过BufferedReader(Reader in, int  
       // sz)这个构造方法的第二个参数来指定，默认为8192bytes  
       // 而charLoop则是用来遍历底层数组，每次读完底层数组里的数据，就把这些数据写到一个StringBuffer里，  
       // 直到读到"\r"或"\n"的时候，写入最后一批数据之后就返回结果并退出整个循环
   bufferLoop:
       for (;;) {
         // nextChar表示下一个要读的字符的位置、nChars=从inputstream中读入的字符数+nextChar  
           if (nextChar >= nChars)
               fill();
        // 读到了inputstream的末尾   
           if (nextChar >= nChars) { /* EOF */
               if (s != null && s.length() > 0)
                   return s.toString();
               else
                   return null;
           }
           boolean eol = false;
           char c = 0;
           int i;

           /* Skip a leftover '\n', if necessary */
           if (omitLF && (cb[nextChar] == '\n'))
               nextChar++;
           skipLF = false;
           omitLF = false;

       // 退出这一层循环有两种情况：1、读到"\r"或者"\n" 2、底层数组cb被读完了  
       charLoop:
           for (i = nextChar; i < nChars; i++) {
               c = cb[i];
               if ((c == '\n') || (c == '\r')) {
                   eol = true;
                   break charLoop;
               }
           }
          // 读取底层数组的开始位置
           startChar = nextChar;
          //当前读取到的位置
           nextChar = i;
          //是否读完了一行（end of line）
           if (eol) {
               String str;
               if (s == null) {
                 // 这里直接用s不就行了吗。为什么要搞个str变量  
                   str = new String(cb, startChar, i - startChar);
               } else {
                   s.append(cb, startChar, i - startChar);
                   str = s.toString();
               }
               // 下一次要读取的位置
               nextChar++;
               if (c == '\r') {
                   skipLF = true;
               }
               return str;
           }


            // 写入读到的数据
           if (s == null)
               s = new StringBuffer(defaultExpectedLineLength);
           s.append(cb, startChar, i - startChar);
       }
   }
}
```
额，有点复杂，还是先不看了。

## public long skip(long n) throws IOException{...}
重写覆盖了父类的`skip`方法。定义如下：
```java
public long skip(long n) throws IOException {
    if (n < 0L) {
        throw new IllegalArgumentException("skip value is negative");
    }
    synchronized (lock) {
        ensureOpen();
        long r = n;
        while (r > 0) {
            if (nextChar >= nChars)
                fill();
            if (nextChar >= nChars) /* EOF */
                break;
            if (skipLF) {
                skipLF = false;
                if (cb[nextChar] == '\n') {
                    nextChar++;
                }
            }
            long d = nChars - nextChar;
            if (r <= d) {
                nextChar += r;
                r = 0;
                break;
            }
            else {
                r -= d;
                nextChar = nChars;
            }
        }
        return n - r;
    }
}
```
过程比较好理解，首先检查了缓冲区是不是已经被读完，是的话重新填充。之后对换行符做处理。之后判断剩余的字符和所要求的字符长度做比较。

## public boolean ready() throws IOException{...}
用来判断”下一个字符”是否可读。定义如下：
```java
public boolean ready() throws IOException {
    synchronized (lock) {
        ensureOpen();

        /*
         * If newline needs to be skipped and the next char to be read
         * is a newline character, then just skip it right away.
         */
        if (skipLF) {
            /* Note that in.ready() will return true if and only if the next
             * read on the stream will not block.
             */
            if (nextChar >= nChars && in.ready()) {
                fill();
            }
            if (nextChar < nChars) {
                if (cb[nextChar] == '\n')
                    nextChar++;
                skipLF = false;
            }
        }
        return (nextChar < nChars) || in.ready();
    }
}
```

## public boolean markSupported(){...}
始终返回true，因为BufferedReader支持mark(), reset()。

## public void mark(int readAheadLimit) throws IOException{...}
该方法用来标记当前`BufferedReader`的下一个要读取位置。定义如下：
```java
public void mark(int readAheadLimit) throws IOException {
   if (readAheadLimit < 0) {
       throw new IllegalArgumentException("Read-ahead limit < 0");
   }
   synchronized (lock) {
       ensureOpen();
       this.readAheadLimit = readAheadLimit; // 设置readAheadLimit
       markedChar = nextChar; // 保存下一个要读取的位置  
       markedSkipLF = skipLF; // 保存“是否忽略换行符”标记    
   }
}
```

## public void reset() throws IOException{...}
重置BufferedReader的下一个要读取位置，将其还原到`mark()`中所保存的位置。
```java
public void reset() throws IOException {
   synchronized (lock) {
       ensureOpen();
       if (markedChar < 0)
           throw new IOException((markedChar == INVALIDATED)
                                 ? "Mark invalid"
                                 : "Stream not marked");
       nextChar = markedChar;
       skipLF = markedSkipLF;
   }
}
```

## public void close() throws IOException{...}
关闭流，需要关闭其封装的流，并将其中一些引用置为null。定义如下：
```java
public void close() throws IOException {
    synchronized (lock) {
        if (in == null)
            return;
        try {
            in.close();
        } finally {
            in = null;
            cb = null;
        }
    }
}
```


参考
* [BufferedReader 源码分析](http://blog.csdn.net/asivy/article/details/18704449)
* [JAVA IO 之BufferedReader源码分析 ](http://blog.chinaunix.net/uid-1911213-id-3156755.html)
