# Unity Jenkins打包

[toc]

## 项目打包注意点

因为我们使用的xlua作为热更新，所以我们这里需要编译lua和xlua，是我们能够在unity中使用，要想在Unity中使用lua以及XLua，lua和xlua的编译需要特定的版本，涉及到CMake，NDK，SDK等。

1. cmake版本
2. android sdk版本
3. ndk 版本



## Mac Os下搭建Unity环境

1. Unity的安装
2. XCode安装
3. VS安装
4. Android Studio的安装
5. IDEA的安装

## Android打包

### 编译XLua

1. 设置环境变量 cd ~，进入用户主目录
2. source ./bash_profile,每次重新启动终端都要这么做，我们可以配置一下，`vim ~/.zshrc`,然后在里面加上一句`source ~/.bash_profile`
3. 然后切换到我们的xlua build目录中，执行 `bash make_android_lua53.sh`,然后在plugin_lua53中我们可以看到plugin目录，将android的plugin拷贝到unity中的plugin android目录中。

### 生成签名，并配置到Unity中

1. 打开android studio，build中的generatesigned bundle / apk,选择Apk，然后改配置，生成签名,这里的alias，可以设置成你们的游戏名。
2. 然后在控制台重新导出一下key，`keytool -importkeystore -srckeystore /Users/catserver/trunk/client/Key/dorocat -destkeystore /Users/catserver/trunk/client/Key/dorocat.keystore -deststoretype pkcs12`
3. 在Unity的player setting中的other setting中配置签名，首先导入签名，然后key里面的alias中选择别名，并加上密码。

现在可以看到在调用到CS.UnityEngine.Debug时候，我们在Lua中已经获取到了这个类对应Lua Table了，那么接下来调用CS.UnityEngine.Debug.Log("hello world")，大提上与之前的获取Type类是一致的。不过要注意的是调用static的方法字段和对象的方法字段使用的是不同的table。**这次文章都讨论的是静态方法的调用.**

