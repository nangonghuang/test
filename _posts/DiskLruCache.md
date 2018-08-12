---
title: DiskLruCache
date: 2017-12-06 09:05:11
tags: 
categories: Android_基础
---

DiskLruCache是用来在磁盘上缓存文件的类，<!--more-->在网上看到说的都是谷歌写的，不过我找到的DiskLruCache目前都是jakewharton的库，但里面的注释都是Android Open Source Project，不是很清楚这之间的原因。

一个是独立的库，通过

> ```
> api 'com.jakewharton:disklrucache:2.0.2'
> ```

使用，文件流的操作采用Java IO，另外一个是okhttp里面的，文件流操作采用ok.io，两者都是类似的。

一次写入操作的代码是

```Java
 /**
  * 在这段代码里面，需要关注的，
  * 一是Editor是什么；在DiskLruCache里面，Editor是用来对Entry进行操作的类，而Entry又是对缓存文件
  * 的封装，包括cleanFile和dirtyFile，我们每次拿imageUrl生成一个key,然后根据这个key创建一个Entry条 	
  * 目，因此Editor就可以控制缓存文件的读写。文件的写入是先写到DirtyFile，然后再通过Editor.commit提
  * 交，提交成功则把DirtyFile重命名为cleanFile,失败则丢弃，同时也会写到相应的日志文件里面去
  */

new Thread(new Runnable() {  
    @Override  
    public void run() {  
        try {  
            String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";  
            String key = hashKeyForDisk(imageUrl);  
            DiskLruCache.Editor editor = mDiskLruCache.edit(key);  
            if (editor != null) {  
                OutputStream outputStream = editor.newOutputStream(0);  
                if (downloadUrlToStream(imageUrl, outputStream)) {  
                    editor.commit();  
                } else {  
                    editor.abort();  
                }  
            }  
            mDiskLruCache.flush();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }  
}).start();
```

读取操作：

```Java
 /**
         * 这里涉及到Snapshot类，Snapshot是在mDiskLruCache.get(key)的时候才生成的，主要作用是根据
         * key取到的Entry，对这个Entry做一个封装，把Entry的一些参数暴露出来给Snapshot,比如拿到文件
         * 输入流给你，至于为什么要这么做，猜测一是为了隐藏Entry类，只把需要的参数暴露出来，二是承担
         * 读取的作用，让Editor专心去写。很多文件写入的封装都是用一个Editor去写并且不负责读，写的好处
         * 在于隔离了数据生效的及时性，可以通过Editor来控制同步或者异步写入数据的过程，同时也能在中间
         * 做一些处理，至于为什么不让Editor类负责读数据...只能猜测就是这么设计这个类的功能的。。。
         */

try {  
    String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";  
    String key = hashKeyForDisk(imageUrl);  
    DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);  
    if (snapShot != null) {  
        InputStream is = snapShot.getInputStream(0);  
        Bitmap bitmap = BitmapFactory.decodeStream(is);  
        mImage.setImageBitmap(bitmap);  
    }  
} catch (IOException e) {  
    e.printStackTrace();  
}
```

和读写缓存相关的就是这些类了，另外在DiskLruCache中很大的一部分内容是缓存日志的读写操作，下面的日志文件的注释

> ```
> /**
>  * 这个缓存用了一个名字较journal的日志文件，一个典型的日志文件是这样的格式：
>  *     libcore.io.DiskLruCache
>  *     1
>  *     100
>  *     2
>  *
>  *     CLEAN 3400330d1dfc7f3f7f4b8d4d803dfcf6 832 21054
>  *     DIRTY 335c4c6028171cfddfbaae1a9c313c52
>  *     CLEAN 335c4c6028171cfddfbaae1a9c313c52 3934 2342
>  *     REMOVE 335c4c6028171cfddfbaae1a9c313c52
>  *     DIRTY 1ab96a171faeeee38496d8b330771a7a
>  *     CLEAN 1ab96a171faeeee38496d8b330771a7a 1600 234
>  *     READ 335c4c6028171cfddfbaae1a9c313c52
>  *     READ 3400330d1dfc7f3f7f4b8d4d803dfcf6
>  *
>  * 头五行构成了日志文件的header。它们是固定的字符串"libcore.io.DiskLruCache"，磁盘缓存的
>  * 版本号，应用程序的版本号，valueCount（一个key对应几套读写缓存文件），空白行
>  *
>  * 下面的连续行每一行都表示对一个canche Entry的状态的记录，包括三个数据，状态，key，和可选的
>  * 状态相关的数值
>  *   DIRTY行  表示一个Entry正在被创建或者更新。每一个成功的DIRTY动作都应该紧跟着一个CLEAN行
>  *          或者REMOVE行，如果缺失CLEAN或者REMOVE表示这一行的key对应的缓存文件需要被删除掉
>  *   CLEAN行  表示一个Entry已经成功的记录下来并且可读了，后面跟着的数据表示每一个文件的长度
>  *   READ行   表示一次成功的读取记录
>  *   REMOVE行  表示已经删除掉的Entry的记录
>  *
>  * The journal file is appended to as cache operations occur. The journal may
>  * occasionally be compacted by dropping redundant lines. A temporary file named
>  * "journal.tmp" will be used during compaction; that file should be deleted if
>  * it exists when the cache is opened.
>  */
> ```

读取这个日志文件用到了一个工具类函数

```Java
 /**
   * Reads the next line. A line ends with {@code "\n"} or {@code "\r\n"},
   * this end of line marker is not included in the result.
   *
   * @return the next line from the input.
   * @throws IOException for underlying {@code InputStream} errors.
   * @throws EOFException for the end of source stream.
   */
//读取一行数据
// 从输入流里面读8192的数据到buf[]里面去，检查有没有换行符，有就输出，没有再new ByteArrayOutputStream
// 类，循环的读数据到buf再写到output流里面去，一直读到文件末尾都没有换行符则抛出Exception
  public String readLine() throws IOException {
    synchronized (in) {
      if (buf == null) {
        throw new IOException("LineReader is closed");
      }

      // Read more data if we are at the end of the buffered data.
      // Though it's an error to read after an exception, we will let {@code fillBuf()}
      // throw again if that happens; thus we need to handle end == -1 as well as end == pos.
      if (pos >= end) {
        fillBuf();
      }
      // Try to find LF in the buffered data and return the line if successful.
      for (int i = pos; i != end; ++i) {
        if (buf[i] == LF) {
          int lineEnd = (i != pos && buf[i - 1] == CR) ? i - 1 : i;
          String res = new String(buf, pos, lineEnd - pos, charset.name());
          pos = i + 1;
          return res;
        }
      }

      // Let's anticipate up to 80 characters on top of those already read.
      ByteArrayOutputStream out = new ByteArrayOutputStream(end - pos + 80) {
        @Override
        public String toString() {
          int length = (count > 0 && buf[count - 1] == CR) ? count - 1 : count;
          try {
            return new String(buf, 0, length, charset.name());
          } catch (UnsupportedEncodingException e) {
            throw new AssertionError(e); // Since we control the charset this will never happen.
          }
        }
      };

      while (true) {
        out.write(buf, pos, end - pos);
        // Mark unterminated line in case fillBuf throws EOFException or IOException.
        end = -1;
        fillBuf();
        // Try to find LF in the buffered data and return the line if successful.
        for (int i = pos; i != end; ++i) {
          if (buf[i] == LF) {
            if (i != pos) {
              out.write(buf, pos, i - pos);
            }
            pos = i + 1;
            return out.toString();
          }
        }
      }
    }
  }

  /**
   * Reads new input data into the buffer. Call only with pos == end or end == -1,
   * depending on the desired outcome if the function throws.
   */
  private void fillBuf() throws IOException {
    int result = in.read(buf, 0, buf.length);
    if (result == -1) {
      throw new EOFException();
    }
    pos = 0;
    end = result;
  }
```

其他更详细的细节需要去看源码了。



感谢 [郭霖的博客](http://blog.csdn.net/guolin_blog/article/details/28863651)

