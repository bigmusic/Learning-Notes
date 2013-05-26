**我的Cubieboard系统,基于Linaro-Nano:**
[http://cn.cubieboard.org/forum.php?mod=viewthread&tid=347](http://cn.cubieboard.org/forum.php?mod=viewthread&tid=347)
*ps:谢谢chinaspc同学啊啊啊啊啊啊~~*

*我编译的时候只安装在自己的目录里,用'ln -s'来切换版本,可以参考这里*
[https://increaseyourgeek.wordpress.com/2010/08/18/install-node-js-without-using-sudo/](https://increaseyourgeek.wordpress.com/2010/08/18/install-node-js-without-using-sudo/)

**参考ArchLinuxARM的PKGBuild的History,可以自己编译ARM版的Node.js了**

[https://github.com/archlinuxarm/PKGBUILDs/commits/master/community/nodejs](https://github.com/archlinuxarm/PKGBUILDs/commits/master/community/nodejs)

>这里是for 0.6.19 only的:
>在PKGBuild有段很重要的注释,导向了一个解决0.6.19的办法
>[https://github.com/joyent/node/issues/2131#issuecomment-3208846](https://github.com/joyent/node/issues/2131#issuecomment-3208846)
>
>当然,A10是Armv7-a,这里要把东西参数改一下,进入node的源码目录里的deps/v8,找到SConstruct,编辑之
>
>在82行加入armv7-a,修改完后是这样的
><pre><code>80 'gcc': {
>81      'all': {
>82        'CCFLAGS': ['$DIALECTFLAGS', '$WARNINGFLAGS', '-march=armv7-a'],
></code></pre>
>在1083改为hard,修改完后是这样的
><pre><code>1081  'armeabi': {
>1082    'values': ['hard', 'softfp', 'soft'],
>1083    'default': 'hard',
></code></pre>
><p>保存退出</p>

--------------------------------------------------------------------------------------------------------------
**在node的源码根目录,建立一个install.sh文件,把对应版本的注释去掉,按Enter前看看有没有错误信息,比如没找到ssl之类的**


    #for 0.10.5
    #export GYPFLAGS="-Darmeabi=hard -Dv8_use_arm_eabi_hardfloat=true -Dv8_can_use_vfp3_instructions=true -Dv8_can_use_vfp2_instructions=true -Darm7=1"
    #./configure --prefix=/home/bigmusic/opt/node-v0.10.5 --without-snapshot --with-arm-float-abi=hard
    
    #for 0.8.22
    #export GYPFLAGS="-Darmeabi=hard -Dv8_use_arm_eabi_hardfloat=true -Dv8_can_use_vfp3_instructions=true -Dv8_can_use_vfp2_instructions=true -Darm7=1"
    #./configure --prefix=/home/bigmusic/opt/node-v0.8.22 --without-snapshot --with-arm-float-abi=hard
    
    
    #for 0.6.19
    #./configure --prefix=/home/bigmusic/opt/node-v0.6.19 --without-snapshot --openssl-libpath=/usr/lib/ssl --openssl-includes=/usr/include/openssl
    
    read -n 1 -p "Press ENTER to Continue or Press Ctrl+C to exit..."
    
    make clean
    
    make
    
    make install


<p>键入./install.sh进行编译,路径要自己改好喔</p>
