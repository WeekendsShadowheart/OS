# 操作系统专题实验实验三
## 模拟ext2系统基本概念
  ext2是Linux操作系统中广泛使用的一种文件系统，设计简单高效，适用于存储和管理文件。它支持文件的权限管理、链接和目录结构，并具有较高的稳定性和灵活性。

ext2采用块block作为基本存储单位，每个文件被分割成多个块存储，块的大小可以是1KB、2KB或4KB，支持文件大小和分区大小的灵活调整。ext2的文件系统结构包括超级块superblock、组描述符group desc、块位图block bitmap、inode位图inode bitmap、inode表inode table以及数据块。

其中，超级块记录文件系统的全局信息，如总块数、空闲块数等；组描述符用于描述每个块组的状态；块位图和inode位图分别管理块和inode的分配情况；inode表存储文件的元数据；数据块则存放实际的文件数据。

## 代码实现
  本项目实际上是一个简化版的类EXT2文件系统的实现，通过一个磁盘文件weekends.img
  来模拟真正的磁盘，并在其上管理简单的inode、数据块、目录等结构。
  
### 1.使用的数据结构与数据结构设计

  GroupDesc：在本项目中只定义一个组。GroupDesc只占用一个“块”。同时，超级块省略，其功能由组描述符块代替，包括了需要增加的卷名、文件系统大小以及索引结点的大小等。它的具体实现形式如下：
  
![1.1步骤1 步骤2](./assets/3/GroupDesc.png)

  Inode：在EXT2文件系统中,索引节点inode用于描述一个文件或目录的基本信息。为了简化，本项目仅实施了一部分：
  
![1.1步骤1 步骤2](./assets/3/Inode.png)

  DirEntry：用于目录的数据块中，存储单个文件或子目录的基本信息。“目录”就是由若干个 DirEntry 依次排列构成：
  
![1.1步骤1 步骤2](./assets/3/DirEntry.png)

### 2.内存的分配与释放

  allocate_inode:从 inode 位图中分配或释放一个 inode 编号,先加载组描述符，
  检查 free_inodes_count 是否为 0, 
  从 inode bitmap 里调用 bitmap_find_zero 找到空位,
  将该位翻转为 1，表示已占用；更新 inode bitmap 到磁盘,
  组描述符 free_inodes_count-- 并写回磁盘。
  
![1.1步骤1 步骤2](./assets/3/allocate_inode.png)

  free_inode：
  
![1.1步骤1 步骤2](./assets/3/free_inode.png)

  alloc_free_block：类似于alloc_free_inode, 分配时找空闲位、置 1 并把块内容清零；释放则置 0 。
  
![1.1步骤1 步骤2](./assets/3/alloc_free_block.png)


### 3.初始化

  init_disk_file：把所有磁盘块写零；
  写入组描述符；
  分配第一个 inode 作为根目录 root_ino=1；
  为根目录分配一个数据块，并在其中写入 “.” 和 “..” 两个目录项；
  更新根目录 inode 大小、时间，组描述符等
  
![1.1步骤1 步骤2](./assets/3/init_disk_file.png)

  init_memory：初始化内存中的全局变量，包括组描述符、打开表、当前目录、当前路径等。
  
![1.1步骤1 步骤2](./assets/3/init_memory.png)


### 4.命令函数

shell_loop：在交互式环境中解析用户输入的命令行，分发给对应函数执行。输入 quit 则退出循环。

![1.1步骤1 步骤2](./assets/3/shell_loop.png)

login指令：

![1.1步骤1 步骤2](./assets/3/login指令.png)

change_password指令：

![1.1步骤1 步骤2](./assets/3/change_password指令.png)

format指令：

![1.1步骤1 步骤2](./assets/3/format指令.png)

check指令：

![1.1步骤1 步骤2](./assets/3/check指令.png)

ls指令：

![1.1步骤1 步骤2](./assets/3/ls指令.png)

mkdir指令：

![1.1步骤1 步骤2](./assets/3/mkdir指令.png)

rmdir指令：

![1.1步骤1 步骤2](./assets/3/rmdir指令.png)

create指令：

![1.1步骤1 步骤2](./assets/3/create指令.png)

delete指令：

![1.1步骤1 步骤2](./assets/3/delete指令.png)

cd指令：

![1.1步骤1 步骤2](./assets/3/cd指令.png)

chmod指令：

![1.1步骤1 步骤2](./assets/3/chmod指令.png)

open指令：

![1.1步骤1 步骤2](./assets/3/open指令.png)

close指令：

![1.1步骤1 步骤2](./assets/3/close指令.png)

write指令：

![1.1步骤1 步骤2](./assets/3/write指令.png)

read指令：

![1.1步骤1 步骤2](./assets/3/read指令.png)


### 5.main函数

main函数：打开/创建模拟磁盘文件 ext2_disk.img；
如果文件是新建或检测到无效文件系统，则进行格式化；
要求用户输入密码通过 cmd_login 认证；
进入命令行循环交互；
退出时关闭磁盘文件并结束程序。

![1.1步骤1 步骤2](./assets/3/主函数.png)

## 实验结果以及改进
### 实验结果呈现

login_quit：

![1.1步骤1 步骤2](./assets/3-2/login_quit.png)

错误指令发现：

![1.1步骤1 步骤2](./assets/3-2/错误指令发现.png)

create_file：

![1.1步骤1 步骤2](./assets/3-2/create_file.png)

ls_cd：

![1.1步骤1 步骤2](./assets/3-2/ls_cd.png)

file_write：

![1.1步骤1 步骤2](./assets/3-2/file_write.png)

file_read：

![1.1步骤1 步骤2](./assets/3-2/file_read.png)

### 使用SIMD加速
SIMD，Single Instruction, Multiple Data，是一种并行计算技术，可在单个指令周期内对多个数据进行相同的操作。
常用于多媒体处理、矩阵运算、图像处理等场景。对于本实验，可以用它来加速for语句和可以并行的结构。

使用SIMD修改部分函数：

![1.1步骤1 步骤2](./assets/3-2/使用SIMD修改.png)

### 使用OpenMP加速
OpenMP是一个支持多线程并行编程的 API，广泛应用于共享内存多处理器系统中。通过简单的编译指令和函数调用，开发者可以快速将代码并行化，实现线程分配、任务划分、同步控制等功能。对于本实验，可以用它来加速for语句和可以并行的结构。

使用OpenMP修改部分函数：

![1.1步骤1 步骤2](./assets/3-2/使用SIMD修改.png)



### 实验遇到问题以及改进

在进行实验的过程中，我有很多次遇到了segment fault，以下是部分错误记录：

![1.1步骤1 步骤2](./assets/3-2/valgrind错误1.png)

![1.1步骤1 步骤2](./assets/3-2/valgrind错误2.png)

![1.1步骤1 步骤2](./assets/3-2/错误代码示例.png)

实验过程中我使用了valgrind对错误进行了调查，大部分错误的原因都是因为内存泄露或者双重释放。
这些错误都可以通过更灵活地使用malloc函数分配内存来解决。同时，对于实验中出现的双重释放内存问题，我通过仔细检查free()函数等方法加以解决。


### 实验参考教学
https://www.bilibili.com/video/BV1V84y1A7or/?spm_id_from=333.337.search-card.all.click&vd_source=6e5ec0eed316e576942858c39a096cc3

https://www.bilibili.com/video/BV1jr421c7rL/?spm_id_from=333.337.search-card.all.click&vd_source=6e5ec0eed316e576942858c39a096cc3
