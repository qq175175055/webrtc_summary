# 修改配置添加ffmpeg组件

[toc]

## 1. 正规军操作方法

### 1.1 mac或者linux平台下编译并添加ffmpeg组件

mac或者linux平台下是最简单的，不涉及交叉编译，很多工具系统已经具备。  

第一步：./configure   (后面可以跟其他选项，如果不跟 是默认配置,此命令功能是生成makefile文件，具体的选项后面会讲解到)  

```bash
./configure  --prefix=/Users/feng/third-src/FFmpeg/FFmpeg-dir
```

第二步：make 或make -j10 （j10是同时开10个线程执行，此命令是执行makefile文件生成二进制文件比如 *.o  *.bin  *.lib）  

```bash
make -j10
```

第三步：make install （把生成的二进制文件安装到configure设置的prefix路径)  

```bash
make install
```

## 2. webrtc中ffmpeg的编译和配置

webrtc中ffmpeg的编译是通过ninja进行的，主要是编写gn来构建编译脚本。其中生成的webrtc的库中会包含ffmpeg相关的obj数据。  

### 2.1 理想中的编译过程

第一步：进入third_party/ffmpeg/chromium/scripts目录，执行脚本build_ffmpeg.py

```bash
# 以android为例，开启avfilter和libfreetype组件
./build_ffmpeg.py android arm-neon --enable-avfilter --enable-libfreetype
```

其中build_ffmpeg.py中内置编写了很多配置，也可以手动修改。如（--disable-everything，#禁止所有选项目）  

第二步：执行脚本copy_config.sh，将第一步生成的相关配置拷贝到对应的目录中。

```bash
./copy_config.sh
```

第三步：执行脚本generate_gn.py，根据第二步的配置，完成third_party/ffmpeg/ffmpeg_generated.gni的生成。其中third_party/ffmpeg/BUILD.gn中会引用ffmpeg_generated.gni中的变量。

```bash
./generate_gn.py
```

第四步：回到webrtc，并进行编译

```bash
ninja -C out/arm-neon
```

上述是理想的情况，但是现实是骨感的。上述的每一步都可能出错，其中前三步是最容易出错的。只能按照错误一步一步解决。

### 2.2 实用的编译过程

根据2.1中描述的过程，可以手工修改相关的配置，以达到最后的编译过程。此种方法适用于添加少量ffmpeg的组件。具体如下：

第一步：进入third_party/ffmpeg/chromium/config/Chrome/android/arm-neon/目录，打开config.h文件。其中Chrome是对应的brand，android/arm-neon对应平台和架构。在config.h中我们可以看到很多宏定义，这些宏定义描述了本次编译过程是否编译对应的组件。例如：  

```c++
#define CONFIG_DRAWTEXT_FILTER 0 
// 代表不编译drawtext filter，因此后续寻找drawtext这个filter时，会失败
```

第二步：进入third_party/ffmpeg/目录，打开ffmpeg_generated.gni，添加对应的.c源文件。这个过程可能无法一蹴而就，可以根据编译过程中出错来一步一步添加。如果仅仅增加某个组件，往往仅需要添加很少的源文件。

第三步：回到webrtc，并进行编译。
