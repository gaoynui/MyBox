conda & mamba

------

## Conda

1.介绍

Conda 是一个开源的软件包管理系统和环境管理系统，用于安装多个版本的软件包及其依赖关系，并在它们之间轻松切换。 Conda 是为 Python 程序创建的，适用于 Linux，OS X 和Windows，也可以打包和分发其他软件。

conda分为anaconda和miniconda。anaconda是包含一些常用包的版本（这里的常用不代表你常用 微笑.jpg），miniconda则是精简版，需要啥装啥，所以推荐使用miniconda。

2.安装

miniconda官网:[官网](https://conda.io/miniconda.html)

linux安装：

```
添加谷歌通用nameserver:
/etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
下载安装脚本:
wget -c https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
给脚本赋予执行权限:
chmod 777 Miniconda3-latest-Linux-x86_64.sh
运行脚本:
bash Miniconda3-latest-Linux-x86_64.sh

注：
安装过程询问是否将conda加入环境变量选择no
```

启动:

进入miniconda/bin目录下,执行activate命令

当出现(base)说明启动成功

3.添加镜像

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
```

显示已添加的镜像：

```
conda config --get channels
```

4.退出conda

```
conda deactivate
```

5.创建canda环境

之前创建的时候显示的是（base）这是conda的基本环境，有些软件依赖的是python2的版本，当你还是使用你的base的时候你的base里的python会被自动降级，有可能会引发别的软件的报错，所以，可以给一些特别的软件一些特别的关照，比如创建一个单独的环境。

查看当前存在的环境：conda env list

启动环境：conda activate envName

退出环境: conda deactivate

删除环境：conda remove -n envName --all

重命名环境：conda create -n newName --clone lodName

​                      conda remove -n oldName --all

------

## mamba

1.介绍

mamba转为conda加速而生，其改写了conda下载资源的固有方式，以多线程的方式对网络资源进行并行下载从而大幅提升conda效率。

mamba并不是重写conda的所有功能，只是针对conda的低效的功能进行重写，并添加一些conda使用新功能。

2.安装

```
conda install -c conda-forge mamba
```

3.使用

```
下载
mamba install -c conda-forge 依赖 -y
```

```
查看当前库在环境下的所有可用版本
mamba repoquery search
```

```
查看依赖关系
指定库依赖哪些库：
	mamba repoquery depends 库名
指定库被哪些库依赖：
	mamba repoquery whoneeds ipython
```

