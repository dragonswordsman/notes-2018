### 为什么要有后端

接下来的研究工作需要通过gem5 dump出一些类似于trace的东西。
计算了一下，最快每分钟会产生几十MB的trace。
由于服务器的磁盘不是SSD，如果多个gem5的进程同时写磁盘可能会让程序变得I/O密集。
此外，以后的工作可能产生更多的trace，可能需要压缩trace，
这不应该在gem5模拟的主循环中进行，
根本的解决方法就是提供一个性能不错的后端。


### 选择什么通信方式和后端？

性能最高的IPC方式应该是共享内存，但是写起来比较麻烦；
目前的吞吐量要求不算高，因此先将就用socket吧。
为了实现快，后端先用python吧。
文件格式协议用protobuf，找了一个socket传递protobuf的
[简单例子](https://github.com/shinezyy/proto-buf-socket)
