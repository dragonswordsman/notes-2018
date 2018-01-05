### Simpoint

[Simpoint 官网](http://cseweb.ucsd.edu/~calder/simpoint/)详细介绍了
simpoint，我这里简单介绍下。

1. simpoint为何诞生？

  体系结构研究有3种方法：
  - 流片
  - 用FPGA模拟
  - 用模拟器模拟

  从上到下，价格越来越低，速度越来越慢。流片一次需要几百上千万；FPGA模拟性价比很不错，但是debug麻烦；因此在架构探索阶段一般都采用模拟器。
  模拟器的运行速度很慢，比如gem5的乱序模型需要30~120min才能模拟两亿条指令。
  Spec 2006中很多benchmark的指令数都接近或者超过了1万亿条，显然无法在可以介绍的时间内运行完。
  为了解决这个问题，体系结构研究者们希望从程序中抽取出能代表程序性质的片段来测试新架构，Simpoint为此而诞生。

- simpoint的输入输出是什么？

  首先需要介绍**基本块向量(BBV)** ，什么是BBV呢？
  一个程序可以切分为N个基本块，BBV的每一维表示的是一段程序进入每个基本块的次数。
  虽然simpoint里面定义的基本块与龙书里面的不完全一致（具体可以参考官网），但是大体上这样理解也没什么问题。
  Simpoint的输入就是一系列BBV组成的矩阵。
  假设一个1000 billion条指令的程序有N个基本块，我们将它切分成十万个长度为10 million的片段，每一个片段都可以生成一个N维的BBV。
  如果要抽取这个程序具有代表性的片段，那么需要送给simpoint的输入就是十万个N维的BBV。
  而simpoint的输出告诉你这十万个片段中，哪些是最具有代表性的。

- simpoint如何找到有代表性的程序段？

  Simpoint用K-means对这些BBV进行聚类，如果收敛了，从每个类中选一个点作为代表，这个点对应的片段的权重与该类的大小有关。如果没收敛呢，好像也没啥好办法，这个我没有深究。我的选择是重设一个随机种子重跑，知道收敛为止。

- 怎么产生simpoint所需要的BBV矩阵？

  Gem5可以输出simpoint所需的BBV。


### 用gem5生成BBV

BBV的好处是微架构无关，同一份二进制、相同的输入在不同的微架构上，用commit的动态指令序列来看一定是相同的，因此BBV也是确定的。因此可以先用功能型模拟的方法得到BBV，这样所需的时间较短。

我要为spec 2006产生BBV，所以在
[simpoint.py](https://github.com/shinezyy/gem5/blob/master/configs/spec2006/simpoint.py)
中创建spec的benchmark对应的进程，然后在
[create_simpoint.sh](https://github.com/shinezyy/gem5/blob/master/util/create_simpoint.sh)
中为gem5指定相关参数。
Simpoint.py如何创建spec 2006的进程在专门的文章中说明。
这里介绍一下create_simpoint.sh中与simpoint相关的参数，这是该脚本的最主要的命令：

```
$GEM5_DIR/build/$arch/gem5.$gem5_ver\
    --outdir=$output_dir\
    $GEM5_DIR/configs/spec2006/$conf\
    --simpoint-profile --simpoint-interval=100000000 --fastmem \
    -I 1000000000000\
    --mem-size=4GB\
    --benchmark="$benchmark"\
    --benchmark_stdout=$output_dir\
    --benchmark_stderr=$output_dir\
    --cpu-type='AtomicSimpleCPU' \
```
- *$GEM5_DIR/configs/spec2006/$conf*：就是simpoint.py
- *--simpoint-profile*：开启simpoint功能
- *--simpoint-interval=100000000*：将每一个simpoint片段设为一亿条指令
- *--fastmem*：最简单、最快的内存模型
- *-I 1000000000000*：最多执行一万亿条指令，否则太耗时
- *--cpu-type='AtomicSimpleCPU'*：gem5中最快的CPU模型

create_simpoint.sh使用示例：
```
$gem5_root/util/create_simpoint.sh -b "GemsFDTD" -o \
 $gem5_take_simponit_dir/GemsFDTD -a ARM -c simpoint.py -v fast
```
- *$gem5_root*：gem5的根目录
- *GemsFDTD*：benchmark名
- *$gem5_take_simponit_dir*：存放各个benchmark的输出的总目录
- *-a ARM -v fast*：使用ARM ISA(包括AARCH64)的gem5.fast执行
