
在Linux文件系统中经常提及硬链接(Hard Link)和符号链接(Symbolic Link)，Windows中也可以创建链接，但由于丰富的图形界面操作，很少提及链接。Windows 的 NTFS 文件系统支持三种链接：硬链接(Hard Link)、符号链接(Symbolic Link)和目录链接(junction point)，此外还有一个大家非常熟悉链接机制：快捷方式。


## 创建链接


创建链接可以通过 dos 命令 mklink 或者 powershell 中的New\-Item 创建。
mklink命令的使用说明如下图所示。默认是创建文件符号链接，使用`/D` 参数则是创建目录的符号链接，使用 `/H` 是创建硬链接，使用`/J`是创建目录联结，也称为软链接(soft link)。
[![image](https://img2024.cnblogs.com/blog/3056716/202409/3056716-20240930174613020-392078853.png)](https://github.com)


使用 powershell 创建链接的方式如下:



```
New-Item <链接路径> -ItemType <链接类型> -Target <链接目标>

```

其中 ItemType 的取值可选：HardLink、SymbolicLink、Junction


## 几种链接的区别


与 Linux 的文件系统中的 inode 与 block 类似，在 NTFS 文件系统中数据对象也赋予了独一无二的**文件 ID** 以及**与之对应的文件路径**，文件路径和文件 ID 对应，文件 ID 和数据对象绑定，最终才呈现为可供用户打开、编辑的文件。


### 快捷方式(shortcut)


快捷方式以`.lnk`文件方式存在，文件大小仅有几百字节，与原始文件大小无关。适用于 Explorer 等应用程序，并非 NTFS 内置机制，从Win95开始得到支持。FAT32也支持。适用于文件、目录，只能使用绝对路径。可以跨盘符，可以跨主机，可以使用UNC路径、网络驱动器。


### 符号链接


符号链接是将自己链接到一个目标文件或目录的路径上。当系统识别到符号链接时，它会跳转到符号链接所指向的目标中去，而不改变此时的文件路径。
[![image](https://img2024.cnblogs.com/blog/3056716/202409/3056716-20240930174641480-1524142002.png)](https://github.com)


符号链接从Vista开始得到支持，NTFS内置机制。适用于文件，目录。可以理解为另一种形式的快捷方式(shortcut)，文件大小为0字节和不占用空间。可以使用相对/绝对路径，可以跨盘符，跨主机，可以使用UNC路径和网络驱动器。


### 硬链接


硬链接和符号链接的原理完全不同，符号链接是指向目标路径的链接，而硬链接则是指向目标数据对象的链接。因为一个卷中的数据对象都有一个独一无二文件 ID，也可以说硬链接是指向目标文件 ID 的链接。
[![image](https://img2024.cnblogs.com/blog/3056716/202409/3056716-20240930174659695-1107612036.png)](https://github.com)


硬链接从Windows NT4开始得到支持，是NTFS内置机制，FAT32不支持。只适用于文件，只能使用绝对路径。本身无文件，不占用额外空间。hardlink与targetfile必须位于同一卷，可以简单理解成不能跨盘符。


### 目录联接


目录联接从Windows2000/XP开始得到支持，是NTFS内置机制。只适用于目录。只能使用绝对路径。目录链接通过[重分析点](https://github.com):[蓝猫机场](https://fenfang.org)实现，目录链接可以跨卷，但是不能跨主机。


### 详细对比


几种链接方式详细比较如下表所示




|  | shortcut | hard link | junction point | symbolic link |
| --- | --- | --- | --- | --- |
| 创建方式 | 右键 \-\> 创建快捷方式 | mklink /H Link Target | mklink /J Link Target | mklink /D Link Target |
| 存在方式 | 以.lnk文件方式存在，适用于Explorer等应用程序。非NTFS内置机制，从Win95开始得到支持。FAT32支持。 | NTFS内置机制，从Windows NT4开始得到支持。FAT32不支持。 | NTFS内置机制，从Windows2000/XP开始得到支持。是 NTFS 3\.0 及以上文件系统（Windows 2000 及以上系统）的特性，它是链接本地目录（可跨卷）的访问点，通过交接点的操作都会被系统映射到实际的目录上。通过建立交接点，可以在保证一个目录实例（目录的一致性）的前提下，允许用户或程序从本地文件系统中的多个位置访问此目录。 | NTFS内置机制，从Vista开始得到支持。文件类型是.SYMLINK |
| 适用范围 | 同时适用于文件、目录，只能使用绝对路径。 | 只适用于文件，只能使用绝对路径。 | 只适用于目录。只能使用绝对路径。即使创建junction point时使用了相对路径，保存到NTFS中时将隐式转换成绝对路径。 | 同时适用于文件、目录。这是一种超级shortcut，文件大小为0字节和不占用空间。 |
| 使用限制 | 可以跨盘符，可以跨主机，可以使用UNC路径、网络驱动器。 | hard link与targetfile必须位于同一volume，可以简单理解成不能跨盘符。 | junction point必须与target directory位于同一local computer，可以简单理解成不能跨主机, 在local computer范围内，可以跨盘符。不能使用UNC路径；假设Z是通过网络映射生成的盘符，同样不适用于Z。 | 可以使用相对、绝对路径。假设创建symbolic link时使用了相对路径，保存到NTFS中的就是相对路径，不会隐式转换成绝对路径。可以跨盘符，可以跨主机，可以使用UNC路径、网络驱动器。 |
| 移动能力 | 本身有文件，可以复制，移动等操作。 | / | / | / |
| 文件 | 文件大小仅有几百字节, 跟原文件大小无关，文件类型是.lnk。 | 本身无文件，为文件创建多入口。由于不同的文件指向的是同样的数据，所以无论给同一个文件创建多少个硬链接，他们占整个卷的数据大小都是一样的。 | 对交接点内文件和子目录的“建立、删除、修改”等操作都被映射到对应的目录中的文件和子目录上，对交接点的“复制、粘贴、剪切、配置 ACL”，只会影响此交接点，在同一卷内移动交接点，只会影响此交接点，但在不同卷间移动交接点，会将此交接点转换为正常目录，并且交接点对应目录下的所有内容都会被移动。 | 符号链接（Symlink，Softlink）是对文件或目录的引用，实际上符号链接本身是一个“记录着所引用文件或目录的绝对或相对路径”的特殊文件，通过符号链接的操作都会被重定向到目标文件或目录。对符号链接和快捷方式的“读、写、遍历”等操作都会被重定向到目标文件或目录，但对它们的“复制、删除、移动、配置 ACL”等操作只针对自身。 |
| 关联 | 删除shortcut，不影响target。 | 在Explorer中删除hard link，不影响targetfile。删除target file，不影响hardlink。事实上由于hard link的语义，此时剩下的hardlink就是原始数据的唯一访问点。只有当一个文件 ID 对应的所有硬链接被删除时，数据才真正被标记为删除。 | 删除target directory，junction point仍将存在，但失效了，变得不可用。这个很好理解，因为此时junction point指向不存在的目录。 | 在Explorer中删除symboliclink，不影响target。删除target，symboliclink仍将存在，但失效了，变得不可用。它们可以像普通文件一样操作，但所有对符号链接的操作都实际作用于目标对象。符号链接对用户而言是透明的，符号链接看上去和普通的文件和文件夹没有区别，操作方法也一模一样（更类似于 Linux 的软链接）。 |


## 链接的应用


* 硬链接：可以在不复制文件的情况下，实现文件的快速访问以及文件的备份，还可以防止重要文件误删，因为删除的是文件的链接，而非文件数据本身。
* 符号链接：可以把一个路径映射到另一个路径，或者指向远程文件或目录，甚至可以通过网络连接到其他计算机上的文件。
* 目录联接：实现路径重定向，当访问链接目录时，系统会自动重定向到实际目录，例如：Vista的"C:\\Documents and Settings"是指向"C:\\Users"的junctionpoint，这样一些使用了硬编码"C:\\Documents and Settings"的老程序可以在Vista上正常工作。此外，还可以解决Windows文件路径长度限制带来的问题（从 Windows 10 版本 1607 开始，可以通过设置注册表以及应用程序清单启用长路径）。


## 参考


1. [https://learn.microsoft.com/zh\-cn/windows/win32/fileio/hard\-links\-and\-junctions](https://github.com)
2. [https://learn.microsoft.com/en\-us/powershell/module/microsoft.powershell.management/new\-item?view\=powershell\-7\.4](https://github.com)
3. [https://learn.microsoft.com/zh\-cn/windows/win32/fileio/maximum\-file\-path\-limitation?tabs\=registry\#enable\-long\-paths\-in\-windows\-10\-version\-1607\-and\-later](https://github.com)


