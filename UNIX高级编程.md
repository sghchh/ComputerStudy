## 一、文件IO

本章节描述的函数都是**不带缓冲的IO，不带缓冲指的是每个read和write都调用内核中的一个系统调用**。

### 1. 文件描述符

对于内核而言，所有打开的文件都通过文件描述符引用；文件描述符是一个非负整数，其中0、1、2被用来表示标准输入、标准输出、标准错误三个。

### 2. 打开文件函数

该功能由open与openat函数实现：

```c
int open(const char *path, int open_flag, mode_t mode);
int openat(int fd, const char *path, int open_flag, mode_t mode);
```

其中path表示目标文件的路径名。结合该参数与openat中的fd参数，讲解一下openat确定文件的机制：

* 如果在openat中的path参数是一个绝对路径的话，那么此时fd参数没有作用，其效果和open函数一样
* 如果openat的path参数是一个相对路径的话，那么此时的fd所打开的文件的目录路径用来与path拼接成一个绝对路径，最终打开的文件由该绝对路径指定
* 如果openat的path参数是一个相对路径，而且fd为特殊常量AT_FDCWD，那么此时表示在当前工作目录的基础上拼接path成一个绝对路径

其中，参数open_flag用来指明打开文件的权限：读、写、读+写等。

若**打开文件出错，该函数返回-1**。

### 3. 创建文件

create函数用来创建一个文件，并且以只写的权限打开该文件：

```c
int create(const char *path, mode_t mode);
```

**若创建文件出错，该函数返回-1**。

### 4. 关闭文件

close函数用来关闭一个文件：

```c
int close(int fd);
```

**若关闭失败，则返回-1**。

### 5. 调整文件偏移量

函数lseek用来调整当前光标在文件中的位置，即**当前文件偏移量(其类型为off_t)**，其以字节为单位。打开文件时，除非open_flag中添加了O_APPEND，否则其默认当前的偏移量为0，而读写操作会自动更改的当前文件偏移量的位置。我门可以使用lseek函数调整当前文件偏移量：

```c
off_t lseek(int fd, off_t offset, int where);
```

其中where有三个值可以选择：

* SEEK_SET：将当前文件偏移量设置为从文件开始处offset个字节
* SEEK_CUR：将当前文件偏移量设置为从当前位置往后offset个字节
* SEEK_END：将当前文件偏移量设置为从文件末尾向前offset个字节

若**调整失败，函数返回-1**。

> 注意：文件偏移量可以大于文件当前的长度，如果此时对文件进行写入操作，中间会形成一个空洞，但是在磁盘中进行存储的时候，并不会因为有一个空洞而浪费空间，并不会为空洞分配磁盘块。

### 6. 文件读

从文件中读数据的函数为read函数：

```c
ssize_t read(int fd, void *buffer, size_t nbytes);
```

当成功读取数据后，该函数的返回值为实际读取到的字节数，当已到文件尾时返回0，如果出错则返回-1.**注意该函数会修改当前文件偏移量**。

该方法负责将数据读取到参数buffer中，需要提醒的是，这个参数buffer的意义是将读取到的数据填充到buffer数组中；**而与本章开头讲到的非缓冲IO并无关联**。

### 7. 文件写

write函数与read函数相对应：

```c
ssize_t write(int fd, void *buffer, size_t nbytes);
```

该函数的注意点与特点都与read函数相对应。

### 8. 常见原子操作

#### 8.1 追加写

在调用open或者openat函数时，如果参数open_flag设置了O_APPEND标签，那么每次执行写操作时都会自动将当前文件偏移量设置为该文件的末尾，并且这是一系列是一个原子操作。

#### 8.2 pread与pwrite函数

这两个函数的原型如下：

```c
ssize_t pread(int fd, void *buf, size_t nbytes, off_t offset);
ssize_t pwrite(int fd, void *buf, size_t nbytes, off_t offset);
```

这两个函数的作用相当于先调用lseek函数改变当前文件偏移量，之后进行读或者写操作，区别有以下两点：

* 上面两个函数具有原子性，其定位于读写操作无法中断
* 不会更新当前文件偏移量

### 9. 文件写入磁盘控制

当我们向文件写入数据时，数据到达磁盘的过程通常是：**内核将数据复制到缓冲区、排入队列、晚些时候写入磁盘**。

```c
int fsync(int fd);
int fdatasync(int fd);
```

这两个函数可以干预文件数据写入磁盘的时机，当函数调用时将所有修改过的块缓冲区排入写队列，并且等待写磁盘操作结束才返回；二者的区别是，fdatasync只会影响文件的数据部分，而fsync还会更新文件的属性。



## 二、标准IO

标准IO将IO操作进一步的抽象了，**所有的IO操作只是简单地从程序移进或移除字节**，因此，这种字节流就被成为**流(Stream)**。

而标准IO不同于第一章的文件IO，其都是围绕流展开的。而**stdio.h中专门定义了一个FILE结构，在这里FILE是一个数据结构，用来访问一个流**。注意，在本章中“文件”一词就是指这个FILE结构，并不是磁盘上的文件。

有多种字符集，一个字符可以用一个字节或者多个字节表示，因此，标准IO流可以指定为多字节还是单字节字符集，称其为**流的定向**。

``` c
int fwide(FILE *fp, int mode);
```

该函数用来将一个**还未定向的流设置定向**：

* mode为负，则尝试设置为字节定向
* mode为正，则尝试设置为多字节定向
* mode为0，不做改动

注意，**该函数不会更改已有定向的流的设置，只能为那些还未设置定向的流设置**。对于一个未定向的流，不执行该函数，如果执行了其他的、带有定向特征的函数，那么会自动为该流设置为相应的定向。比如，一个未定向的流执行了一个多字节的read函数，那么该流就自动被设置为多字节定向了。

### 1. 缓冲

第一章中的文件IO都是非缓冲的IO，而这里的标准IO是可以设置缓冲的类型的。

* 全缓冲：该情况下，在填满标准IO的缓冲区后才进行实际的IO操作
* 行缓冲：该情况下，当流中遇到换行符时或者缓冲区已经满了，标准IO就执行实际的IO操作
* 不带缓冲：

在系统默认情况下：

1. 标准错误不带缓冲
2. 指向终端设备的流要么是行缓冲、要么是全缓冲

当然，我们也可以更改系统的默认行为：

```c
void setbuf(FILE *restrict fp, char *restrict buf);
int setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
```

这两个函数中，第一个只能选择打开或者关闭缓冲，但是不能设置缓冲的模式。如果想要关闭缓冲，只需要将buf参数设置为NULL。而第二个函数的mode可以设置缓冲的类型：**_IOFBF、_IOLBF、_IONBF分别代表全缓冲、行缓冲、不带缓冲**。还有一个重要的区别是，第一个函数不能设置缓冲的长度，其缓冲的长度采用的是stdio.h中BUFSIZ常量的值；而第二个函数可以通过size参数设置缓冲的长度，如果buf参数为NULL的话，那么此时其缓冲依然会被自动分配，其长度也是BUFSIZ常量的值。

> 注意：这里缓冲区设置时，其参数是通过一个指针传递的，但是在C语言中，并不能保证通过一个指针访问一块内存时不会出现越界的情况；所以此时第二个函数中的指定缓冲区长度size、保证不会越界就显得很重要了。

### 2. 打开流

打开一个流，有如下三个函数：

```c
FILE *fopen(const char *restrict pathname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const *restrict type, FILE *restrict fp);
FILE *fdopen(int fd, const char *type);
```

其中第二个和第一个的区别就是，第二个在一个指定流上打开该文件，通常这个指定流是标准输入输出流，而如果fp对应的流是一个定向流，那么首先清除该定向，因此，**三个函数都是返回一个非定向流**。

而第三个函数用来将一个**文件描述符**包装为一个流，因为文件描述符所指代的数据源范围更广，在一些特定的数据输入和输出的情况下需要使用文件描述符打开，而fdopen函数则可以将文件描述符与标准IO流结合起来。

而第二个参数用来执行对IO流的读写方式：

* r、rb：只读
* w、wb：只写
* a、ab：末尾追加
* r+、r+b或rb+：读写
* w+、w+b或wb+：文件截断至0，读写
* a+、a+b或ab+：文件尾部读写（当文件不存在时可以创建文件）

上面各对中含有b与不含有b用来区分是文本文件还是二进制文件。

```c
int fclose(FILE *fp);
```

### 3. 字符读、写流

```c
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);
```

其中getchar等同于getc(stdin)；上面三个函数**在遇到出错或者到达文件尾时，都会返回EOF**，想要区分这两种情况需要另外两个函数：

```c
int ferror(FILE *fp);
int feof(FILE *fp);
```

从流中读取数据之后，可以调用ungetc将字符再压回流中：

```c
int ungetc(int c, FILE *fp);
```

该函数只能一个一个字符回压，而且这种压入并不是写入到底层文件中，而是写入缓冲区中。

与读流相对应的三个函数：

```c
int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c);
```

### 4. 行读、行写

```c
char *fgets(char *restrict buf, int n, FILE *restrict fp);
cahr *gets(char *buf);
```

其中gets默认输入流为stdin。

对应的输出流为：

```c
int fputs(const char *restrict str, FILE *restrict fp);
int puts(const cahr *str);
```

> 注意：虽然这里行读和行写的参数中都有一个缓冲区，但是这个与setbuf中的不是同一个
>
> 关于缓冲区，请阅读该文章：[C 标准库IO缓冲区 内核缓冲区](https://blog.csdn.net/sole_cc/article/details/47983225)

### 5. 二进制IO

二进制IO有点像Java中的Object IO，这里负责读写**多个结构**：

```c
size_t fread(void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
size_t fwrite(void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
```

其中，ptr一般为一个数组指针或者malloc申请的内存块，用来存放所有结构数据，size参数为每一个结构数据的尺寸，nobj参数为所有结构数据的个数，fp参数为想要写入数据或者从中读出数据的流。

同上面的读写流一样，**当遇到错误或者流的尾部的时候，会返回EOF，想要区分就得调用ferror或者feof方法**。

### 6. 定位流

标准IO库中也提供了一些方法可以设置在流中的当前位置：

```c
long ftell(FILE *fp);
int fseek(FILE *fp, long offset, int where);
void rewind(FILE *fp);
```

在一个文件打开时，文件指示器默认从起始位置开始，并且以**字节为度量单位**。

* ftell返回当前流中文件指示器的位置，出错返回-1L
* fseek用于设置文件指示器的位置，其中的where同上一章中的lseek一样，SEEK_SET、SEEK_CUR、SEEK_END分别表示文件的开始、当前指示器位置、文件的末尾
* rewind用来将指示器的位置设置为文件开头

还有两个很想的函数，无非在于描述文件指示器位置的数据类型不同，采用的是off_t类型：

```c
off_t ftello(FILE *fp);
int fseeko(FILE *fp, off_t offset, int where);
```

而，在C的标准库中，也提供了类似于打标签的函数：

```c
int fgetpos(FILE *restrict fp, fpos_t *restrict pos);
int fsetpos(FILE *fp, const fpos_t *pos);
```

其中fsetpos用来将当前的文件指示器的位置存入参数pos中，而fgetpos函数用来获取上一次fsetpos设置的位置。

## 三、高级IO

### 1. 非阻塞IO

前面在说IO的时候，大都是指**低速系统调用**，其可能会使得进程永远阻塞。而非阻塞IO使我们可以发出open、read、write这样的IO操作时，使得这些操作永远不会被阻塞，如果这些操作不能完成，则直接返回出错。

对于一个文件描述符，有两种方式为其制定非阻塞IO：

* 在调用open获取描述符时，指定O_NOBLOCK标志
* 对于一个已经打开的描述符，调用fcntl开启O_NOBLOCK标志

### 2. 记录锁

**记录锁**的功能是：**当第一个进程正在读或修改文件的某个部分时，使用记录锁可以组织其他进程修改同一个区域。**记录锁为文件的并发操作提供了安全保障。

#### 文件记录锁结构

在介绍加锁的函数原型之前，我们需要先看一看文件锁结构是如何定义的，即是如何描述一个记录锁的。

```c
struct flock {
	short l_type;
	short l_whence;
	off_t l_start;
	off_t l_len;
	pid_t l_pid;
}
```

各结构的说明如下：

* l_type：为锁类型，可选的值有：F_RDLCK(共享读锁)、F_WRLCK(独占写锁)、F_UNLCK(解锁一个区域)，这点需要注意，**记录锁解锁的方式为该字段设置为F_UNLCK，并不是调用其他的函数**
* l_whence:同之前的lseek函数一样，该字段可选的值为：SEEK_SET、SEEK_CUR、SEEK_END，表示从哪里算起
* l_start:相对于l_whence指示的比较标准的偏移字节量
* l_len：表明加锁区域的长度，为0代表锁的范围可以是最大偏移量
* l_pid：所属进程的id

其中，对于单个进程提出的多个锁请求，如果一个进程对于一个文件区域已经有了一把锁，又企图在同一个文件区域再加一把锁，那么**新锁会替换掉已有的锁**(如果能够加锁的话)。

#### 记录锁原型

在一个描述符的基础上增设记录锁的函数原型如下：

```c
int fcntl(int fd, int cmd, struct flock *flockptr);
```

其中的fd和flockptr参数没什么说的，这里的cmd参数支持三种命令：

* F_GETLK：判断由flockptr描述的锁在该文件区域上是否会被另外一把锁所排斥(注意：**这里的另一把锁一般都是其他进程的锁**)。如果存在，那么将flockptr设置为那个锁的信息；如果不存在，那么将flockptr的l_type设置为F_UNLCK。因此，该方法可以用来测试是否可以在一个文件区域上进行加锁操作。
* F_SETLK：在该文件区域上设置由flockptr所描述的锁。如果是尝试加锁，而被其他的锁所排斥，那么该方法会立即出错，将errno设置为EACCES或者EAGAIN
* F_SETLKW：在该文件区域上设置由flockptr所描述的锁。如果是尝试加锁，而被其他的锁所排斥，那么该方法会阻塞，直到其他进程释放了那个排斥的锁。这里F_SETLKW中的W解释为Wait

#### 锁的继承和释放规则

有三条规则：

* 进程终止时，其所持有的锁都会释放；当一个描述符关闭时，基于该描述符的所有锁都会释放(不同的描述符指向同一个文件也会影响)
* fork产生的子进程不继承父进程设置的锁
* 执行exec后，新程序继承原执行程序的锁

### 3. IO多路转接

多路转接的场景为：**需要从多个文件描述符中读入数据，而后将数据写入多个文件描述符**。

不管是多进程、多线程还是异步IO方式，想要是想上面的功能，其复杂度将会非常的大，系统为此提供了多路转接的功能。

#### 描述符集

多路转接的场景涉及到多个描述符，因此我们需要先认识一下描述符集。

描述符集在结构上可以理解为一个很大的字节数组，同时我们可以在描述符集上调用如下几个函数：

```c
int FD_ISSET(int fd, fd_set *fdset);
void FD_CLR(int fd, fd_set *fdset);
void FD_SET(int fd, fd_set *fdset);
void FD_ZERO(fd_set *fdset);
```

从上到下功能分别是：

* 判断fd是否在描述符集中，如果是返回0
* 将fd从描述符集中剔除
* 将fd添加到描述符集中
* 将描述符集清空

#### 多路转接select函数原型

IO多路转接的函数原型为select函数：

```c
int select(int maxfd1, fd_sest *restrict readfds, fd_set *restrict writefds, fd_set *restrict execeptfds, struct timeval *restrict tvptr);
```

其中第一个参数的值为readfds、writefds、execeptfds三个中描述符集**最长的长度加一**。

对于最后一个参数，用来指定愿意等待的时间长度，有如下三种情况：

* NULL：永远等待，如果捕捉到一个信号则中断等待，此时返回-1，errno设置为EINTR；如果有一个描述符已经准备好，则返回准备好描述符的数目。
* typtr->tv_sec != 0 || typtr->tv_usec != 0：等待指定的秒数和微秒数。如果到达等待时间还没有描述符准备好，则返回0；同样也会受到信号影响而中断。
* typtr->tv_sec == 0 && typtr->tv_usec == 0：**不等待，只有此情况下select才不会被阻塞**

上面多次使用*描述符准备好*这种表述，但是却没有说明何为准备好？其定义如下：

* 对于readfds中的描述符进行read操作不会阻塞
* 对于writefds中的描述符进行write操作不会阻塞
* exceptfds中的描述符有一个未决异常条件
* 对于读、写和异常条件，**普通文件的文件描述符总是返回准备好**

### 4. 异步IO

### 5. 存储映射IO

**存储映射IO能将一个磁盘文件映射到存储空间的一个缓冲区上**，这样就可以在不使用read和write的情况下达到执行IO的目的。

#### 映射函数

文件映射的功能是使用mmap函数实现的：

```c
void *mmp(void *addr, size_t len, int prot, int flag, int fd, off_t off);
```

该函数**成功时返回的是最终映射区域的起始地址，出错返回MAP_FAILED**。

关于各个参数的意义如下：

* addr：指定映射存储区域的起始地址，**通常设置为0表示由系统决定**
* fd：想要映射的文件的描述符
* len：想要映射的内容为fd文件中的字节数
* off：想要映射的内容在fd中的起始偏移量
* prot：该参数指定了对于存储映射区的权限要求，可以为PROT_READ、PROT_WRITE、PROT_EXEC的相或操作，也可以是PROT_NONE。但是，**不可以超出在打开fd描述符时的权限**，如果fd是只读打开的，那么就不可以为映射存储区设置WRITE
  * PROT_READ：映射区可读
  * PROT_WRITE：映射区可写
  * PROT_EXEC：映射区可执行
  * PROT_NONE：映射区不可访问

* flag：该参数用于指定对映射区操作的影响，为下面两者之一
  * MAP_SHARED：该标志表示对存储区的操作将影响到原始的文件
  * MAP_PRIVATE：本标志说明对于映射区的操作将导致创建一个该映射文件的一个私有副本，后续对于该存储区的操作都是在该副本上进行，并不会影响源文件

有两个信号与映射区有关：**SIGSEGV和SIGBUS**。

SIGSEGV通常指示进程试图访问对它不可用的存储区，比如对于一个READ的区域执行Write操作，或者越界；SIGBUS表示进程访问一个已经不存在的存储区。

**通过fork函数产生的子进程可以继承父进程的存储映射区，而通过exec函数执行的新程序则不能。**

而关闭存储映射区的函数原型如下：

```c
int munmap(void *addr, size_t len);
```



# 进程

## 一、进程环境

### 1. main函数

C程序总是从main函数开始执行，其函数原型为：

```c
int main(int argc, char *argv[], char *envp[])
```

其中，各个参数的意义如下：

* argc代表命令行参数argv数组中元素的个数
* argv为命令行参数，需要提醒的是**命令行参数中的第一个参数一定是该可执行程序的名字**
* envp为从系统获得的环境变量的信息

在内核中调用main函数之前首先调用一个特殊的**启动例程**，在程序的链接过程中，链接器将该启动例程设置为程序的起始地址；**启动例程从内核获取到命令行参数与环境变量的值**，为main函数的调用做准备。

> 注：**命令行参数传递给程序的工作是exec函数的功能**；
>
> 在大多数情况下，第三个参数并没有实际用途，所以一些标准将其去掉了

而且，启动例程控制当main函数返回后调用exit函数终止进程。

### 2. 退出函数

对于**正常退出(也称为自愿终止)**的一般最终都是调用的exit系列的函数，该系列函数有三个：

```c
void exit(int status);
void _Exit(int status);
void _exit(int status);
```

其中后两个都是立即进入内核，而exit()则先执行一些**清理处理**，包括关闭打开的流等操作，然后返回内核。

> 实际上，exit()函数最终返回内核也是调用的后两个函数中的某一个

#### 终止处理程序

而我们也可以自定义一些函数，由exit()函数自动调用，用于程序在退出之前执行一些操作，称**这些自定义的函数为终止处理程序**。

终止处理程序需要由**atexit函数**进行登记：

```c
int atexit(void (*func)(void));
```

**需要注意的是，exit()在调用这些终止处理程序时，其调用顺序和登记顺序是相反的。而且，终止处理程序的调用发生在标准IO清理程序之前(想想也合理，因为用户在编写这些终止处理程序时可能还需要使用到IO)**

### 3. 环境表

环境表就是main函数的最后一个参数envp，环境表是一个**字符指针数组**，该数组以**NULL指针做结尾**(所以可以明白为什么main函数需要一个argc参数来指明argv的个数，而envp只需要通过判断其元素是否为NULL就可以发现该数组的末尾)；而**环境表中的每一个表项都代表一个以NULL结尾的字符串**。main函数中的envp参数正是该环境表的指针。

### 4. 共享库

共享库的存在使得可执行文件中不在需要包含公用的库函数，只需要在所有进程都可以使用的存储区中保存这种库历程的一个副本；这样程序在第一次执行或者第一次调用某一个库函数时，采用动态链接方法将程序与共享库函数相连接，这样减少了可执行文件的长度。

## 二、进程控制

### 1. 进程标识

进程标识没啥好说的，需要记住几个特殊的标识：

* ID为0的进程是**调度进程，也叫交换进程**，该进程是**内核的一部分**；并不执行磁盘上的程序，因此也叫**系统进程**。
* ID为1的进程是init进程，该进程并不属于内核，**是一个用户进程**；该进程负责读取和系统有关的初始化文件，并引导系统到一个状态。该进程永不会终止，并且拥有超级用户特权。

C语言标准中，提供了一些函数来返回进程的各种标识符：

```c
pid_t getpid();  // 进程ID
pid_t getppid();  // 父进程ID
uid_t getuid();  // 实际用户ID
uid_t geteuid();  // 有效用户ID
gid_t getgid();  // 实际组ID
gid_t getegid();  // 有效组ID
```

### 2. fork()函数

一个进程可以**调用fork()函数创建一个新进程**，即子进程。特殊的一点是，**fork函数会返回两次，在父进程调用该函数创建子进程的地方，该函数返回子进程的ID，而由于子进程是父进程的一个副本，所以在子进程的该函数处返回的是0**。

既然子进程是父进程的一个副本，那么也就是说代码都一样，那么该怎么使用呢？总不能两个进程执行一模一样的过程吧？fork的用法通常有两种：

* 一个父进程希望复制自己，使得子进程与自己同时执行不同的代码段。在网络服务进程中，父进程监听到请求后fork一个子进程去处理该请求
* 一个进程想要执行一个不同的程序。通常是子进程从fork返回之后立刻调用exec

#### 子进程是父进程的副本

子进程获得父进程的数据空间、堆和栈的副本(注意：这些只是副本，并不是共享)；共享的是正文段(很好理解，因为执行的代码一样，所以正文段可以共享)。

### 3. wait系列

子进程在异常或者正常退出时，内核会自动帮其关闭所有打开的描述符等，同时子进程也会生成一个变量，用来保存其生前的一些信息供父进程查看。

一个已经终止、但是其父进程尚未获取终止子进程有关信息的进程被称为僵死进程。

子进程随时都有可能死掉，系统需要提供给系统一个函数用来获取死掉的子进程的信息，这就是wait系列函数

```c
pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
```

首先要说明的是，这个返回的pid_t只是代表是哪一个子进程，**而死掉的进程的信息是储存在参数statloc中的，可以看到这是一个int指针，所以可以知道信息是通过将一个int型数据不同bit位进行特殊编码保存的**。

#### wait函数

该函数调用后，如果所有的子进程都没有死掉，那么会阻塞住；而且，该函数当有一个子进程死掉的时候就会立刻返回，所以该函数永远只能等待下一个死掉的子进程，并没有等待特定子进程死掉的功能。

> wait函数有个大毛病就是会阻塞进程，又因为子进程随时会死掉，如果父进程想要获取子进程的信息而直接调用wait函数的话，很大概率会阻塞住，因此，通常wait函数会搭配信号机制使用：子进程死掉之后，内核会向其父进程发送一个特定的信号，父进程响应这个信号调用wait函数可以立马获得子进程的信息

#### waitpid函数

从名字就能看出，该函数一定会提供等待特定的pid子进程。实际上远不止如此，其参数pid有更丰富的含义：

* pid=-1：等待任意子进程，此时同wait函数一样
* pid>0：等待进程ID为pid的子进程
* pid=0：等待组ID等于调用进程组ID的任一子进程
* pid<-1：等待组ID等于pid绝对值的任一子进程

对于第三个参数options，其可以是0，如果想要提供更加复杂的功能，其也可以是如下三个常量的或运算：

* WNOHANG：此时如果pid指定的子进程没有死，那么waitpid不阻塞，返回0
* WCONTINUED
* WUNTRACED

后两个涉及到作业控制，先不说。

#### wait3与wait4函数

这两个函数与上面两个函数相对应，只不过**多了一个用来保存子进程使用资源的情况的参数**：

```c
pid_t wait(int *statloc, int options, struct rusage *rusage);
pid_t waitpid(pid_t pid, int *statloc, int options, struct rusage *rusage);
```

第三个参数**rusage**表示资源统计信息，保活CPU时间总量、系统CPU时间总量等。

### 4. exec函数

exec函数的功能是执行另一个程序，新程序从其自己的main函数开始执行；调用exec并不创建新的进程，其进程ID也不变，exec只是将一个新程序替换了当前进程的正文段、数据段、堆栈。

## 三、进程间通信

### 1. 管道通信

关于管道通信有如下几点注意事项：

* 为了保证可移植性，**默认管道通信是一个半双工管道**
* 只能在具有公共祖先的两个进程之间使用，即他们必须得由一个进程fork出来

关于管道，我们可以将其想象成一个在内核共享的缓存区，只不过该缓存区的数据是随着读入和写出而不断变化的。

#### 创建管道的函数原型

```c
int pipe(int fd[2]);
```

一个进程创建的管道**被两个文件描述符指代**，fd[0]为读打开，fd[1]为写打开，而且fd[1]的输出是fd[0]。

上面讲到过，在fork子进程的时候，子进程会继承父进程打开的文件描述符，因此，就是根据这一原理，通过管道进行通信的进程必须具有这种关系，即**该管道必须是他们共同的父进程创建的**。在进行通信时，发送方需要关闭自己的管道读端fd[0]，而接收端需要关闭自己的管道写端fd[1]，由于读写端都只是文件描述符，因此关闭他们就是使用的**关闭文件描述符的函数close(int fd)**.

### 2. FIFO

### 3. XSI IPC

### 4. 共享存储

共享存储允许两个或者多个进程共享一个给定的存储区域。

# 线程

## 一、线程

所有线程共享所在进程的所有信息，包括可执行程序的代码、程序的全局内存和堆内存、栈以及文件描述符。

而线程也有一些数据是各自不同的：线程ID、一组寄存器的值、栈、调度优先级和策略、信号屏蔽字、errno变量、线程私有数据。

### 1. 线程标识

线程也有一个线程ID用来标识不同的线程，其数据结构为**pthread_t**，而且标准中提供了pthread_equal和pthread_self函数分别用来判断线程id是否相等，以及获取调用线程的标识符：

```c
int pthread_equal(pthread_t tid1, pthread_t tid2);
pthread_t pthread_self(void);
```

### 2. 线程创建

线程的创建函数原型如下：

```c
int pthread_create(pthread_t *restrict tidp, const pthread_attr_t *restrict attr, void *(*strt_rtn)(void *), void *restrict arg);   // 成功返回0，错误则返回错误码
```

其中tidp会保存系统设置的线程id；attr为线程属性参数，其内容在下一节讲解；strt_rtn为一个函数指针，创建的线程就是从该指针位置开始执行，该函数可以接受一个参数，可以将想要传递的参数封装到一个结构体中传递过来；最后一个参数arg就是strt_rtn函数指针指向的函数所接收的参数的值。

**新创建的线程会继承调用pthread_create函数的线程的浮点环境和信号屏蔽字，但是该线程的挂起信号集会被清除。**

> 注意：在新线程中不要通过创建线程函数的调用线程传递的参数tidp变量来访问自己的线程id，因为很可能新创建的线程已经开始运行了，但是ptread_create还没有从调用线程处返回，也就是其参数中的tidp的写入时机可能会晚于新线程的执行；因此新线程一定要通过pthread_self方法获取自己的线程id

### 3. 线程退出

线程退出的方式有三种：

* 线程执行return，返回值为线程的退出码
* 被同一进程的其他线程取消
* 调用pthread_exit函数

```c
void pthread_exit(void *rval_ptr);
```

该函数的参数**用来保存线程的退出状态信息**，对于线程return的情况，其return返回的退出码也是这个意思。

而其他线程可以调用pthread_join函数等待线程退出，并且**可以获取线程退出的状态信息(信息被保存在参数中)：pthread_exit调用时的参数、或者return返回码、被其他线程取消则为PTHREAD_CANCELED**：

```c
int pthread_join(pthread_t thread, void **rval_ptr);
```

调用pthread_join的线程会一直阻塞，直到指定的线程退出。

上面提到过很多次，线程可以被其他线程取消，就是调用的下面的方法：

```c
int pthread_cancel(pthread_ t tid);
```

该函数默认情况下等同于tid标识的线程调用了参数为PTHREAD_CANCELED的pthread_exit函数；但是，这种等同并不是强制性的，线程可以选择忽略取消。

像进程一样，线程在退出时也可以调用一系列**线程清理处理程序**，这些函数由下面两个方式进行注册和去除：

```c
void pthread_cleanup_push(void (*rtn)(void *), void *arg);
void pthread_cleanup_pop(int execute);
```

其中rtn参数是清理程序的地址，arg是该清理程序的参数；调用这些清理程序的时机如下：

* 线程调用pthread_exit时
* 响应pthread_cancel函数请求时
* 用非零execute参数调用pthread_cleanup_pop时

注意，可以看到当线程以return的形式退出时，并不会执行清理程序，这可以作为其与其他退出方式的不同点记忆。

**线程的分离状态的含义为：线程的底层存储资源可以在线程终止时立即被回收**。可以调用pthread_detach函数将线程分离：

```c
int pthread_detach(pthread_t tid)
```



## 二、线程控制

### 1. 线程限制

操作系统对线程会有一些限制，限制参数如下：

* PTHREAD_DESTRUCTOR_ITERATIONS：线程退出时操作系统实现试图**销毁线程特定数据的最大次数**
* PTHREAD_KEYS_MAX:进程可以创建的键的最大数目
* PTHREAD_STACK_MIN：一个线程的栈可用的最小字节数
* PTHREAD_THREADS_MAX：进程可以创建的最大线程数

### 2. 线程属性

关于这些属性相关的函数，都有一些同一的特点：

* 有一个初始化函数，把**属性设置为默认值**
* 每个属性都有一个从属性对象中获取属性值的函数，成功返回0，失败返回错误码；二者这种调用模式是将属性值存储在参数指定的内存位置，并不是作为返回值

线程属性类型为：**pthread_attr_t**，其内部具体有四个属性：

* detachstate：线程的分离状态
* guardsize：线程末尾警戒缓冲区大小
* stackaddr：线程栈的最低地址
* stacksize：线程栈的最小长度

初始化和销毁属性的函数原型：

```c
int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destory(pthread_attr_t *attr)
```

正如前面所说，这两个函数成功则返回0，否则返回错误编号

#### detachstate属性值

detachstate属性值可以将线程表示为**分离状态**，该状态下，**表示系统对线程的终止状态不感兴趣**，线程终止后，系统只是回收其资源而不管其状态如何。

该属性值为：**PTHREAD_CREATE_DETACHED和PTHEAD_CREATE_JOINABLE**二者之一

设置和访问的函数原型为：

```c
int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr, int *detachstate);
int pthread_attr_setdetachstate(pthread_attr_t *attr, int *detachstate);
```

其返回值的意义同上面初始化和销毁属性的意义一样。

> Q：线程的终止状态有什么用？获取之后能做什么？

#### 线程栈属性

上面提到的线程属性中，有两个涉及到线程栈的，一个是stacksize，另一个是stackaddr。

其中的stackaddr规定了线程栈最小栈地址，stacksize规定了线程栈的最小尺寸：

```c
int pthread_attr_getstack(const pthread_attr_t *restrict attr, void **restrict stackaddr, size_t *restrict stacksize);
int pthread_attr_setstack(const pthread_attr_t *restrict attr, void *restrict stackaddr, size_t restrict stacksize);
```

stacksize属性没什么好说的，只需要注意一点，该值不能小于**PTHREAD_STACK_MIN**。而stackaddr属性则可以通过setstack函数来**更改线程栈的位置**，通常在当前内存不够时使用malloc开辟另一块内存，然后将线程栈最小地址设置为malloc返回的地址。

需要注意的是，有些CPU中，栈的增长方向是从大到小，所以**stackaddr可能不是栈开始的地址，有可能还是栈顶的临界位置**。

#### 栈的警戒缓冲区

guardsize属性控制着栈末尾之后用来避免栈溢出的扩展内存的大小，但是如果更改了stackaddr属性值的话，该属性值会无效。当线程栈指针溢出到警戒区域时，应用程序会通过信号收到出错信息，开发人员可以根据该信号来避免栈溢出的危险。

```c
int pthread_attr_getguardsize(const pthread_attr_t *restrict attr, size_t guardsize);
int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
```

### 3. 互斥量属性

就像线程具有属性一样，线程的同步对象也有属性。而其中的互斥量属性结构为：**pthread_mutexattr_t**。

其规律同线程属性一样：

```c
int pthread_mutexattr_init(pthread_mutexattr_t *attr);
int pthread_mutexattr_destory(pthread_mutexattr_t *attr);
```

而互斥量属性也有更加细分的属性值。

#### 进程共享属性值

在大多数情况下，由于进程的线程之间可以共享进程的资源和数据，因此很多需要涉及到同步问题，因此就养成了提到线程就认为是在同一个进程下。

**而系统允许不同的进程将同一个内存数据块映射到各自独立的地址空间中去，这样一来，来自不同进程的线程在访问各自映射的数据时也要进行同步控制**。此时，如果**进程共享互斥量属性设置为PTHREAD_PROCESS_SHARED**，从多个进程批次之间共享的内存数据块中分配的互斥量就可以用于这些进程的同步。可以说，该属性值是用在进程之间的互斥量同步。

而将其设置为**PTHREAD_PROCESS_PRIVATE**则实现的是进程内个线程之间的互斥量，这样相较于进程之间的互斥量实现可以节省开销。

```c
int pthread_mutexattr_getshared(const pthread_mutexattr_t *restrict attr);
int pthread_mutexattr_setshared(pthread_mutexattr_t *attr, int pshread);
```

#### 健壮属性

#### 类型互斥量属性

该属性值控制着互斥量的锁定特性，有四种值：

* PTHREAD_MUTEX_NORMAL：不做任何特殊的错误检查或死锁检测
* PTHREAD_MUTEX_ERRORCHECK：提供错误检查
* PTHREAD_MUTEX_RECURIVE：**拥有互斥量的线程可以多次加锁**，只有解锁次数等于加锁次数时，才会释放互斥量
* PTHREAD_MUTEX_DEFAULT：默认类型，不同的操作系统在实现时可以将互斥量类型设置为上面三种中的一种

| 互斥量类型 | 没有解锁时加锁 | 不占用时解锁 | 在已解锁时解锁 |
| :--------: | :------------: | :----------: | :------------: |
|   NOTMAL   |      死锁      |    未定义    |     未定义     |
| ERRORCHECK |    返回错误    |   返回错误   |    返回错误    |
| RECURSIVE  |      允许      |   返回错误   |    返回错误    |

相关函数原型为：

```c
int pthread_mutexattr_gettype(const pthread_mutexattr_t *restrict attr, int *restrict type);
int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
```

### 4. 读写锁属性

读写锁与互斥量类似，也有属性：

```c
int pthread_rwlockattr_init(pthread_rwlockattr_t *attr);
int pthread_rwlockattr_destory(pthread_rwlockattr_t *attr);
```

读写锁只支持进程共享属性，其意义同互斥量中的进程共享属性值一致：

```c
int pthread_rwlockattr_getshared(const pthread_rwlockattr_t *restrict attr);
int pthread_rwlockattr_setshared(pthread_rwlockattr_t *attr, int pshread);
```

### 5. 条件变量属性

条件变量支持**进程间共享**与**时钟**两种属性值，其中进程间共享就不说了，同上面的一样，而时钟属性控制计算pthread_cond_timedwait函数的超时参数时采用的是哪个时钟：

```c
int pthread_condattr_getclock(const pthread_condattr_t *restrict attr, clockid_t *restrict clock_id);
int pthread_condattr_setclock(pthread_condattr_t *attr, clockid_t *clock_id);
```

### 6. 屏障属性

屏障属性只支持**进程共享属性**。









