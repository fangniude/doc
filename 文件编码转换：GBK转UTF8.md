因为历史原因，项目中的一些文件，是GBK编码的，需要统一转成UTF-8。

可以使用以下命令：

```shell 
find ./ -name "*.properties" -exec enca -x UTF-8 {} \;
```

该命令中，用到了2个命令：find 和 enca。

find为系统自带，enca是我手工安装的。

```shell 
brew install enca
```

先找到所有的 properties文件，然后用enca将文件转码。

**enca有个好处：**如果原来的编码就是目标编码，则不会转。