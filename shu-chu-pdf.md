#### gitbook输出pdf

执行命令：  


```
gitbook pdf
```

如果环境没有完全装好，肯定会报错，我这里报了如下错误：

InstallRequiredError: "ebook-convert" is not installed.

Install it from Calibre: https://calibre-ebook.com

这时候需要去安装应用：  
[https://calibre-ebook.com](https://calibre-ebook.com)

安装完后重启终端：

再执行：

```
gitbook pdf
```

可能还会出错，可以尝试一下一下命令（Mac版）：

```
ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/local/bin
```

这个时候，我的执行

```
gitbook pdf
```

成功了。

![](/assets/exportpdf.png)

![](/assets/exportpdfok.png)

