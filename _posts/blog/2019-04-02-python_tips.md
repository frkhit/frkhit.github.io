---
layout: post
title: python小技巧
category: 技术
tags: 
    - python
keywords: 
description: 
---

# python小技巧

## 1. 使用指定 pypi源安装包
```
pip install jieba gensim numpy tornado *.whl -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com
pip install -r req.txt -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com
```

## 2. 将指定包安装到lib目录下
```
echo path_to_package >> xx/lib/python2.7/site-packages/pk1.pth
```

## 3. cheat: 命令示例
```
python -m pip install cheat

# tar命令使用示例
cheat tar

# To extract an uncompressed archive:
tar -xvf /path/to/foo.tar

# To create an uncompressed archive:
tar -cvf /path/to/foo.tar /path/to/foo/
....
```

*更新*

如果使用 `cheat` 过程中, 提示 `找不到 cheatsheet`, 换一种方式安装:

``` 
# 卸载 python cheat
python -m pip uninstall -y cheat

# 下载对应版本的 cheat 
# link: https://github.com/cheat/cheat/releases

# 安装
chmod +x cheat-linux-amd64 && sudo mv cheat-linux-amd64 /usr/local/bin/cheat

# 首次运行, 会提示安装 cheatsheet 文件

# 使用
cheat tar

```


## 4. pip 工具

逐行安装, 忽略错误:
```
cat requirements.txt | xargs -n 1 python -m pip install
```

## 5. python 文件服务器
```
# python3 or python2
python -m http.server || python -m SimpleHTTPServer

# bind addr
python -m http.server -b 127.0.0.1 8080

```

## 6. logger无法输出

基本描述：`python`中获取 `logger` 变量，`level`为 `logging.INFO`, 然而使用 `logger.info("xxx")` 无法输出日志。

debug 过程: 发现`logger.handlers`不为空，但是`logger.disabled为True`, 导致没有执行日志输出。检查后，发现初始化时，`logger.disabled`为 `False`；排除其他代码修改`logger.disabled`值。

问题修复：

参考[
Why do all my loggers start auto-disabled?](https://bytes.com/topic/python/answers/832745-why-do-all-my-loggers-start-auto-disabled). Vinay Sajip 提到原因: 初始化`logger`后，代码调用`fileConfig`会修改 `logger.disabled`。

```
Calling fileConfig() disables any loggers existing at the time of the
call. Make sure you call fileConfig() before instantiating any
loggers; after that you can instantiate as many as you like, and they
will all be enabled. Make sure you don't call fileConfig() again, as
in that call all loggers which are not named in the configuration will
be disabled.

The reason why fileConfig() disables existing loggers is that it was
designed as a one-off configuration (which completely replaces any
existing configuration with that specified in the config file) - not
as an incremental configurator.

Best regards,

Vinay Sajip
```
因此，解决方法是，先调用`fileConfig`再初始化自己的`logger`。

## 7. 库打包

打包成 whl 文件格式

```
python setup.py bdist_wheel
```

安装所有 whl 库

```
python -m pip install requests *.whl
```

## 8. 输出缓存问题

场景 1: 

使用`nohup python pytorch_demo.py > all.log 2>&1 &` 无法在 all.log 文件中获取到日志, 

但单独执行`python pytorch_demo.py ` 能看到输出结果.

解决方案是使用 `-u`不缓存输出, 

`nohup python -u pytorch_demo.py > all.log 2>&1 &`

场景 2:

docker 中 直接运行 python 服务, 通过 `docker log -f <container>` 看不到日志,

解决方法是, 启动 python 服务时, 增加 `-u` 参数.


进阶:

可以使用 `PYTHONUNBUFFERED=1`  实现上面的功能. 

Dockerfile  中可以使用 `ENV PYTHONUNBUFFERED 1`.

## 9. 命令行直接执行 python 代码

```
python -c 'import foo; print foo.hello()'
```

指定执行路径：

```
PYTHONPATH=/opt/code/py python -c 'import foo; print foo.hello()'
```

## 10. 代码热加载

`importlib` 提供了根据路径，动态加载模块的功能。

```
import sys
import importlib


def load_module(module_name: str):
    """
    加载模块
    :param module_name: str, 例如 "util.new_util", "app_config"
    :return: 
    """
    importlib.import_module(module_name)


def delete_module(module_name: object):
    """ 删除模块 """
    if module_name in sys.modules:
        del sys.modules[module_name]
```

## 11. logger 启用的简单代码

``` 
import logging

logger = logging.getLogger()

logger.setLevel(logging.INFO)
ch = logging.StreamHandler()
ch.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
ch.setFormatter(formatter)
logger.addHandler(ch)

logger.info("hello world!")

```

## 12. urllib 遇到 ssl 证书错误

在一些场合必须使用 python 内置 `urllib` 库. `urllib` 在请求 `https` 时, 会因 `ssl` 证书错误, 而崩溃.

一种临时性的解决方法如下:

```
import ssl
import urllib.request

ssl._create_default_https_context = ssl._create_unverified_context

params = json.dumps({"a":"b", "c": 123})
req = urllib.request.Request(url, data=params.encode('utf-8'), headers={'content-type': 'application/json'})
response = urllib.request.urlopen(req)
content = response.read().decode('utf-8')
print(content)

```

## 13. zip 压缩文件

```
import os
import unittest
import zipfile


def zip_compress_file(src_file: str, zip_file_name: str):
    with zipfile.ZipFile(zip_file_name, mode="w") as zf:
        zf.write(src_file, os.path.basename(src_file), compress_type=zipfile.ZIP_DEFLATED)


def zip_uncompress_file(zip_file_name: str, extract_dir: str):
    with zipfile.ZipFile(zip_file_name) as zf:
        zf.extractall('{}/'.format(extract_dir.rstrip("/")))


class TestPython(unittest.TestCase):
    def testZip(self):
        """ """
        content = "dfadfdasfasdf;ksafk;asdfk;asf,'saf,as"
        file_1 = os.path.expanduser("~/bac.text")
        tar_file = "./x.zip"

        try:
            with open(file_1, "w") as f:
                f.write(content)

            # compress
            zip_compress_file(src_file=file_1, zip_file_name=tar_file)

            self.assertTrue(os.path.exists(tar_file))
            os.remove(file_1)
            self.assertTrue(not os.path.exists(file_1))

            # result
            zip_uncompress_file(zip_file_name=tar_file, extract_dir=os.path.dirname(os.path.abspath(file_1)))

            self.assertTrue(os.path.exists(file_1))
            with open(file_1, "r") as f:
                new_content = f.read()
                self.assertTrue(content == new_content)
        finally:
            for file in [tar_file, file_1]:
                if os.path.exists(file):
                    os.remove(file)

```

## 14. 代码风格

- 禁止 ide 将未使用模块清理: `from os import path  # noqa: F401`

## 15. pickle 大文件

```python

import logging  # noqa
import os  # noqa
import typing  # noqa
import pickle


def dump_cache(obj, cache_file: str):
    max_bytes = 2 ** 31 - 1
    bytes_out = pickle.dumps(obj, protocol=4)
    with open(cache_file, 'wb') as f_out:
        for idx in range(0, len(bytes_out), max_bytes):
            f_out.write(bytes_out[idx:idx + max_bytes])


def load_cache(cache_file: str) -> object:
    max_bytes = 2 ** 31 - 1
    bytes_in = bytearray(0)
    input_size = os.path.getsize(cache_file)
    with open(cache_file, 'rb') as f_in:
        for _ in range(0, input_size, max_bytes):
            bytes_in += f_in.read(max_bytes)
    return pickle.loads(bytes_in)

```