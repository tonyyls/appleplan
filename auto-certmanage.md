## 证书自助管理思路&技术点

在线编译iOS应用在移动应用开发管理过程中及其重要。

大体思路：通过程序驱动Mac终端命令行实现P12证书安装, 描述文件的导入、删除以及合法性校验。

iOS自助管理的原型大概是这样：
![自助管理原型](http://img.blog.csdn.net/20160312220928381)

从上图可以看出，需要打包者提供用于发布程序的App ID、 p12证书、证书密码、以及对应的描述文件。

## 技术点一, 利用security命令安装p12证书
根据原型，我们选择p12文件后，需要将p12文件导入到Mac 机器，检验证书的时候需要先将p12安装到Mac的Keychain里面。
导入的命令如下：

``` shell
security import `p12_filepath` -k /Users/yulongsheng/Library/Keychains/login.keychain -P `p12_password` -T /usr/bin/codesign
```

注意：

p12_filepath 是在Mac机器上的物理路径，单引号是不需要的，这里只是为了强调；

p12_password 是p12文件对应的导入密码，单引号同样也是不需要的；

yulongsheng 是机器的名称，用回编译服务器的即可。 

输入正确的命令导入后：
![导入证书](http://img.blog.csdn.net/20160312223310616)

程序执行上面的命令，通过 BufferedInputStream 和 BufferedReader 可以获取到返回的字符串信息，根据关键字来判断证书是否导入成功（文末有Java调用示例）。

注意：如果Keychain是输入Lock状态的，证书是无法正常导入的，需要先解锁，手工解锁或者用下面的命令解锁。

![锁住的KeyChain](http://img.blog.csdn.net/20160312224154940)

解锁命令

```
security unlock-keychain -p `mac_password` /Users/yulongsheng/Library/Keychains/login.keychain
```

注：mac_password 是Mac机器的登录密码

## 技术点二, 解析描述文件 mobileprovision获取相关信息

mobileprovision描述文件里面包含了 AppId，证书信息、设备信息等，要得到这些信息得费些功夫。mobileprovision文件直接用文本工具打开是无法看到里面的内容的，经过特殊编码的文件。 

![这里写图片描述](http://img.blog.csdn.net/20160312225059194)

因此需要对其进行解析，先将其转成plist文件（xml格式的文件），命令如下：

```
security cms -D -i provisionfile > plistfile
```

执行结果如下：

![描述文件转plist](http://img.blog.csdn.net/20160312225635478)

描述文件的真面目：

![描述文件真面目](http://img.blog.csdn.net/20160312230221716)

这些信息我们可以通过程序读取出来，通过security快速的获取UUID 和 过期时间，命令如下：

```
#获取uuid,下个章节有用
/usr/libexec/PlistBuddy -c 'Print UUID' /Users/yulongsheng/Desktop/Time/chendu.plist

＃获取描述文件过期时间，打印出来的格式需要转换下才行
/usr/libexec/PlistBuddy -c 'Print ExpirationDate' /Users/yulongsheng/Desktop/Time/chendu.plist
```

其它的信息，例如 AppId, TeamIdentifier, TeamName等无法快速拿到，只能通过程序解析plist后才能得到。

## 技术点三, 安装描述文件
手工安装的情况下，只需要双击安装即可，观察其安装的路径不难发现，实际上，它是被安装到了 ~/Library/MobileDevice/Provisioning Profiles/  目录下：

![描述文件路径](http://img.blog.csdn.net/20160312232730631)
 
该目录下的描述文件将会被xcode读取到。另外，注意观察 描述文件安装进来后是 以uuid.mobileprovision命名的，因此上一个技术点里面获取的uuid就可以派上用场了。 

##技术点四, 校验p12文件和描述文件是否一致

校验点：

1.TeamName是否匹配的上；

2.描述文件是否过期（只要它不过期，P12肯定没过期）


### TeamName是否和P12对应
假设已经从plist里面获取到了TeamName，例如：
ChengDu QIDI Info Co., Ltd.
那么您需要在KeyChain里面去找到是否安装了带有这个关键字的证书，用下面的命令可以查看到安装在KeyChain里面的合法的证书：

```
security find-identity -v codesigning /Users/yulongsheng/Library/Keychains/login.keychain
```
![合法的证书](http://img.blog.csdn.net/20160312234223089)

因此，如果TeamName在输出的合法的证书里面找到了，那么就认为描述文件和P12是对应上了。 

### 校验描述文件过期时间
从上面可以获取到ExpirationDate，与当前时间进行匹配，当然，最好多考虑几天或者1周时间，在界面上提醒用户 还剩下多长时间。

##技术点五, Java执行Mac下shell命令

前面几个技术点谈到了用shell命令来实现各种操作，但在GUI层面，我们只能选择用Java程序来驱动这些命令。简单调用示例如下：

```java
package com.bingo.main;

import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStreamReader;  
import java.util.ArrayList;  
import java.util.List;  

public class main {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		Process process = null;  
        List<String> processList = new ArrayList<String>();  
        try {  
        	String cmd="security find-identity -v codesigning /Users/yulongsheng/Library/Keychains/login.keychain";
        	//执行终端命令
        	process = Runtime.getRuntime().exec(cmd);
        	BufferedReader input = new BufferedReader(new InputStreamReader(process.getInputStream()));  
            String line = "";  
            while ((line = input.readLine()) != null) {  
                processList.add(line);  
            }  
            input.close();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        //输出返回结果
        for (String line : processList) {  
            System.out.println(line);  
        }  
	}

}
```
![输出执行结果](http://img.blog.csdn.net/20160313002259091)

以上就是iOS证书自助管理需要攻破的技术点！