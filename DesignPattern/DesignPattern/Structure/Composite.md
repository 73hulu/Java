# Composite
组合设计模式。将对象组合成**树形结构**以表示“部分-整体”的层次结构。

组合模式使得用户对单个对象和组合对象的使用具有唯一性。

![组合设计模式](https://ws1.sinaimg.cn/large/006tNbRwly1fx26aspk83j30pq0j4dgp.jpg)

什么意思呢？比如，我们创建两个对象，他们的性质实际上是不同的，他们之间具有“整体-部分”的关系，但是我们可以用同样的方法来操纵他们。

最典型的例子就是文件系统，"文件夹"和“文件”是文件系统的组成，如果文件系统是一棵树，那么“文件夹”是一个非叶子节点，“文件”是一个叶子节点，它们之间是树形关系。两者也有一些共性，比如都可以进行复制、剪切、粘贴、重命名等，但是又有一些细微的差别，比如文件夹的删除操作，我们需要删除它以下的所有子文件夹和文件，而对于一个文件，只用删除文件本身就行。

那么定义当中的一致性就体现在，客户端不用管当前操作的是文件还是文件夹，只需要使用同样语义的方法进行操作就行，不用特别去处理。


所以，根据组合模式，我们首先给出一个接口，这个接口就是所谓的“component”，它定义了文件和文件夹的公共行为。
```Java
package com.composite;


//文件系统中的节点接口
public interface IFile {
    void delete();
    String getName();

    void createNewFile(String name);
    void deleteFile(String name);
    IFile getIFile(int index);
}
```

 类图中的operation方法是一个宏观定义，它代表的意思是叶子节点和非叶子节点的公共行为，并不是说只有一个operation方法，本次LZ给出两个共有行为作为代表，即删除操作和获取文件名称的操作。

 下面来看看非叶子节点的操作，即文件夹的操作：
 ```Java
 package com.composite;

import java.util.ArrayList;
import java.util.List;

//文件夹
public class Folder implements IFile{

    private String name;
    private IFile folder;
    private List<IFile> files;

    public Folder(String name) {
        this(name, null);
    }

    public Folder(String name,IFile folder) {
        super();
        this.name = name;
        this.folder = folder;
        files = new ArrayList<IFile>();
    }

    public String getName() {
        return name;
    }

    //与File的删除方法不同，先删除下面的文件以及文件夹后再删除自己
    public void delete() {
        List<IFile> copy = new ArrayList<IFile>(files);
        System.out.println("------------删除子文件-------------");
        for (IFile file : copy) {
            file.delete();
        }
        System.out.println("----------删除子文件结束-------------");
        if (folder != null) {
            folder.deleteFile(name);
        }
        System.out.println("---删除[" + name + "]---");
    }

    public void createNewFile(String name) {
        if (name.contains(".")) {
            files.add(new File(name,this));
        }else {
            files.add(new Folder(name,this));
        }
    }

    public void deleteFile(String name) {
        for (IFile file : files) {
            if (file.getName().equals(name)) {
                files.remove(file);
                break;
            }
        }
    }

    public IFile getIFile(int index) {
        return files.get(index);
    }
}
 ```

我们看到这里面最主要的地方在于它有一个`List<IFile>`属性，这个属性是树结构的关键点，当我们删除一个文件夹时，即`delet`e方法，我们会首先删除该文件夹下面的所有文件以及文件夹，这与我们平时使用的windows操作系统的文件操作是一致的。

叶子节点即文件的实现如下：
```Java
package com.composite;

//文件
public class File implements IFile{

    private String name;
    private IFile folder;

    public File(String name,IFile folder) {
        super();
        this.name = name;
        this.folder = folder;
    }

    public String getName() {
        return name;
    }

    public void delete() {
        folder.deleteFile(name);
        System.out.println("---删除[" + name + "]---");
    }

    //文件不支持创建新文件
    public void createNewFile(String name) {
        throw new UnsupportedOperationException();
    }
    //文件不支持删除文件
    public void deleteFile(String name) {
        throw new UnsupportedOperationException();
    }
    //文件不支持获取下面的文件列表
    public IFile getIFile(int index) {
        throw new UnsupportedOperationException();
    }

}
```
文件类中的delete方法与文件夹中的不同，一个文件的删除操作，只需要删除它自己即可。我们还会注意到，下面的三个方法，LZ全部抛出了不支持的操作的异常，这也是与我们传统意义上的文件操作是一致的，一个文件当然不能在该文件下进行创新新文件、删除文件以及获取某个文件的操作。当然，你也可以直接将三个方法放空，或者返回null值。但是这样不推荐，因为不利于调试。


现在看一个文件系统的操作
```Java
package com.composite;

public class Main {

    public static void main(String[] args) {
        IFile root = new Folder("我的电脑");
        root.createNewFile("C盘");
        root.createNewFile("D盘");
        root.createNewFile("E盘");
        IFile D = root.getIFile(1);
        D.createNewFile("project");
        D.createNewFile("电影");
        IFile project = D.getIFile(0);
        project.createNewFile("test1.java");
        project.createNewFile("test2.java");
        project.createNewFile("test3.java");
        IFile movie = D.getIFile(1);
        movie.createNewFile("致青春.avi");
        movie.createNewFile("速度与激情6.avi");

        /* 以上为当前文件系统的情况，下面我们尝试删除文件和文件夹 */
        display(null, root);
        System.out.println();

        project.delete();
        movie.getIFile(1).delete();

        System.out.println();
        display(null, root);
    }

    //打印文件系统
    public static void display(String prefix,IFile iFile){
        if (prefix == null) {
            prefix = "";
        }
        System.out.println(prefix + iFile.getName());
        if(iFile instanceof Folder){
            for (int i = 0; ; i++) {
                try {
                    if (iFile.getIFile(i) != null) {
                        display(prefix + "--", iFile.getIFile(i));
                    }
                } catch (Exception e) {
                    break;
                }
            }
        }
    }
}
```
在我们操作的过程中，我们并不知道当前的操作是文件还是文件夹，我们都以同样的方法去操作，不同的节点对应了不同的操作实现。



上面是一个标准的组合模式的使用方法，但是其中有不妥之处，比如对于文件来说，有三个方法是不支持的，之所以出现这种情况是，是因为IFile中提供的是宽接口，这样做的目的是为了保持对客户端的透明性，然而相应却带来不安全性。

一个解决方法是，为了安全性牺牲透明性，即把IFile中叶子节点不支持的三个行为全部删掉，由此可见，组合模式中，安全性和透明性是互相矛盾的，这是由于叶子节点和非叶子节点行为的不一致及需要提供一个统一行为的接口造成的，是不可调和的矛盾。

一个解决办法是，在使用这些有安全性问题的方法之前先判断其类型，比如：
```Java
IFile movie = D.getIFile(1);
 if (movie instanceof Folder) {
     Folder folder = (Folder) movie;
     //下面使用folder进行文件夹独有的操作
 }
```
但是这样好像又不符合组合模式"用户对单个对象和组合对象的使用具有唯一性"的特点。

参考
* [设计模式之组合模式](https://www.cnblogs.com/haoerlv/p/7773224.html)
