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