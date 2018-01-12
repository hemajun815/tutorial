# 配置pip使用国内源
对于Python开发用户来讲，使用pip安装软件包是家常便饭。但国外的源下载速度实在太慢，浪费时间，而且经常出现下载后安装出错问题。所以把pip安装源配置成国内镜像，可以大幅提升下载速度，从而提高安装成功率。
## 国内源
- 清华：https://pypi.tuna.tsinghua.edu.cn/simple
- 阿里云：http://mirrors.aliyun.com/pypi/simple/
- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
- 华中理工大学：http://pypi.hustunique.com/
- 山东理工大学：http://pypi.sdutlinux.org/ 
- 豆瓣：http://pypi.douban.com/simple/
## 临时使用
可以在使用pip的时候加参数`-i https://pypi.tuna.tsinghua.edu.cn/simple`
> 示例：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyspider` 这个命令就会从清华源的镜像中安装pyspider库了。
## 永久修改
#### linux环境
修改 ~/.pip/pip.conf (没有就创建文件夹及文件)文件，内容如下：
```console
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```
#### Windows环境
在当前用户主目录下创建pip目录，如：C:\Users\××\pip，新建文件pip.ini，内容如下：
```console
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```
## 注意
新版Ubuntu系统要求使用https源。
