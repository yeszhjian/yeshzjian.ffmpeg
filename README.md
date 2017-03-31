You can use the [editor on GitHub](https://github.com/yeszhjian/yeshzjian.ffmpeg/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

配置ndk环境

启动终端Terminal
进入当前用户的home目录 
输入cd ~ 或 /Users/YourUserName
创建.bash_profile 
输入touch .bash_profile
编辑.bash_profile文件

输入open -e .bash_profile
因为是为了配置NDK开发环境，输入Android NDK下目录,前面是android sdk的，可以不用动它，最终.bash_profile文件如下：
export PATH=$(PATH):/Users/yeszhjian/android-sdks/platform-tools    
export NDK_ROOT=/Users/yeszhjian/Downloads/android-ndk-r10e                    
export PATH=$PATH:$NDK_ROOT

保存文件，关闭.bash_profile
更新刚配置的环境变量 
输入source .bash_profile
看看刚刚设置的环境变量
离开了编辑器后，在终端输入 $PATH 并且按enter键来确认是否编辑成功，此时应该会出现所有的环境变量(以：号相分隔) 

这里写图片描述 

表明配置成功
接下来·开始进行测试ndk是否能正常编译jni 
(1) 终端进入到 NDK下面的 samples 目录下。 
(2) 输入 cd hello-jni/ ，回车，然后执行 ndk-build 
出现以下界面代表配置成功。 


编译FFmpeg
在编译前，在源码中，修改FFmpeg的configure
下载FFmpeg源代码之后，首先需要对源代码中的configure文件进行修改。由于编译出来的动态库文件名的版本号在.so之后（例如“libavcodec.so.5.100.1”），而android平台不能识别这样文件名，所以需要修改这种文件名。在configure文件中找到下面几行代码(在3209-3212行)：
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'  
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'  
替换为下面内容：
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'  
SLIB_INSTALL_LINKS='$(SLIBNAME)'  
接下来开始写shell脚本

这种情况应该有两种原因：
1.在WIN底下用文本编辑工具修改过参数变量，在保存的时候没注意编码格式造成的，
2.也有可能是在VIM里修改，第一行末尾按到ctrl+v 
这里避开这个弯，我找到FFmpeg下一个version.sh的shell脚本，复制了一份 
重命名为build_android.sh。脚本如下：

#!/bin/sh
NDK=/Users/yeszhjian/Library/Android/sdk/ndk-bundle
SYSROOT=$NDK/platforms/android-24/arch-arm
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
function build_one
{
./configure \
--prefix=$PREFIX \
--enable-shared \
--disable-static \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-avdevice \
--disable-doc \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--target-os=linux \
--arch=arm \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
}
CPU=arm
PREFIX=$(pwd)/android/$CPU
ADDI_CFLAGS="-marm"
build_one

如果大家要编译，记得改下前三行，对应自己机器上的环境 
接着开始执行这个shell脚本，在终端输入 ./ build_android.sh 

开始进行自动编译,编译完后这时会发现 FFmpeg下多了一个文件夹android，所有的头文件跟.so库都在下面。


移植到Android平台

接下来写在Android studio写一个示例，调用ffmpeg中方法 
建一个工程：在src/main下建一个jni目录 
把前面编译好的android目录移植过来 
