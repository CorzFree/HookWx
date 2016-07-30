
#微信hook
<p>很久之前我的老师说过，他的朋友圈每天都有新款的bra，每天都像发现新大陆.....(此处省略一点点字)</p>
<div>
<img src="http://7xrlno.com1.z0.glb.clouddn.com/93763916-B1C6-4987-8D7F-FBD887919781.png" width="191"/>
</div>

后来有一天，我的朋友圈也有了这东西，单纯如我，悄悄的点开卖家的头像
<div style="width:191px;margin:0 auto;margin-top:-20px">
<img src="http://7xrlno.com1.z0.glb.clouddn.com/76634698-CC69-4064-9F26-6A17F3B788FB.jpg" />
</div>
按下了屏蔽，后面各种广告穿插在朋友圈，每天一条，我忍！但是你每两个小时就要发一条，我只好狠心的屏蔽你了
作为一个零零后，我们该如何优雅的打广告，当朋友圈已经容不下我们卖萌，我们只能寻找另一个地方装x，比如微信运动：  
<div><img src="http://7xrlno.com1.z0.glb.clouddn.com/222IMG_3882.PNG"  width="191"/></div>
用一张笔者的自拍成功的占领了一次封面，换个思路，假若这是一张广告的图片呢（上面有各种各样的bra），现在大多数人都启用了微信运动（只要极个别的人怕打开了之后，每天去哪都被发现了），每天晚上十点，微信会推一条步数排行的信息给用户，我们弄点黑科技，修改下我们的步数，只要微信一推送，就相当于在给我们推广告

###开始之前

在五月份的时候，微博上的一篇文章（[一步一步实现iOS微信自动抢红包(非越狱)](http://www.jianshu.com/p/189afbe3b429))让我开始了解到逆向的世界，之后又开始知道了[乌云网](http://www.wooyun.org)（可惜现在被停摆了），让我了解到，安全只是相对的，没有越狱的iphone，也是可以透露你的信息，期间，我试过几次给微信加入抢红包功能（非越狱），但是每次安装完成都是闪退，直到昨天，再次试了一下，终于成功了，站在巨人的肩膀上，很多事都会简单多了
 ----
###砸壳

我们从AppStore下载的app，其实都加密了，我们要修改微信的某些东西，必须先解密，俗称砸壳，此处有两种方法：

- 从xx助手下载越狱应用（比如PP助手）  
	下载完查看是否已经解密，查看cryptid这个标志位来判断app是否被加密。1代表加密了，0代表被解密了：
	
	> ➜  Desktop otool -l WeChat.app/WeChat | grep crypt    
		     cryptoff 16384    
		    cryptsize 40910848    
		      cryptid 0     
		     cryptoff 16384    
		    cryptsize 43974656    
		      cryptid 0     

 
      
- 自己准备一个越狱机器，手动砸	  
	笔者自己也是参照[一步一步实现iOS微信自动抢红包(非越狱)](http://www.jianshu.com/p/189afbe3b429)来砸壳的，此处只对笔者遇到的一些坑做记录：
	 - 砸壳之后，会生成`WeChat.decrypted`文件，我们需要将其拷贝到我们电脑，操作如下:  
		 1. 打开系统设置->共享->勾上远程登录
	<div>
<img src="http://7xrlno.com1.z0.glb.clouddn.com/5AF02739-2263-46FD-A0C3-3A0FA3036BFE.png.jpeg" width="70%"/>
	</div>  
		
		 2. 使用`scp`拷贝到电脑  
		  `scp WeChat.decrypted crw@192.168.1.104:/Users/crw/Desktop`
	 
	 - 砸壳之后，需要抽出纯净的架构
	  微信下面包含两个架构，以iphone 4砸壳为例：
	  

		> ➜  success otool -l WeChat | grep crypt  
		     cryptoff 16384  
		    cryptsize 40910848  
		      cryptid 0  
		     cryptoff 16384   
		    cryptsize 43974656  
		      cryptid 1  
		      
		只有`armv7`架构是解密的，另一个`arm64`还是加密状态，此处我们需要抽出`armv7`架构，否则后面重签名安装后会闪退:  
		`lipo WeChat -thin armv7 -output WeChat_armv7`  
  
  
--- 

###重签名
我们最终的目的是给微信加上修改步数，抢红包的功能，但这一切的前提必须是，能够成功安装改版后的微信，并且成功打开，所以，我们并不急着给微信加功能，重签名这一步完成后，基本就没有什么大坑了

以下内容摘自[分分钟让你在 微信运动 霸占榜首](http://www.jianshu.com/p/bfd4abd78f21)  

通过Xcode创建一个作为壳子的项目，要使用有开发权限的`bundle Id`
任意取名，选好你的证书描述文件，然后编译。会生成一个`Mytest1.app`

<div >
<img src="http://7xrlno.com1.z0.glb.clouddn.com/e30a91e0-6cd4-4791-ba7b-c78ed29c824b.png" width="70%"/>
	</div>  

我们需要这个Mytest1.app文件中的`embedded.mobileprovision`   
<div style="width:70%;margin:0 auto;padding-bottom:15px;">
<img src="http://7xrlno.com1.z0.glb.clouddn.com/356eb065-d463-4d45-815c-6954b5e8c038.png"/>
	</div>  

然后还需要新建`Entitlements.plist`， 这里需要用到证书的`Team-id`，不知道可以在钥匙串中的证书中找到，注意这里的`Team-id` 一定要是`distribution`证书的。
如： `iPhone Distribution: Tian Xiao (ABCDEFGHIB) 中的 ABCDEFGHIB`  

    <?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
	    <key>application-identifier</key>
	    <string>ABCDEFGHIB.dimsky.MyTest1</string>
	    <key>com.apple.developer.team-identifier</key>
	    <string>ABCDEFGHIB</string>
	    <key>get-task-allow</key>
	    <true/>
	    <key>keychain-access-groups</key>
	    <array>
	        <string>ABCDEFGHIB.dimsky.MyTest1</string>
	    </array>
	</dict>
	</plist>

<div >
<img src="http://7xrlno.com1.z0.glb.clouddn.com/d878d60f-1e55-4069-a209-242ec81ca970.png" width="70%"/>
	</div>  

所有需要的文件都已经生成，然后把`embedded.mobileprovision` 和`WeChat`二进制文件拷贝至`WeChat.app`中替换。

打开`WeChat.app`中的`info.plist`文件，修改`Bundle identifier`为你证书所对应的`Bundle identifier`，与上面的对应，再把粗暴一点，将`WeChat.app`里面的`Watch`文件夹，连同`PlugIns`文件夹一起删去，你也可以将这个两个签名，但是我试了一直有问题，后面我删了，一次成功

签名:  
`codesign -f -s "iPhone Developer: Tian Xiao (XXXXXXXX)" --entitlements Entitlements.plist WeChat.app`

打包:  
`xcrun -sdk iphoneos PackageApplication -v WeChat.app  -o ~/WeChat.ipa`

安装:  
我使用pp助手安装，你也可以选择其他方式，若安装后成功打开微信，恭喜你，还差一步就成功了

---

###注入新功能

这里我整合了[一步一步实现iOS微信自动抢红包(非越狱)](http://www.jianshu.com/p/189afbe3b429)和[分分钟让你在 微信运动 霸占榜首](http://www.jianshu.com/p/bfd4abd78f21)  的代码：  

    if ([m_nsContent rangeOfString:@"打开红包插件"].location != NSNotFound){
		HBPliginType = kOpenRedEnvPlugin;
	}
	else if ([m_nsContent rangeOfString:@"关闭红包插件"].location != NSNotFound){
		HBPliginType = kCloseRedEnvPlugin;
	}
    else if ([m_nsContent rangeOfString:@"关闭抢自己红包"].location != NSNotFound){
        HBPliginType = kCloseRedEnvPluginForMyself;
	}
	else if ([m_nsContent rangeOfString:@"关闭抢自己群红包"].location != NSNotFound){
        HBPliginType = kCloseRedEnvPluginForMyselfFromChatroom;
	}
	else if ([m_nsContent rangeOfString:@"修改微信步数#"].location != NSNotFound){
		NSArray *array = [m_nsContent componentsSeparatedByString:@"#"];
        if (array.count == 2) {
               StepCount = ((NSNumber *)array[1]).intValue;
		}
	}
	else if([m_nsContent rangeOfString:@"恢复微信步数"].location != NSNotFound){
		StepCount = -1;
	}
                
	if (StepCount != kNothing) SAVESETTINGS(StepCountKey, [NSNumber numberWithInteger:StepCount]);
                
	if (HBPliginType != kNothing) SAVESETTINGS(HBPliginTypeKey, [NSNumber numberWithInt:HBPliginType]);

编译得到：  

<div style="width:50%;margin:0 auto;padding-bottom:15px;">
<img src="http://7xrlno.com1.z0.glb.clouddn.com/4651846B-E13E-49B8-93DF-FF8F4B65EAB5.png"/>
	</div>  

将dylib注入:
`./yololib WeChat Runaway.dylib`

然后将` Runaway.dylib` 拷贝到`WeChat.app `中，对` Runaway.dylib`进行签名:    
`codesign -f -s "iPhone Developer: Tian Xiao (XXXXXXX)" WeChat.app/Runaway.dylib `

再根据上面的重签名走一次，安装，若在iphone上成功打开，我们的工作已经完成，接下来，只要：  

<div >
<img src="http://7xrlno.com1.z0.glb.clouddn.com/IMG_3907.PNG" width="191"/>
	</div>  


---
**注：**  
**以上广告部分纯属个人yy，本文仅供学习交流，请勿用于商业用途**
