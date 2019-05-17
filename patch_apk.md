#  去掉APK的自校验功能

----------
## 背景

> 这是我在DIY一个apk的时候，自己需要的功能都修改完了，但最后打包，签名后，安装到机器上后，弹出以下图片，发现这个apk有自校验，今天主要说说怎么去掉这个APK的自校验。

![mark](http://fs-image.pull.net.cn/blog/20190516/a4RFJkMHW2Cb.png!pusafe2)

## 通常使用的几种自校验方式
- 证书校验
- 文件指纹校验

## 测试证书校验
**把机器上的apk卸载，然后用一个官方的apk,打开后，（把证书删除掉），只是重新打包一次，看看是不是证书问题。**

> 具体方法如下：
> 
> adb pull /data/app/base.apk （从机器中提取出apk）
> adb instatll base.apk 修改完apk后，重新打包，然后安装到本机
> 打开机器上的APK后，出现一句话“您使用的不是官方的app,请重新下载”，然后就自动退出了。
> 当然不是官方的APK,这个被我改过了（而且只是换了证书，其他的没变），显然是有自校验代码。
> 这次运气好（看来做什么都需要运气），一次就被我找到了关键点。

## 找验证证书的关键代码,patch掉这处检查

打开jeb2工具，反编译APK：

**搜索“您使用的不是官方的app,请重新下载",找到关键点：**
![mark](http://fs-image.pull.net.cn/blog/20190516/PEiH8nTUG37O.png!pusafe2)

**看看这个函数的调用者，原来是跟证书相关的**

![mark](http://fs-image.pull.net.cn/blog/20190516/YFfJwPYMw9ep.png!pusafe2)

**进入IM().__看到原来是调用了bdcvf.so文件的a函数进行对证书的校验**

![mark](http://fs-image.pull.net.cn/blog/20190516/P277DSeSQcHf.png!pusafe2)

**看到了OnCreate函数，应该是到头了：**

![mark](http://fs-image.pull.net.cn/blog/20190516/eXnIP9Hmw5Xl.png!pusafe2)

**打开ida pro，载入bdcvf.so文件，找到a函数：**

![mark](http://fs-image.pull.net.cn/blog/20190516/WfnLWVvUAF23.png!pusafe2)

**找到校验的地方**

![mark](http://fs-image.pull.net.cn/blog/20190516/yHlEnfP1t782.png!pusafe2)

**下面就是算法了**
![mark](http://fs-image.pull.net.cn/blog/20190516/FmHeYlg628tC.png!pusafe2)

## 总结一下，大概流程图

![mark](http://fs-image.pull.net.cn/blog/20190516/ucuk6PBe10Ir.png!pusafe2)


## patch方法
**我们的目的是去掉apk的自校验，具体怎么校验的就不看了，我说一下patch的方法
在so文件的a方法中，大于等于0，改为JUMP,让他必须成功，这个自校验就去掉了。**


##检查成果
用apk_ide重新打包，然后按装，点开修改后的APK，正常运行，然后你就可以随意进行篡改代码了。

## 不足
这种修改有一个弊端，如果服务器对apk证书进行校验的话，服务器可能会停止对这个apk的服务器，那么你这样修改就没有意义了。



