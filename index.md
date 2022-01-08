## CosmoMC安装（2020.3.24）
#### 一. 准备
##### 1. 操作系统
Ubuntu 18.04 LTS
##### 2. Intel Fortran Compliler(ifort)
~~学生可获得免费的ifort，需要带有edu的学校邮箱，验证注册intel后可以通过邮件得到。
教育版网址：[Qualify for Free Software](https://software.intel.com/en-us/qualify-for-free-software/student)~~
（现已不可用，Intel提供了免费的Intel® oneAPI Toolkits，替代了原ifort。只能网上搜寻资源）。
版本要求14+ 。不要使用版本19，运行CosmoMC时会出现问题。
本次使用的是18版本：parallel_studio_xe_2018_update4_cluster_edition
##### 3. Open MPI
网址：[Open MPI Version4.0](https://www.open-mpi.org/software/ompi/v4.0/)
本次使用的是：openmpi-4.0.3
##### 4. CFITSIO
网址：[cfisdio_latest](http://heasarc.gsfc.nasa.gov/FTP/software/fitsio/c/cfitsio_latest.tar.gz)
##### 5. Planck likelihood code and data
网址：[Code](http://pla.esac.esa.int/pla/aio/product-action?COSMOLOGY.FILE_ID=COM_Likelihood_Code-v3.0_R3.01.tar.gz)
网址：[Data](http://pla.esac.esa.int/pla/aio/product-action?COSMOLOGY.FILE_ID=COM_Likelihood_Data-baseline_R3.00.tar.gz)
##### 6. CosmoMC
建议使用git克隆下载。在github上直接下载的zip，解压后安装失败。
参见：[COSMOMC Download](https://cosmologist.info/cosmomc/private_DONTLINK_downs/)

#### 二. 安装
##### 1. 本次使用的Ubuntu系统是最小安装的桌面版，处于初始状态，需要更新并安装一些必要的组件：
```
sudo apt install update && sudo apt install upgrade
sudo apt install gfortran
sudo apt install g++
sudo apt install make
sudo apt install gedit
sudo apt install gnuplot
sudo apt install gnuplot-x11
sudo apt install python-pip
sudo apt install cython
sudo apt install python-numpy
sudo apt install python-matplotlib
sudo apt install python-scipy
sudo apt install liblapack-dev
sudo pip install pyfits
sudo pip install getdist
```
***
##### 2. 安装ifort
本次安装在当前用户下，故不使用sudo（建议sudo安装在默认目录下）。
解压后运行install.sh或install_GUI.sh都可以，后者是图形界面安装。在下载目录运行命令:
```
tar -zxvf parallel_studio_xe_2018_update4_cluster_edition.tgz
cd parallel_studio_xe_2018_update4_cluster_edition
./install_GUI.sh
```
安装过程可以参考 <https://blog.csdn.net/sowhatgavin/article/details/71055032>
安装完成后配置环境变量：
```
gedit ~/.bahsrc
```
在最下方加入：
```
source /yourintelpath/intel/compilers_and_libraries_2018.5.274/linux/bin/ifortvars.sh intel64
```
然后
```
source ~/.bashrc
```
测试：
```
ifort -V
```
显示：
```
Intel(R) Fortran Intel(R) 64 Compiler for applications running on Intel(R) 64, Version 18.0.5.274 Build 20180823
Copyright (C) 1985-2018 Intel Corporation.  All rights reserved.
```
***
##### 3. 安装openmpi
进入下载目录，运行命令：
```
tar -zxvf openmpi-4.0.3.tar.gz
cd openmpi-4.0.3
./configure --prefix=/youropenmpipath/openmpi F77=/yourintelpath/intel/compilers_and_libraries_2018.5.274/linux/bin/intel64/ifort FC=/yourintelpath/intel/compilers_and_libraries_2018.5.274/linux/bin/intel64/ifort F90=/yourintelpath/intel/compilers_and_libraries_2018.5.274/linux/bin/intel64/ifort
make
make install
```
配置环境变量：
```
gedit ~/.bashrc
```
在最下方加入：
```
PATH="/youropenmpipath/openmpi/bin":${PATH}
export PATH
LD_LIBRARY_PATH="/youropenmpipath/openmpi/lib/openmpi":$LD_LIBRARY_PATH
export LD_LIBRARY_PATH
```
然后：
```
source ~/.bashrc
```
测试：
```
mpif90 -V
```
显示：
```
Intel(R) Fortran Intel(R) 64 Compiler for applications running on Intel(R) 64, Version 18.0.5.274 Build 20180823
Copyright (C) 1985-2018 Intel Corporation.  All rights reserved.
```
***
##### 4. 安装cfitsio
进入下载目录，运行命令：
```
tar -zxvf cfitsio_latest.tar.gz
cd cfitsio-3.47
./configure --prefix=/yourcfitsiopath/cfitsio
make
make install
```
配置环境变量：
```
gedit ~/.bashrc
```
在最下方加入：
```
export LD_LIBRARY_PATH=/yourcfitsiopath/cfitsio/lib:${LD_LIBRARY_PATH}
```
然后：
```
source ~/.bashrc
```
***
##### 5. 安装 Planck Likelihood Code
进入下载目录，运行命令：
```
tar -zxvf COM_Likelihood_Code-v3.0_R3.01.tar.gz
```
将解压好的文件移动到想要安装的目录，之后进入最里层文件夹，如：
```
cd /yourplanckpath/planck/code/plc_3.0/plc-3.01
```
修改Makefile文件
```
gedit Makefile
```
检查CFITSOIPATH，修改为：
```
CFITSOIPATH=/yourcfitsiopath/cfitsio
```
检查MKL，修改为:
```
MKLROOT=/yourintelpath/intel/compilers_and_libraries_2018.5.274/linux/mkl
LAPACKLIBPATHMKL=-L$(MKLROOT)/lib/intel64
```
保存并退出。执行命令：
```
./waf configure --install_all_deps --cfitsio_prefix=/yourcfitsiopath/cfitsio
./waf install
```
配置环境变量：
```
gedit ~/.bashrc
```
最下方加入：
```
export PLANCKLIKE=cliklike
export CLIK_PATH=/yourplanckpath/planck/code/plc_3.0/plc-3.01
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CLIK_PATH/lib
```
然后：
```
source ~/.bashrc
```
现在配置Planck Data 。进入下载目录，执行命令：
```
tar -zxvf COM_Likelihood_Data-baseline_R3.00.tar.gz
```
复制其中的数据文件hi_l, low_l, lensing到plc-3.01文件夹中。
***
##### 6. 安装ComoMC
运行命令：
```
git clone --recursive https://github.com/cmbant/CosmoMC.git
```
链接plc-3.01数据：
```
cd CosmoMC/
ln -s /yourplanckpath/planck/code/plc_3.0/plc-3.01 ./data/clik_14.0
```
编译安装：
```
cd source/
make
```
下载的CosmoMC本身包含CAMB程序，可进行编译运行：
```
cd ..
cd camb/fortran
make
```
可运行命令
```
./camb ../inifiles/params.ini
```
进行测试，目录中会出现.dat文件。
测试CosmoMC，进入CosmoMC文件夹，运行命令：
```
mpirun -np 4 ./cosmomc test.ini
```
***至此，CosmoMC安装完成。***


#### 三. 参考资料

[英文guide:1808.05080](https://arxiv.org/abs/1808.05080)

[中文guide:星空下(知乎)](https://zhuanlan.zhihu.com/p/51193997)

## Cobaya安装（2022.1.8）

若要使用Cobya跑MCMC，仅仅执行到安装完成cfitsio即可，之后直接安装[Cobya](https://cobaya.readthedocs.io/en/latest/)，Planck Likelihood Code可使用其
自动安装模块自动安装。
