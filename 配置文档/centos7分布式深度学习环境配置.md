# Centos7:Spark-GPU集群环境搭建

## 准备工作

> 参考文档：[0-虚拟机安装、ssh设置与jdk安装 (yuque.com)](https://www.yuque.com/wutong-ky4id/eq6hh8/owdeuq?#%20%E3%80%8A0-%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AE%89%E8%A3%85%E3%80%81ssh%E8%AE%BE%E7%BD%AE%E4%B8%8Ejdk%E5%AE%89%E8%A3%85%E3%80%8B)，[0-主机名、ssh、防火墙、ntp、源 (yuque.com)](https://www.yuque.com/wutong-ky4id/eq6hh8/eoi14d?#%20%E3%80%8A0-%E4%B8%BB%E6%9C%BA%E5%90%8D%E3%80%81ssh%E3%80%81%E9%98%B2%E7%81%AB%E5%A2%99%E3%80%81ntp%E3%80%81%E6%BA%90%E3%80%8B)

### 安装jdk及环境变量配置

```
rpm -ivh jdk-8u361-linux-x64.rpm

vim /etc/profile

export JAVA_HOME=/usr/java/jdk1.8.0_131
export JRE_HOME=\$JAVA_HOME/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

source /etc/profile
```

### 安装CUDA11.6

> 参考[【centos】安装nvida CUDA平台附带安装cudnn库及TensorRT8_centos安装cudnn_颢师傅的博客-CSDN博客](https://blog.csdn.net/hh1357102/article/details/128399356)

## Pytorch深度学习环境配置

安装miniconda

```
bash Miniconda3-latest-Linux-x86_64.sh
conda config --set auto_activate_base false
```

卸载miniconda

```
sudo rm -rf path/miniconda3
```

Pytorch国内镜像源安装

```
conda install pytorch==1.12.0 torchvision==0.13.0 torchaudio==0.12.0 cudatoolkit=11.3 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/linux-64/

# 方法二：
# add `--force-reinstall` if you installed the cpu version before.
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu117
```

分布式地质遥感解译常用库安装

```
pip install -U "ray[default]"
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple scipy
matplotlib
opencv-python
scikit-learn
pandas
pyarrow
scikit-image

#GDAL
conda uninstall poppler
conda install poppler
conda install -c conda-forge gdal
```

### Horovod安装

安装pyspark，`conda install pyspark==3.2.0`**（此处安装的版本需与集群的Spark版本一致！）**

安装gcc，参考[为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本 - VPS侦探 (vpser.net)](https://www.vpser.net/manage/centos-6-upgrade-gcc.html)

```
yum -y install centos-release-scl
yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
#临时
scl enable devtoolset-7 bash
#永久
echo "source /opt/rh/devtoolset-7/enable" >>/etc/profile
```

安装cmake

```
# 先下载openssl
yum -y install ncurses-devel
yum install openssl-devel

tar -zxvf cmake-3.24.3.tar.gz
cd cmake-3.24.3
./bootstrap
gmake
gmake install

# 验证
cmake --version
which cmake
```

安装nccl，注意与cuda版本对应

```
sudo yum-config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-rhel7.repo
sudo yum install libnccl-2.12.12-1+cuda11.6 libnccl-devel-2.12.12-1+cuda11.6 libnccl-static-2.12.12-1+cuda11.6
```

安装openmpi，参考[CentOS 7安装OpenMPI_centos安装openmpi_RealMoYe的博客-CSDN博客](https://blog.csdn.net/baidu_26646129/article/details/88425619)

```
gunzip -c openmpi-4.1.5.tar.gz | tar xf -
cd openmpi-4.1.5
./configure --prefix=/usr/local
make all install

#配置环境变量
export PATH=/usr/local/lib:$PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

# 验证
where is openmpi
cd ~/openmpi-4.1.5/examples
make
./hello_c
```

安装horovod

```
HOROVOD_NCCL_INCLUDE=/usr/include HOROVOD_NCCL_LIB=/usr/lib64 HOROVOD_GPU_OPERATIONS=NCCL pip install --no-cache-dir horovod
```

## Spark集群环境搭建

pyspark指定python解释器，参考[(30条消息) PySpark集群完全分布式搭建_pyspark hdfs_Ahaxian的博客-CSDN博客](https://blog.csdn.net/weixin_46031805/article/details/127178998)

Spark配置GPU感知，参考[(30条消息) 关于Spark-Rapids的GPU化迁移至数据仓库的改造_spark rapids_MMJ_64的博客-CSDN博客](https://blog.csdn.net/qq_33550858/article/details/128422185)，`sudo chmod 777 /opt/sparkRapidsPlugin/*`

## 问题修复

docker远程桌面

```
systemctl start firewalld
firewall-cmd --list-all

docker run -it -p 10000:80 -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc:latest

ystemctl stop firewalld.service
firewall-cmd --state
```

reboot后nvidia-smi不可用，先卸载之前安装的gcc，重新安装nvidia驱动[(30条消息) Centos系统出现: NVIDIA-SMI has failed because it couldn’t communicate with the NVIDIA driver_言连的博客-CSDN博客](https://blog.csdn.net/qq_44810242/article/details/127393574)

yum出错，参考[rpm version `XZ_5.1.2alpha‘ not found_rpm: /share/nas6/zhouxy/library/xz/current/lib/lib_AI视觉网奇的博客-CSDN博客](https://blog.csdn.net/jacke121/article/details/109039005)，[pycurl.so: undefined symbol 解决方法 - 简书 (jianshu.com)](https://www.jianshu.com/p/0785cb525547)

Centos7安装桌面环境，参考[CentOS安装桌面环境_一只有梦的网虫的博客-CSDN博客](https://blog.csdn.net/qq_33865313/article/details/117747734?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167948904216800186585778%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=167948904216800186585778&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~pc_rank_34-1-117747734-null-null.142%5Ev76%5Epc_new_rank,201%5Ev4%5Eadd_ask,239%5Ev2%5Einsert_chatgpt&utm_term=centos%E6%A1%8C%E9%9D%A2%E6%8C%89%E6%89%BE&spm=1018.2226.3001.4187)，[CentOS 7 配置 VNC 远程桌面连接_centos7远程桌面_方先森有点懒的博客-CSDN博客](https://blog.csdn.net/hffwj/article/details/124450231?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167654705016800182115655%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=167654705016800182115655&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-124450231-null-null.142%5Ev73%5Econtrol,201%5Ev4%5Eadd_ask,239%5Ev2%5Einsert_chatgpt&utm_term=vnc&spm=1018.2226.3001.4187)
现在用这篇：[centos安装图形化界面及安装或卸载VNC服务--人类高质量安装系列文章_linux卸载vncserver_透透明明的博客-CSDN博客](https://blog.csdn.net/weixin_45208444/article/details/119892364?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169070138116800215075826%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169070138116800215075826&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-119892364-null-null.142^v91^control_2,239^v12^insert_chatgpt&utm_term=vnc%E5%8D%B8%E8%BD%BD&spm=1018.2226.3001.4187)