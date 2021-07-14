---
layout: post
title: update gcc
date: 2019-06-19 11:21:52
tags:
    - [ gcc ]
categories:
    - [ services ]
---
## gcc update
##### 在centos 6.8 上安装swoole-4.2.13时，make编译时提示gcc版本大于等于4.8版本
```
gcc4.4.7升级gcc4.8
1.下载源码包
wget http://ftp.gnu.org/gnu/gcc/gcc-4.8.0/gcc-4.8.0.tar.bz2
tar -jxvf  gcc-4.8.0.tar.bz2

2.下载编译所需依赖库
cd gcc-4.8.0
./contrib/download_prerequisites
cd ..

3.建立编译输出目录
mkdir gcc-build-4.8.0

4.进入此目录，执行以下命令，生成makefile文件
cd  gcc-build-4.8.0
../gcc-4.8.0/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib

5.编译
#j 后面的是核心数，编译速度会比较快
make -j 4 #编译时间有点长，不到半个小时

6.安装
sudo make install

7、替换当前gcc4.7版本
update-alternatives --install /usr/bin/gcc gcc /usr/local/bin/x86_64-pc-linux-gnu-gcc 40
#倒数第三个是名字，倒数第二个参数为新GCC路径，最后一个参数40为优先级
mv /usr/bin/gcc /usr/bin/gcc.bak // 将原本的gcc重命名(删除亦可)
ln -s /usr/local/bin/x86_64-pc-linux-gnu-gcc /usr/bin/gcc
#使用gcc4.8版本
参考地址:https://blog.csdn.net/qq_24849765/article/details/75893393

报错信息:解决类似 /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21’ not found 的问题
参考链接:https://blog.csdn.net/q936889811/article/details/79947796
找到安装gcc安装位置:
find 安装目录 -name "libstdc++.so*"
列如:
cp /kuaibao/software/gcc-build-4.8.0/prev-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.18 /usr/lib64/

mv libstdc++.so.6 libstdc++.so.6.old
ln -s libstdc++.so.6.0.18 libstdc++.so.6
strings /usr/lib64/libstdc++.so.6 | grep GLIBC
```