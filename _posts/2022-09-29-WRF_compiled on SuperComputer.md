---
layout:     post
title:      WRF在超算上编译
subtitle:   
date:       2022-09-02
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - WRF
    - LINUX
---


---

（1）加载编译器
module load openmpi/2.1.5
（2）下载依赖包存入LIBRARIES中

```bash
cd Build_WRF
mkdir LIBRARIES
cd LIBRARIES
```
需下载mpich、netcdf、Jasper、libpng、zlib共5项
```bash
wget https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/mpich-3.0.4.tar.gz
wget https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/netcdf-4.1.3.tar.gz
wget https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/jasper-1.900.1.tar.gz
wget https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/libpng-1.2.50.tar.gz
wget https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compile_tutorial/tar_files/zlib-1.2.7.tar.gz
```

（3）设置WRF/WPS安装环境
```bash
vim ~/.bash_profile
```

把下面的内容添加进去 （I esc :wq）
```bash
export DIR=/project/udhan/Build_WRF/LIBRARIES
export CC=mpicc
export CXX=mpic++
export FC=mpif90
export F77=mpif90
export F90=mpif90
export JASPERLIB=$DIR/grib2/lib
export JASPERINC=$DIR/grib2/lib
export LDFLAGS=-L$DIR/grib2/lib
export CPPFLAGS=-I$DIR/grib2/include
```
刷新
```bash
source ~/.bash_profile
```
（4）依顺序进行解压安装netCDF等依赖包
```bash
source ~/.bashrc
tar xzvf netcdf-4.1.3.tar.gz
cd netcdf-4.1.3
./configure --prefix=/home/udhan/project/wrf/netcdf --enable-fortran --disable-static --disable-netcdf-4 --enable-shared --with-pic --enable-parallel-tests -enable-pnetcdf --enable-large-file-tests --enable-largefile
make
make install

vim ~/.bash_profile
export PATH=$DIR/netcdf/bin:$PATH
export NETCDF=/home/udhan/project/wrf/netcdf
source ~/.bash_profile

cd ..
tar xzvf mpich-3.0.4.tar.gz
cd mpich-3.0.4
./configure --prefix=$DIR/mpich
make
make install
vim ~/.bash_profile
export PATH=$DIR/mpich/bin:$PATH
source ~/.bashrc
cd ..
tar xzvf zlib-1.2.7.tar.gz
cd zlib-1.2.7
./configure --prefix=$DIR/grib2
make
make install
cd ..
tar xzvf libpng-1.2.50.tar.gz
cd libpng-1.2.50
./configure --prefix=$DIR/grib2
make
make install
cd ..
tar xzvf jasper-1.900.1.tar.gz
cd jasper-1.900.1
./configure --prefix=$DIR/grib2
make
make install
```
若运行./configure出现以下问题时：
```bash
configure: error: F90 and F90FLAGS are replaced by FC and FCFLAGS respectively in this configure, please unset F90/F90FLAGS and set FC/FCFLAGS instead and rerun configure again.
```
可以在命令输入(适用于bash)
```bash
unset F90
unset F90FLAGS
```

（5） 编译WRF
```bash
cd WRF
./configure
```

serial 表示串行计算；
smpar 表示内存共享并行计算(shared memory option)，即使用openMP，大部分多核电脑都支持这项功能；
dmpar 表示分布式并行计算(distributed memory option)，即使用MPI 进行并行计算，主要用在计算集群

选择34


https://www.jianshu.com/p/9107d56df2dd
