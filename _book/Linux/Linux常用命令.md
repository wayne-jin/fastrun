# linux基本使用

------

### linux常见命令的使用

#### sudo

需要根权限的每一个命令都需要这个sudo命令。你可以在需要根权限的每个命令之前使用sudo。

#### ls(list)

终端会显示目录里面的所有文件和文件夹。假设我在/home文件夹里面，想查看/home里面的目录和文件。

/home$ ls

#### cd

更改目录(cd)。输入你想要进入到的那个文件夹的名称。如果想要返回上一级，只要将双圆点(..)作为参数。

假设我在/home目录中，想进入到始终在/home里面的usr目录。下面是我可以使用cd命令的方法：

/home $ cd usr /home/usr $

#### mkdir

创建文件夹或子文件夹。

$ mkdir folderName

#### cp

拷贝粘贴。

$ cp src des

#### rm

rm删除文件。如果文件需要根权限才能移除，可以使用-f。使用-r来进行递归移除，从而移除你的文件夹。

$ rm myfile.txt

#### apt-get

不同的发行版，这个命令各不相同。基于Debian的Linux发行版中，想安装、移除和升级任何软件包，我们可以使用高级包装工具(APT)软件包管理器。apt-get命令可帮助你安装需要在Linux中运行的软件。

在其他发行版(比如Centos)中，有不同的软件包管理器。Centos可以使用yum。

```
$ sudo apt-get update
$ sudo yum update
```

#### grep

使用grep命令，根据给定的关键字帮助找到文件。

$ grep user /etc/passwd

#### cat

显示文件里面的文本。

$ cat CMakeLists.txt

### linux下编译环境的安装和使用

#### 安装g++

sudo apt-get install g++

查看版本

g++ --version(或g++ -v)