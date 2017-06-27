<center>**selenium环境搭建**</center>

**太长不看版：**
-
**装python===> 下载selenium===> 下载浏览器驱动**

**done！**
# python环境搭建
## 1. 下载
Python最新源码，二进制文档，新闻资讯等可以在Python的官网查看到：

Python官网：http://www.python.org/

你可以在以下链接中下载 Python 的文档，你可以下载 HTML、PDF 和 PostScript 等格式的文档。

Python文档下载地址：http://www.python.org/doc/
## 2. 安装
### Unix & Linux 平台安装 Python:
打开WEB浏览器访问http://www.python.org/download/

选择适用于Unix/Linux的源码压缩包。

下载及解压压缩包。

如果你需要自定义一些选项修改Modules/Setup 

```shell
./configure 
make
make install
```

执行以上操作后，Python会安装在 /usr/local/bin 目录中，Python库安装在/usr/local/lib/pythonXX，XX为你使用的Python的版本号。

### Windows 平台安装 Python:
打开WEB浏览器访问http://www.python.org/download/

在下载列表中选择Window平台安装包，包格式为：python-XYZ.msi 文件 ， XYZ 为你要安装的版本号。

要使用安装程序 python-XYZ.msi, Windows系统必须支持Microsoft Installer 2.0搭配使用。只要保存安装文件到本地计算机，然后运行它，看看你的机器支持MSI。Windows XP和更高版本已经有MSI，很多老机器也可以安装MSI。
 
下载后，双击下载包，进入Python安装向导，安装非常简单，你只需要使用默认的设置一直点击"下一步"直到安装完成即可。

### MAC 平台安装 Python:

自带，不用装
## 环境变量配置

程序和可执行文件可以在许多目录，而这些路径很可能不在操作系统提供可执行文件的搜索路径中。

path(路径)存储在环境变量中，这是由操作系统维护的一个命名的字符串。这些变量包含可用的命令行解释器和其他程序的信息。

Unix或Windows中路径变量为PATH（UNIX区分大小写，Windows不区分大小写）。

### Mac OS
不用配置

### 在Unix/Linux 设置环境变量

- 在 csh shell: 

输入 ```setenv PATH "$PATH:/usr/local/bin/python"```, 按下"Enter"。

- 在 bash shell (Linux): 

输入 ```export PATH="$PATH:/usr/local/bin/python"``` ，按下"Enter"。

- 在 sh 或者 ksh shell: 输入 ```PATH="$PATH:/usr/local/bin/python"``` , 按下"Enter"。

注意: /usr/local/bin/python 是Python的安装目录。

### 在 Windows 设置环境变量

在环境变量中添加Python目录：

在命令提示框中(cmd) : 
输入 ```path=%path%;C:\Python``` 按下"Enter"。

注意: C:\Python 是Python的安装目录。

也可以通过以下方式设置：

1. 右键点击"计算机"，然后点击"属性"

2. 然后点击"高级系统设置"
选择"系统变量"窗口下面的"Path",双击即可！

3. 然后在"Path"行，添加python安装路径即可(我的D:\Python32)，所以在后面，添加该路径即可。ps：记住，路径直接用分号"；"隔开！

最后设置成功以后，在cmd命令行，输入命令"python"，就可以有相关显示。

## 运行Python
你可以通过命令行窗口进入python并开在交互式解释器中开始编写Python代码。
你可以在Unix，DOS或任何其他提供了命令行或者shell的系统进行python编码工作。

```shell
$ python # Unix/Linux 

或者 

C:>python # Windows/DOS
```

## API
https://docs.python.org/2/index.html

# selenium安装
https://pypi.python.org/pypi/selenium

## 1. 安装pip
https://pip.pypa.io/en/stable/installing/#installing-with-get-pip-py

下载get-pip.py，然后运行```python get-pip.py```

## 2. 安装selenium
```shell
pip install -U selenium
```

## 3. 安装drivers

Chrome: https://sites.google.com/a/chromium.org/chromedriver/downloads

Edge:   https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/

Firefox:    https://github.com/mozilla/geckodriver/releases

Safari: https://webkit.org/blog/6900/webdriver-support-in-safari-10/

**Make sure it’s in your PATH, e.g., place it in /usr/bin or /usr/local/bin.**

## 4. 测试是否安装成功
运行下面python代码，能自动启动浏览器并且打开url则安装成功

```python
from selenium import webdriver
browser = webdriver.Firefox()#根据下载的驱动选择对应的浏览器
browser.get('http://seleniumhq.org/')
```





