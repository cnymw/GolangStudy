# linux 文件操作

## linux 文件系统

目录|用途
---|:--:
/|虚拟目录的根目录，通常不会在这里存储文件
/bin|二进制目录，存放许多用户级的GNU工具
/boot|启动目录，存放启动文件
/dev|设备目录，Linux 在这里创建设备节点
/etc|系统配置文件目录
/home|主目录，Linux 在这里创建用户目录
/lib|库目录，存放系统和应用程序的库文件
/media|媒体目录，可移动媒体设备的常用挂载点
/mnt|挂载目录，另一个可移动媒体设备的常用挂载点
/opt|可选目录，常用于存放第三方软件包和数据文件
/proc|进程目录，存放现有硬件及当前进程的相关信息
/root|root 用户的主目录
/sbin|系统二进制目录，存放许多 GNU 管理员级工具
/run|运行目录，存放系统运作时数据
/srv|服务目录，存放本地服务的相关文件
/sys|系统目录，存放系统硬件信息的相关文件
/tmp|临时目录，可以在该目录中创建和删除临时工作文件
/usr|用户二进制目录，大量用户级的 GNU 工具和数据文件都存储在这里
/var|可变目录，用以存放经常变化的文件，比如日志文件

## 浏览文件系统

### 显示当前目录

pwd 命令可以显示出 shell 会话的当前目录，这个目录被称为当前工作目录。

```bash
BenjaminYoungdeMacBook-Pro:~ benjamin$ pwd
/Users/benjamin
```
### 进入目录

1. 进入绝对文件路径：

```bash
BenjaminYoungdeMacBook-Pro:~ benjamin$ cd /Users/benjamin/
```

2. 进入相对文件路径

```bash
BenjaminYoungdeMacBook-Pro:~ benjamin$ cd Documents/
BenjaminYoungdeMacBook-Pro:Documents benjamin$ 
```

特殊字符可用于相对文件路径中:

- 单点符（.），表示当前目录
- 双点符（..），表示当前目录的父目录

```bash
BenjaminYoungdeMacBook-Pro:Documents benjamin$ cd ../Downloads/
BenjaminYoungdeMacBook-Pro:Downloads benjamin$ 
```

### 显示文件和目录

ls 命令最基本的形式会显示当前目录下的文件和目录:

```bash
BenjamiungdeMBP:Users benjamin$ ls
Guest		Shared		benjamin
```

以下列出常用的参数以及用途:

参数|用途
---|:--:
-a|显示所有档案及目录（ls 内定将档案名或目录名称为“.”的视为隐藏，不会列出）；
-F|在每个输出项后追加文件的类型标识符，具体含义|“*”表示具有可执行权限的普通文件，“/”表示目录，“@”表示符号链接，“|”表示命令管道FIFO，“=”表示sockets套接字。当文件为普通文件时，不输出任何标识符；
-c|与“-lt”选项连用时，按照文件状态时间排序输出目录内容，排序的依据是文件的索引节点中的ctime字段。与“-l”选项连用时，则排序的一句是文件的状态改变时间；
-d|仅显示目录名，而不显示目录下的内容列表。显示符号链接文件本身，而不显示其所指向的目录列表；
-l|以长格式显示目录下的内容列表。输出的信息从左到右依次包括文件名，文件类型、权限模式、硬连接数、所有者、组、文件大小和文件的最后修改时间等；
-t|用文件和目录的更改时间排序；
-R|递归处理，将指定目录下的所有文件及子目录一并处理，相当于目录下文件及文件夹全遍历；

ls 命令也支持正则查询：

- (?)代表一个字符
- (*)代表零个或多个字符

例如，目录下有两个文件：build.gradle，build.sbt，输入以下命令可以从目录中过滤出这两个文件:

```bash
BenjamiungdeMBP:java_api_client benjamin$ ls -al build*
-rw-r--r--  1 benjamin  staff  2859  6 17 11:34 build.gradle
-rw-r--r--  1 benjamin  staff   849  6 17 11:34 build.sbt
```

## 处理文件

### 创建文件

touch 命令有两个作用：

1. 用于把已存在文件的时间标签更新为系统当前的时间（默认方式），它们的数据将原封不动地保留下来。
2. 用来创建新的空文件。创建的文件大小为0，表示它是空文件。

### 复制文件

cp 命令将文件和目录从一个位置复制到另一个位置。

cp命令需要两个参数——源对象和目标对象：

```bash
cp source destination
```

以下列出常用的参数以及用途:

参数|用途
---|:--:
-d|当复制符号连接时，把目标文件或目录也建立为符号连接，并指向与源文件或目录连接的原始文件或目录；
-f|强行复制文件或目录，不论目标文件或目录是否已存在；
-i|覆盖既有文件之前先询问用户；
-l|对源文件建立硬连接，而非复制文件；
-p|保留源文件或目录的属性；
-R/r|递归处理，将指定目录下的所有文件与子目录一并处理；
-s|对源文件建立符号连接，而非复制文件；

### 链接文件

Linux具有为一个文件起多个名字的功能，称为链接。被链接的文件可以存放在相同的目录下，但是必须有不同的文件名，而不用在硬盘上为同样的数据重复备份。

另外，被链接的文件也可以有相同的文件名，但是存放在不同的目录下，这样只要对一个目录下的该文件进行修改，就可以完成对所有目录下同名链接文件的修改。

对于某个文件的各链接文件，我们可以给它们指定不同的存取权限，以控制对信息的共享和增强安全性。

链接是目录中指向文件真实位置的占位符，有两种类型的文件链接：

- 符号链接，会创建一个新文件，它指向存放在虚拟目录结构中某个地方的另一个文件，这两个通过符号链接在一起的文件，彼此的内容并不相同。
- 硬链接，会创建独立的虚拟文件，其中包含了原始文件的信息及位置，但他们从根本上而言是同一个文件，引用硬链接文件等同于引用了源文件。

具体操作查看：[ln命令](https://man.linuxde.net/ln)

### 重命名文件

在 Linux 中，重命名文件称为移动(moving)，mv 命令可以将文件和目录移动到另一个位置或重新命名。

```bash
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ touch empty
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ ls
empty
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ mv empty empty1
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ ls
empty1
```

### 删除文件

在 Linux 中，删除叫做移除(removing)。bash shell 中删除文件的命令是 rm。

```bash
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ ls
empty1
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ rm -i empty1 
remove empty1? y
```

注意，-i 参数提示你是不是要真的删除该文件。bash shell 中没有回收站或垃圾箱，文件一旦删除，就无法再找回。

## 处理目录

### 创建目录

使用 mkdir 命令即可在 Linux 中创建目录。

```bash
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ mkdir new_dir
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ ls -al
total 16
drwx------+  6 benjamin  staff   192  6 22 20:29 .
drwxr-xr-x+ 50 benjamin  staff  1600  6 22 16:04 ..
drwxr-xr-x   2 benjamin  staff    64  6 22 20:29 new_dir
```

如果想同时创建多个目录和子目录，可以加入 -p 参数：

```bash
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ mkdir -p new_dir/new_dir_son
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ ls -R
new_dir

./new_dir:
new_dir_son

./new_dir/new_dir_son:

```

### 删除目录

删除目录的基本命令是 rmdir。

```bash
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ rmdir new_dir/
rmdir: new_dir/: Directory not empty
```

默认情况下，rmdir 命令只能删除空目录，因为 new_dir 下面还有一个文件，所以 rmdir 命令拒绝删除目录。

也可以在非空目录上使用 rm 命令，使用 -r 参数可以使得命令进入目录，删除其中的文件，然后再删除目录本身。

```bash
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ rm -ri new_dir/
examine files in directory new_dir/? y
examine files in directory new_dir//new_dir_son? y
remove new_dir//new_dir_son? y
remove new_dir/? y
```

也有一个更加方便，但同时非常危险的命令：rm -rf，务必谨慎使用。

```bash
BenjaminYoungdeMacBook-Pro:Desktop benjamin$ rm -rf new_dir/
```

## 查看文件内容

### 查看文件类型

file 命令能够探测文件的内部，并决定文件是什么类型的：

```bash
BenjaminYoungdeMacBook-Pro:Downloads benjamin$  file test
test: ASCII text
```

### 查看整个文件

1. cat 命令是显示文本文件中所有数据的得力工具。

```bash
BenjaminYoungdeMacBook-Pro:Downloads benjamin$ cat test
a
b
c
d
e
```

-n 参数可以给所有的行加上行号。

```bash
BenjaminYoungdeMacBook-Pro:Downloads benjamin$ cat -n test
     1	a
     2	b
     3	c
     4	d
     5	e
```

2. more 命令会分页显示文本文件的内容。你可以通过按空格键或回车键以逐行向前的方式浏览文本文件，浏览完之后，按 q 键退出。

3. less 命令为 more 命令的升级版，它提供了一些极为实用的特性，能够实现在文本文件中前后翻动，而且有一些高级搜索功能。

### 查看部分文件

1. tail 命令会显示文件最后几行的内容，默认情况下，它会显示文件的末尾 10 行。

```bash
BenjaminYoungdeMacBook-Pro:Downloads benjamin$ tail test
a
b
c
d
e
a
b
c
d
e
```

-f 参数是 tail 命令的一个突出特性，它允许你在其他进程使用该文件时查看文件的内容，会保持活动状态，不断显示添加到文件中的内容。

2. head 命令会显示文件开头那些行的内容，它会显示文件前 10 行的文本。

```bash
BenjaminYoungdeMacBook-Pro:Downloads benjamin$ head test
a
b
c
d
e
a
b
c
d
e
``` 