# fuse-crash-recovery

fuse-crash-recovery项目主要是构建一个基于fuse的用户态文件系统的crash自动恢复框架。

##原理

本项目基于libfuse构建。整个方案实现包含两部分，一部分在Linux内核的fuse部分，主要实现在crash恢复阶段
将inflighting的IO请求重新放回fuse的Pending队列中，以待恢复后的用户态fuse文件系统服务重新获取此IO请求；
另一部分基于libfuse构建（是否使用libfuse并没有强依赖)，在libfuse的passthrough_ll样例中展示了用户态的
实现方式。
##进展

目前前置要求用户态fuse服务预先创建文件。在文件的读写操作时出现fuse服务进程crash，可以实现用户无感知
的自动恢复。即目前的实现支持只读fuse文件系统的恢复，但是同时还支持对文件的写操作。
下一步计划支持文件的创建/删除等更多通用操作。
##用户态测试程序

```
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/file.h>
#include <sys/types.h>
#include <sys/stat.h> 
#include <string.h>

// gcc -o testfile testfile.c -D_GNU_SOURCE
// ./example/passthrough_ll -o debug -s  /mnt3
// ./testfile
// [root@VM-1-8-centos libfuse]# ps aux | grep pass
// root       34889  0.0  0.0   8848   864 pts/2    S+   13:10   0:00 ./example/passthrough_ll -o debug -s /mnt3
// root       34896  0.0  0.0   9880   128 pts/2    S+   13:10   0:00 ./example/passthrough_ll -o debug -s /mnt3
// root       34913  0.0  0.0  12112  1060 pts/1    S+   13:10   0:00 grep --color=auto pass
// kill child process
// [root@VM-1-8-centos libfuse]# kill 34896 
void main(void)
{
       int fd, size = 0, ret=0;
       unsigned char *buf;
        posix_memalign((void **)&buf, 512, 4096);
       fd = open("/mnt3/test1.file", O_DIRECT,0755);
       while ( (ret = read(fd,buf,4096)) > 0) {
               //sleep(1);
               size += ret;
               printf(" read +%d, t=%d \n", ret, size);

       }
}
```
