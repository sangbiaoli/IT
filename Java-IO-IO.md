## Java IO

1. File

    * 构造方法

        方法|说明
        --|--
        File(String pathname)|通过路径创建文件实例
        File(String parent, String child)|通过父路径和子路径创建文件实例
        File(File parent,String child)|通过一个父文件路径名和子路径创建文件实例

    * 存储的抽象路径

        ```java
        File file = new File("..\\src\\test1.txt");
        ```

        以上面的代码为例，解析下面的方法
        方法|说明
        --|--
        String path()|得到的是构造file的时候的路径，输出为  *..\src\test1.txt*
        String getAbsolutePath()|得到的是全路径</br>如果构造的时候就是全路径那直接返回全路径</br>如果构造的时候试相对路径，返回当前目录的路径+构造file时候的路径</br>输出为 *D:\Workspaces\git-base\..\src\test1.txt*
        String getCanonicalPath()|CanonicalPath不但是全路径，而且把..或者.这样的符号解析出来</br>输出为  *D:\Workspaces\src\test1.txt*
        boolean isAbsolute()|判断抽象路径是否是绝对的

    * 列举系统根目录

        ```java
        import java.io.File;

        public class DumpRoots {
            public static void main(String[] args) {
                File[] roots = File.listRoots();
                for (File root : roots)
                    System.out.println(root);
            }
        }
        ```

    * 系统磁盘空间

        方法|说明
        --|--
        long getFreeSpace() |返回此抽象路径名所命名的分区中未分配的字节数，如果抽象路径不命名分区则返回0
        long getTotalSpace() |返回此抽象路径名所命名的分区中总的字节数，如果抽象路径不命名分区则返回0
        long getUsableSpace() |返回此抽象路径名所命名的分区中可用于当前JVM的字节数

        ```java
        import java.io.File;

        public class PartitionSpace {
            public static void main(String[] args) {
                File[] roots = File.listRoots();
                for (File root : roots) {
                    System.out.println("Partition: " + root);
                    System.out.println("Free space on this partition = " + root.getFreeSpace());
                    System.out.println("Usable space on this partition = " + root.getUsableSpace());
                    System.out.println("Total space on this partition = " + root.getTotalSpace());
                    System.out.println("***");
                }
            }
        }
        ```

    * 列举目录

        **对于一个File实例，如果这个实例是个路径，则可以获取路径下的文件和目录**

        方法|说明
        --|--
        String[] list()|返回目录下所有文件路径名数组
        String[] list(FilenameFilter filter)|返回目录下满足文件命名过滤器的所有文件路径名数组
        File[] listFiles()|返回目录下所有文件数组
        File[] list(FileFilter filter)|返回目录下满足文件过滤器的所有文件数组
        File[] list(FilenameFilter filter)|返回目录下满足文件命名过滤器的所有文件数组

        ```java
        import java.io.File;
        import java.io.FilenameFilter;

        public class Dir {
            public static void main(String[] args) {
                File file = new File("C:");
                FilenameFilter fnf = new FilenameFilter() {
                    public boolean accept(File dir, String name) {
                        return name.endsWith(".log");
                    }
                };
                String[] names = file.list(fnf);
                for (String name : names)
                    System.out.println(name);
            }
        }
        ```

    * 创建/修改 文件与目录

        **得到一个File实例，通过这个实例既可以创建文件也可以创建目录。**

        方法|说明
        --|--
        boolean createNewFile()|通过抽象的路径来生成文件
        static File createTempFile(String prefix, String suffix)|通过抽象的路径来生成默认的临时文件
        boolean delete()|通过抽象的路径删除文件或目录
        boolean mkdir()|通过抽象的路径来生成目录，成功返回true，否则返回false，这个方法只会一步到位，比如系统有路径 *C:\\*，要创建 **C:\\a.txt** 能成功，**C:\\abc\\a.txt**则不能成功
        boolean mkdirs()|只要抽象的路径不存在，则会一级级的创建，成功返回true，否则返回false

    * 文件权限

        文件自带一些自身的权限：可执行，可读，可写

        方法|说明
        --|--
        boolean setExecutable(boolean executable, boolean ownerOnly) |设置是否可执行，该权限是对于文件所有者还是所有人
        boolean setReadable(boolean executable, boolean ownerOnly) |设置是否可读，该权限是对于文件所有者还是所有人
        boolean setWritable(boolean executable, boolean ownerOnly) |设置是否可写，该权限是对于文件所有者还是所有人

2. RandomAccessFile
3. Streams
4. Writers and Readers

