---
layout: post
title: shell 脚本总结
date: 2019-08-22 10:41:18
tags: 
    - [shell 脚本总结]
categories:
    - [ shell ]
---
#### shell 判断当前用户和传参个数
```
##Judge user root
if [ $UID -ne 0 ];then
    echo "You are not support user,Please call root"
    exit 1;
fi

##Judge arg number
if [ $# -ne 1 ];then
    echo $"Usage: $0 {start|stop}"
    exit 1
fi
```
#### case配置和使用
```
function usage() {
    packge=$1
    case $packge in
    stop)
       echo "Stop Elk Cluster Service"
       elk_stop
       ;;
    start)
       echo "Start Elk Cluster Service"
       elk_start
       ;;
    status)
       echo "Elk Cluster Status"
       elk_status
       ;;
    *)
       echo -e "\033[41;30m Usage: $0 $1 Incorrect parameter \033[0m"
       echo $"Usage: $0 {start|stop|status}"
       exit 0
       ;;
  esac
}

for arg in "$*"
do
        usage $arg
done
```
#### 文件追加
```
echo "124567890" | tee -a /tmp/1.txt  #往/tmp/1.txt

cat >> /tmp/1.txt << EOF  #往/tmp/1.txt
1111111111
EOF

cat >/tmp/1.txt << EOF   #覆盖/tmp/1.txt 原有文件内容
11111
EOF

cat << EOF > /tmp/1.txt  #覆盖/tmp/1.txt 原有文件内容
> 22222222
> EOF

cat <<EOF>>/tmp/1.txt  #往/tmp/1.txt 追加内容 
> 333333
> EOF
```
#### 变量截取和替换
```
${变量#匹配规则}	从头开始匹配，最短删除
${变量##匹配规则}	从头开始匹配，最长删除
${变量%匹配规则}	从尾开始匹配，最短删除
${变量%%匹配规则}	从尾开始匹配，最长删除
${变量/旧字符/新字符}	替换变量内的旧字符为新字符，只替换第一个
${变量//旧字符/新字符}	替换变量内的旧字符为新字符，全部替换

案列:
"i love you, do you love me"
tmp=${var##*ove}
echo $tmp
tmp=${var##*ove}
echo $tmp
tmp=${var%ove*}
echo $tmp
tmp=${var%%ove*}
echo $tmp
tmp=${var/love/LOVE}
echo $tmp
tmp=${var//love/LOVE}
echo $tmp
```
##### 字符串处理
```
计算字符串长度
1、${#string}
2、expr length $string
var="i love you"
echo ${#var}
expr length ${var}

获取字符串中的子串
1、${string:pos}
2、${string:pos:len}
3、${string:(-pos)}
4、expr substr $string $pos $len

```
##### 案列
```

#!/bin/sh
var="i love you"
tmp=${var:1}
echo "$tmp"
tmp=${var:1:2}   # 下标是从0开始的
echo "${tmp}"
tmp=${var:(-5)}
echo "${tmp}"
tmp=`expr substr "${var}" 1 10`  # 注意expr下标是从1开始的
echo "${tmp}"
```
#####  一个完整字符串处理案例
```
#!/usr/bin/env bash
#
string="Bigdata process framework is Hadoop,Hadoop is an open source project"
function tips_info
{
	echo "******************************************"
	echo "***  (1) 打印string长度"
	echo "***  (2) 在整个字符串中删除Hadoop"
	echo "***  (3) 替换第一个Hadoop为Mapreduce"
	echo "***  (4) 替换全部Hadoop为Mapreduce"
	echo "******************************************"
}
function print_len
{
	# -z表示判断字符串长度是否为0
	if [ -z "$string" ];then
		echo "Error,string is null"
		exit 1
	else
		echo "${#string}"
	fi
}
function del_hadoop
{
	if [ -z "$string" ];then
		echo "Error,string is null"
		exit 1
	else
		echo "${string//Hadoop/}"
	fi
}
function rep_hadoop_mapreduce_first
{
	if [ -z "$string" ];then
		echo "Error,string is null"
		exit 1
	else
		echo "${string/Hadoop/Mapreduce}"
	fi
}
function rep_hadoop_mapreduce_all
{
        if [ -z "$string" ];then
                echo "Error,string is null"
                exit 1
        else
                echo "${string//Hadoop/Mapreduce}"
        fi
}
while true
do
echo "【string=\"$string\"】"
tips_info
read -p "Please Switch a Choice: " choice
# case, esac(反写esac); 相当于switch case break
case "$choice" in
	1)
		echo
		echo "Length Of String is: `print_len`"
		echo
		continue
		;;
	2)
		echo
		echo "删除Hadoop后的字符串为：`del_hadoop`"
		echo
		;;
	3)
		echo 
		echo "替换第一个Hadoop的字符串为：`rep_hadoop_mapreduce_first`"
		echo
		;;
	4)
		echo 
                echo "替换第一个Hadoop的字符串为：`rep_hadoop_mapreduce_all`"
                echo
		;;
	q|Q)
		exit 0
		;;
	*)
		echo "error,unlegal input,legal input only in { 1|2|3|4|q|Q }"
		continue	
		;;
esac
done
```
