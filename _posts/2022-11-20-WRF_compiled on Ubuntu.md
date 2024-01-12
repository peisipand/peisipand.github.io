---
layout:     post
title:      WRF在Ubuntu 22.01.1上编译
subtitle:   
date:       2022-11-20
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - WRF
    - LINUX
---



所用的电脑和系统版本

ThinkStation P350 +  Ubuntu 22.04.1 LTS + gcc version 11.3.0 (系统默认的gcc)



之前的一篇博客记录了我在超算上安装WRF的步骤，但是由于超算老是需要排队，于是我便尝试在Ubuntu上安装WRF。

[这个链接](https://mp.weixin.qq.com/s/nLP1kvBlUGjr4ncRcDq-7Q)给出了一键式安装netcdf的教程，但其中遇到了一些错误，具体记录如下。

# 一、安装依赖

安装顺序：先安装zlib, libpng, JasPer，再安装hdf5(需要依赖zlib)，最后安装netcdf(需要依赖hdf5)。

新建文件夹用来存放下载好的依赖包，该位置可以自行修改

```bash
mkdir /home/pzp/Software/WRF_LIBRARY_FILE
```

新建文件夹用来存放下载好的依赖包，该位置可以自行修改

```bash
mkdir /home/pzp/Software/WRF_LIBRARY_FILE
```



配置环境变量(与修改.bashrc的[区别](https://www.jianshu.com/p/11d71bacef1a))

```bash
sudo gedit ~/.bash_profile
```

将以下内容添加进去并保存

```
export DIR=/path_to_directory/Build_WRF/LIBRARIES
export JASPERLIB=$DIR/grib2/lib
export JASPERINC=$DIR/grib2/include
export LDFLAGS=-L$DIR/grib2/lib
export CPPFLAGS=-I$DIR/grib2/include
export PATH=$DIR/netcdf/bin:$PATH
export NETCDF=$DIR/netcdf
```

刷新环境变量

```bash
source ~/.bash_profile
```



这里先给出我修改过后的代码。将以下代码保存为netcdf_install.sh

```bash
#### COPY TO install.sh ###
#!/bin/bash

# VERY BASIC installation script of required libraries
# for installing these packages:
#   zlib-1.2.10
#   libpng-1.2.50
#   jasper-1.900.1
#   netcdf-4.1.3


# If you have downloaded other versions edit these version strings
z_v=1.2.10
l_v=1.2.50
j_v=1.900.1
h_v=1.10.1
nc_v=4.1.3


# Install path, change accordingly
# You can change this variable to control the installation path
# If you want the installation path to be a "packages" folder in
# your home directory, change to this:



#################
# Install z-lib #
#################
tar xzvf zlib-${z_v}.tar.gz
cd zlib-${z_v}
make clean
./configure --prefix=$DIR/grib2
make
make install
cd ../
#rm -rf zlib-${z_v}
echo "Completed installing zlib"

#################
# Install libpng #
#################
tar xzvf libpng-${l_v}.tar.gz     #or just .tar if no .gz present
cd libpng-${l_v}
make clean
./configure --prefix=$DIR/grib2
make
make install
cd ..
echo "Completed installing libpng"


#################
# Install Jasper #
#################
tar xzvf jasper-${j_v}.tar.gz     #or just .tar if no .gz present
cd jasper-${j_v}
make clean
./configure --prefix=$DIR/grib2
make
make install
cd ..
echo "Completed installing Jasper"


#################
# Install netcdf #
#################
tar xzvf netcdf-${nc_v}.tar.gz     #or just .tar if no .gz present
cd netcdf-${nc_v}
make clean
./configure --prefix=$DIR/netcdf --disable-dap --disable-netcdf-4 --disable-shared
make
make install
cd ..




##########################
# Completed installation #
##########################

echo ""
echo ""
echo "##########################"
echo "# Completed installation #"
echo "#   of NetCDF package    #"
echo "#  and its dependencies  #"
echo "##########################"
echo ""
echo ""

```

这些包的下载地址可以参照[wrf的官网指导](https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php)。

安装好netcdf后，查看配置选项 
```
nc-config
```

如果编译没有报错，但是nc-config还是没能正确的输出的话，可能是之前的make没有清理，可以手动 make clean一下。

另外一种情况是 新打开一个终端的时候nc-config就不管用了，这时候需要重新source一下。

# 二、安装WRF

- Once you obtain the WRF source code (and if you downloaded a tar file, have unpacked the tar file), go into the WRF directory:

  > cd WRF

- Create a configuration file for your computer and compiler:

  > ./configure

  You will see various options. Choose the option that lists the compiler you are using and the way you wish to build WRF (i.e., serially or in parallel). Although there are 3 different types of parallel (smpar, dmpar, and dm+sm), we have the most experience with dmpar and typically recommend choosing this option.

- Once your configuration is complete, you should have a

   

  configure.wrf

   

  file, and you are ready to compile. To compile WRF, you will need to decide which type of case you wish to compile. The options are listed below:

  > em_real (3d real case)
  > em_quarter_ss (3d ideal case)
  > em_b_wave (3d ideal case)
  > em_les (3d ideal case)
  > em_heldsuarez (3d ideal case)
  > em_tropical_cyclone (3d ideal case)
  > em_hill2d_x (2d ideal case)
  > em_squall2d_x (2d ideal case)
  > em_squall2d_y (2d ideal case)
  > em_grav2d_x (2d ideal case)
  > em_seabreeze2d_x (2d ideal case)
  > em_scm_xy (1d ideal case)

  > ./compile *case_name* >& log.compile

  where *case_name* is one of the options listed above

  Compilation should take about 20-30 minutes.



安装成功后，log.compile中会出现

`--->                  Executables successfully built                  <---`

此时，test/em_les中也有会生成的ideal.exe和wrf.exe。

# 三、安装WPS

- Go into the WPS directory:

  > cd WPS

- Similar to the WRF model, make sure the WPS directory is clean, by issuing:

  > ./clean

  and then you can configure:

  > ./configure

- You can now compile WPS:

  > ./compile >& log.compile

  Compilation should only take a few minutes.

- If the compilation is successful, there should be 3 executables in the WPS top-level directory, that are linked to their corresponding src/ directories:

```
pzp@pzp-ThinkStation-P350:~/Software/WRF/WPS$ ls *.exe
geogrid.exe  metgrid.exe  ungrib.exe
```

需要注意的是，WPS和WRF默认要放在同一级文件夹，如果不在的话，需要在configure.wps中修改下面这一句。

> WRF_DIR = ../WRF

# 四、安装过程中遇到的问题

##  4.1 安装netcdf的问题

一开始安装了netcdf-c和netcdf-fortran，但是一直报错。之后全部参照官网来安装。

后来又有报错

```
more undefined references to `__module_optional_input_MOD_st_input' follow
collect2: error: ld returned 1 exit status
```

[该博客](https://forum.mmm.ucar.edu/threads/wrf-4-4-build-failure-with-gcc-ubuntu-11-2-0-19ubuntu1.11536/)遇到了同样的问题，并给出了几种解决方案。

一种是：使用 **openmpi** 而不是 mpich即可解决这个问题。

```
sudo apt install libopenmpi-dev libhdf5-openmpi-dev
```

亲测有效。

第二种是建议更换为老版本的gcc，说是gcc 9之后的会有这个问题

同时这个博客中还介绍了 [WRF4.4_Install Script](https://github.com/bakamotokatas/WRF-Install-Script/releases/tag/WRF4.4_Install)，没试过，不过看着应该是没问题的。

##  4.2 安装WPS的问题

第一遍安装WPS时，geogrid.exe和metgrid.exe都可以正常生成，但是ungrib.exe却没有，log文件里也没有报错。经搜索，原来是之前安装的netcdf那个一键脚本中忘记了make clean，这个问题很容易出现。换句话说，不报错不等于没问题，正常的情况下在/home/pzp/Software/Build_WRF/LIBRARIES/grib2目录下，应该有以下这么多文件，如果少，那就最好手动clean一下了。 [参考链接](https://forum.mmm.ucar.edu/threads/ungrib-exe-not-generated-while-compiling-wps.10120/)

```
pzp@pzp-ThinkStation-P350:~/Software/Build_WRF/LIBRARIES/grib2/lib$ ls
libjasper.a   libpng12.so         libpng.la         libz.a          pkgconfig
libjasper.la  libpng12.so.0       libpng.so         libz.so
libpng12.a    libpng12.so.0.50.0  libpng.so.3       libz.so.1
libpng12.la   libpng.a            libpng.so.3.50.0  libz.so.1.2.10
```



经过这次Ubuntu系统上的WRF编译，对这一套流程有了更清晰的认识。

熟能生巧之~

# 五、Ubuntu上运行WRF

之前一直用的WRF3.9版本，这次用的WRF4.4。有些地方还是不太一样的，比如新版本把多个ideal case汇总到了一起，并需要在namelist.input中具体指定。比如我要用的les对应的是9，对应的规则可以参考 README.namelist文件。

```
 &ideal
 ideal_case = 9
 /
```

```
mpirun -np 6 ./ideal.exe
mpirun -np 6 ./wrf.exe
```
至于具体选多少核，一是要看自己的硬件，二要根据自己的案例合理设置核数，不一定给的核数越大就会越快。[官方](https://forum.mmm.ucar.edu/threads/how-many-processors-should-i-use-to-run-wrf.5082/)介绍了分布式计算的原理（通过将大区域切块）并给了一个经验法则。

>For your smallest-sized domain:
>((e_we)/25) * ((e_sn)/25) = most amount of processors you should use
>
>For your largest-sized domain:
>((e_we)/100) * ((e_sn)/100) = least amount of processors you should use


```
查看CPU是几核
#cat /proc/cpuinfo |grep "cores"|uniq
```
