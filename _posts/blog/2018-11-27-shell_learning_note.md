---
layout: post
title: shell 学习笔记
category: 技术
tags: 
    - ubuntu
    - shell
keywords: 
description: 
---

# shell学习笔记

## 1. 一个命令的结果填充到另一个命令中

ssh例子:

```
# 获取远程服务器的 ip, 并 ssh连接到该服务器上
ssh foo@$(cat /data/ip.result)
```

docker例子:

```
# 删除所有仓库名为 redis 的镜像：
docker image rm $(docker image ls -q redis)
```

*需要注意的是, 单引号与双引号的效果不一样, 需要使用双引号*

举例说明如下, 

`py_cmd.py`模拟一个简单的 echo 命令:

```

import argparse


def echo_name(argv=None):
    parser = argparse.ArgumentParser(description='demo')

    # name
    parser.add_argument('--name', type=str, help='name')
    args = parser.parse_args(args=argv)
    print("name_{}".format(args.name))


if __name__ == '__main__':
    echo_name()

```

示例命令:

```
name="M 1 M"

c1=$(python py_cmd.py --name='${name}' )
echo "case1 ${c1}"  # 输出: case1 name_${name} 

c2=$(python py_cmd.py --name="${name}" )
echo "case2 ${c2}"  # 输出: case2 name_M 1 M

```

## 2. sudo执行echo命令

```
sudo sh -c "echo '{
  \"registry-mirrors\": [\"https://registry.docker-cn.com\"]
}' >> /etc/docker/daemon.json"
```

## 3. 清空文件

```echo -n > ~/xx.conf```

## 4. 常用命令

```
# 查看磁盘使用
df -lh

# 查看当前目录所占空间
du -sh ./

```

## 5. ssh命令

```
# 上传文件
scp ./local.file ubuntu@host:/remote/remote.file

# 下载文件
scp ubuntu@host:/remote/remote.file ./local.file
```

## 6. 获取本机ip
获取本机ip:

```
 ifconfig|sed -n '/inet addr/s/^[^:]*:\([0-9.]\{7,15\}\) .*/\1/p'
```

获取当前虚拟机ip:

```
 ifconfig|sed -n '/inet addr/s/^[^:]*:\([0-9.]\{7,15\}\) .*/\1/p' | grep 192.168
```

## 7. 设置屏幕亮度为0

```
[[ "$(cat /sys/class/backlight/intel_backlight/brightness)" -ne "0" ]] && (echo 0 | sudo tee /sys/class/backlight/intel_backlight/brightness)
```

## 8. 复杂命令示例

来源[docker配置](https://github.com/frkhit/docker-python/blob/py36-selenium-chrome/Dockerfile)

```
ARG CHROME_VERSION="google-chrome-stable"
ARG CHROME_DRIVER_VERSION
RUN apt-get update -qqy \

  # install chrome
  && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
  && apt-get update -qqy \
  && apt-get -qqy install ${CHROME_VERSION:-google-chrome-stable} \
  && rm /etc/apt/sources.list.d/google-chrome.list \

  # install chrome drive
  && if [ -z "$CHROME_DRIVER_VERSION" ]; \
  then CHROME_MAJOR_VERSION=$(google-chrome --version | sed -E "s/.* ([0-9]+)(\.[0-9]+){3}.*/\1/") \
    && CHROME_DRIVER_VERSION=$(wget --no-verbose -O - "https://chromedriver.storage.googleapis.com/LATEST_RELEASE_${CHROME_MAJOR_VERSION}"); \
  fi \
  && echo "Using chromedriver version: "$CHROME_DRIVER_VERSION \
  && wget --no-verbose -O /tmp/chromedriver_linux64.zip https://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip \
  && rm -rf /opt/selenium/chromedriver \
  && unzip /tmp/chromedriver_linux64.zip -d /opt/selenium \
  && rm /tmp/chromedriver_linux64.zip \
  && mv /opt/selenium/chromedriver /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION \
  && chmod 755 /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION \
  && ln -fs /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION /usr/bin/chromedriver \

  # install fonts
  && apt-get -qqy install ttf-wqy-zenhei ttf-wqy-microhei fonts-droid-fallback fonts-arphic-ukai fonts-arphic-uming \
  
  # install selenium
  && pip install --upgrade selenium \
  && rm -rf /root/.cache/pip/* \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*
```

## 9. 环境变量动态设置

示例

```
LDFLAGS=-L/usr/local/opt/openssl/lib https_proxy="" http_proxy="" pip install mysqlclient
```

## 10. 安装 BBR

```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && sudo chmod +x bbr.sh && sudo ./bbr.sh
```

## 11. tar 分卷压缩及解压

```
dd if=/dev/zero of=test.log bs=1M count=1000

# 压缩
tar zcf - test.log |split -b 100m - test.tar.gz.

# 解压
mkdir -p test_tmp
mv test.tar.gz.* test_tmp/
cd test_tmp/
cat logs.test.tar.gz.* | tar zx

```

## 12. debian系 系统安装 tzdata 免输入时区

```
DEBIAN_FRONTEND=noninteractive apt-get install -y tzdata
```

## 13. 复杂条件的文件复制操作

```
mkdir -p /opt/python_libs 

# copy folder
cp -r ~/code/my_libs /opt/python_libs/

# copy folder if exists
[ -d "/opt/code/new_libs" ]  && cp -r /opt/code/new_libs /opt/python_libs/

```

## 14. 文件下载

```
# curl
curl -o out.file -sfL http:xxx.com

# wget
wget -qO out.file http:xxx.com

```

## 15. 关闭进程

```
for pid in `ps -ef | grep python3 | grep "server.py" | grep -v grep | awk '{print $2;}'`
do
    kill -9 $pid
done
```

## 16. grep + awk + xargs

在实践中， 需要批量处理数据文件。 处理完成后， 将结果写入命名方式为 `原文件名 + .upload` 的结果文件中。

操作日志记录在 `simple.log` 中， 其日志内容格式如下：

```
2020-03-21 10:00:00 output to file: /tmp/abc_1.log.upload
2020-03-21 10:00:00 xxxx
2020-03-21 10:10:00 output to file: /tmp/abc_2.log.upload
2020-03-21 10:10:00 xxx1xxx

2020-03-21 10:20:00 output to file: /tmp/abc_4.log.upload
2020-03-21 10:20:00 ....

```

现在需要删除已经处理的原文件。

可以使用 `grep + awk + xargs` 实现：

```
for filename in $(cat simple.log | grep "output to file" | awk '{print $NF}' | awk -F "/" '{print $NF}' ); do
    tmp_file="/tmp/${filename%%.upload}"
    if [ -f "$tmp_file" ]; then
      echo "$tmp_file can remove"
      rm "$tmp_file"
    fi
done

```

## 17. 删除修改日期为一天以前的日志文件

```
#!/bin/bash

while IFS= read -r -d '' file
do
  if [[ $file =~ "tmp_" ]]; then
      let count++
      echo "rm file $file"
      rm "$file"
  fi

done <   <(find /tmp/ -maxdepth 1 -mtime +1 -print0)

echo "remove $count files from /tmp/ "
```

## 18. 删除以日期命名的文件目录

临时日志目录， 存在以下子目录：

```
/var/log/result_20200102
/var/log/result_20200103
/var/log/result_20200104
....
/var/log/result_20200324

```

现在需要删除昨天以前的子目录， 具体实现如下

```
#!/bin/bash

# 删除临时文件
today_str=$(date +%Y%m%d)
yesterday_str=$(date -d -1day +%Y%m%d)
for filename in /var/log/result_2020*; do
    if [[ $filename =~ $today_str ]]; then
      echo "$filename today"
    else
      if [[ $filename =~ $yesterday_str ]]; then
        echo "$filename yesterday"
      else
        echo "$filename remove"
        rm -r "$filename"
      fi
    fi
done

```

## 19. 文本替换

linux 版本:

``` 
# 找到所有 py 文件, 将 
# print("data is {}".format(data))  
# 替换为 
# print("数据 是 {}".format(data))

find . -name "*.py" -exec sed -i s/print\(\"data\ is\ \{/print\(\"数据\ 是\ \{/g {} +

```

mac 略有不同:
``` 
# 找到所有 py 文件, 将 
# print("data is {}".format(data))  
# 替换为 
# print("数据 是 {}".format(data))

find . -name "*.py" -exec sed -i '' s/print\(\"data\ is\ \{/print\(\"数据\ 是\ \{/g {} +

```

## 20. pushd 和 popd

``` 
cd ~

# cd /tmp and do something
pushd /tmp/
    echo "abc" > abc.txt
    tar -czvf abc.txt.tar.gz abc.txt
popd

# backup to ~
ls ./

```

## 21. 获取文件 basename

``` 
FILE="/home/vivek/lighttpd.tar.gz"
basename "$FILE"  # 输出: lighttpd.tar.gz
f="$(basename -- $FILE)"
echo "$f"  # 输出: lighttpd.tar.gz
```

## 22. 多行文本

``` 
config_file="config"

# 代码行
cat > ${config_file} <<EOF
a=b
b=c
x=y
EOF
```

## 23. linux挂载swap文件

```
# 挂载 swap

sudo dd if=/dev/zero of=/swapfile bs=1M count=8192 && \
sudo mkswap /swapfile && \
sudo chmod 600 /swapfile && \
sudo swapon /swapfile && \
sudo sh -c "echo '/swapfile swap swap defaults 0 0' >> /etc/fstab " && \
sudo sh -c "echo 10 >/proc/sys/vm/swappiness"

# 停用 swap
sudo swapoff /swapfile && \
sudo rm /swapfile

```

## 24. grep执行二进制文件过滤

日志文件 `file_with_bin.log` 包含二进制数据.
 
直接执行 grep 过滤 `cat file_with_bin.log | grep "2020-07-18 01"` 时, 报错 `Binary file (standard input) matches`.

解决方法是, 使用 `-a` 参数:

```
cat file_with_bin.log | grep -a -20 "2020-07-18 01"
```

## 25. crontab/ssh 启动带gui 程序

参考: [使用crontab执行GUI程序](https://blog.csdn.net/tracker_w/article/details/46573265).

例如, 脚本 '/start_selenium.sh' 会以 `headless=False` 的方式, 启动 selenium, 在 ubuntu 桌面上启动 chrome 浏览器执行任务.

在自己主机中, 使用 ssh 远程到上述脚本所在的 ubuntu 主机, 可以使用命令启动脚本:

` export DISPLAY=:0 && ./start_selenium.sh ` 

## 26. 默认输入 yes

```

wget http://gosspublic.alicdn.com/ossfs/ossfs_1.80.6_ubuntu18.04_amd64.deb && (yes | sudo gdebi ossfs_1.80.6_ubuntu18.04_amd64.deb )

```


## 27. crontab 查看错误日志

正常条件下, crontab 执行的脚本如果出错, 在 /var/log/syslog 中 看不到有用的信息. 这时, 需要借助 `postfix` 接收crontab 的执行日志.

具体原理, 可参考[迷之 crontab 异常：不运行、不报错、无日志](https://my.oschina.net/leejun2005/blog/1788342).

简要流程:

``` 
# step 1 安装 postfix, 并选择 local only
sudo apt install -y postfix

# step 2 启动 postfix
sudo service postfix start

# step 3 读取 crontab 的执行日志:
tail -f /usr/mail/root

```

## 28. 获取主机所有 ip

```shell script

all_ips=( "localhost" "127.0.0.1" "::1" "0.0.0.0" )
echo ${all_ips[@]}
for ip in $(ifconfig | grep inet | awk '{print $2}' | grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b")
do
    echo ${ip}
    all_ips+=(${ip})
done
echo ${all_ips}

sorted_unique_ips=($(echo "${all_ips[@]}" | tr ' ' '\n' | sort -u | tr '\n' ' '))
echo ${sorted_unique_ips[@]}

```
