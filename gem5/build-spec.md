## 用aarch64-linux-gnu-gcc-4.8 编译spec 2006

之所以用4.8不用5.4，是因为用5.4 perlbench也报错，换成4.8少点事。
ubuntu16.04安装工具链：
```
sudo apt install gcc-4.8-aarch64-linux-gnu \
 g++-4.8-aarch64-linux-gnu gfortran-4.8-aarch64-linux-gnu
```
随便找一个样例的\*.cfg，拷贝一份：
```
cp xx.cfg arm64.cfg
```
然后在arm64.cfg里面设置CC等，：
```
CC           = aarch64-linux-gnu-gcc-4.8
CXX          = aarch64-linux-gnu-g++-4.8
FC           = aarch64-linux-gnu-gfortran-4.8
```
因为要在gem5中使用se模式运行，所以要静态链接，改一改OPTIMIZE flags：
```
## Base is low opt
default=base=default=default:
COPTIMIZE     = -O2 -static
CXXOPTIMIZE  = -O2 -static
FOPTIMIZE    = -O2 -static
```
主要问题是旧版本的编译器与新版本不兼容，导致编译出错，比如*dealIII*, *wrf*等。
解决方法是在config文件里面加入portability flag：

```
#####################################################################
# Portability Flags - INT
#####################################################################

400.perlbench=default=default=default:
# Pick one of the defines below, or the other
notes35    = 400.perlbench: -DSPEC_CPU_LINUX_X64
CPORTABILITY = -DSPEC_CPU_LINUX_X64

462.libquantum=default=default=default:
notes60= 462.libquantum: -DSPEC_CPU_LINUX
CPORTABILITY=  -DSPEC_CPU_LINUX

#####################################################################
# Portability Flags - FLOAT
#####################################################################

483.xalancbmk=default=default=default:
CXXPORTABILITY= -DSPEC_CPU_LINUX -include cstdlib -include cstring

447.dealII=default:
CXXPORTABILITY = -include cstdlib -include cstring -include cstddef

481.wrf=default=default=default:
wrf_data_header_size = 8
CPORTABILITY = -DSPEC_CPU_CASE_FLAG -DSPEC_CPU_LINUX
```
