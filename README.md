Leptonica 1.84.0 在Windwos 10下编译说明：

编译需要CMaker、SW、VS2022。在编译之前，我们还要使用vcpkg生成依赖的若干图像库。

cmake-3.27.7-windows-x86_64.msi 下载链接：
https://github.com/Kitware/CMake/releases/download/v3.27.7/cmake-3.27.7-windows-x86_64.msi

sw-master-windows-client.zip 下载链接：
https://github.com/SoftwareNetwork/binaries
下载到一个固定目录（这个目录以后不要删除，否则需要重新配置sw），右键以管理员打开cmd，进入这个目录，执行sw setup，看运气等着sw提示下载结束（国内网络不一定能下载，多试几十次，或者用梯子）。

vcpkg.exe 下载链接：
https://github.com/microsoft/vcpkg/archive/refs/heads/master.zip

将vcpkg的master.zip解压缩到D:\vcpkg
以管理员启动cmd，执行bootstrap-vcpkg.bat。
如果速度慢，把scripts目录里面的bootstrap.ps1中的github.com改成镜像地址，例如hub.njuu.cf。

编译需要的工具下载安装完毕，下面开始干活。

第一步：下载并编译图像库，步骤如下：
vcpkg install tiff --triplet=x64-windows

vcpkg install libpng --triplet=x64-windows

vcpkg install giflib --triplet=x64-windows

vcpkg install ijg-libjpeg --triplet=x64-windows  不成功，大概是要使用libjpeg-turbo

vcpkg install libwebp --triplet=x64-windows

vcpkg install tiff --triplet=x86-windows

vcpkg install libpng --triplet=x86-windows

vcpkg install giflib --triplet=x86-windows

vcpkg install libwebp --triplet=x86-windows

github连不上会报错，提示gz文件下载地址和要生成的文件名，用镜像站（例如 hub.njuu.cf）下载gz文件，然后改名为要生成的文件名，放到vcpkg目录的downloads里面，重新运行vcpkg install 即可。

vcpkg会自动编译debug和release
位于packages目录里面

也可以直接用
vcpkg install leptonica --triplet=x64-windows-static    ----------  生成 leptonica.dll 静态库（包括8个依赖），这个最省心
依赖gif.dll，jpeg62.dll，openjp2.dll，libpng16.dll，zlib1.dll，tiff.dll，libwebp.dll，libwebpmux.dll
问题是我不知道怎么把图像库静态链接到leptonica.dll里面，谁要是知道记着发一个issus告诉我。

第二步，使用CMake编译
在leptonica-master目录中创建build目录。启动CMaker，设置source和build目录。点一次Configure，耐心等着sw下载完毕（会提示一大堆sw下载信息，等好久。然后是一堆Performing  check，继续等。
（如果上面这步报错，删除build目录中的全部文件，然后重试。）
再点一次Configure，然后点一次Generating，这会生成sln，给vs2022编译用。

新建libs\leptonica-master\src\webp，下载libwebp源代码（https://github.com/webmproject/libwebp），拷贝libwebp\src\webp目录里面的.h头文件，到上面的目录，1.83.x的leptonica编译需要它们（1.84.0好像不需要了）。
然后使用vs2022在leptonica-master\build目录中打开leptonica.sln，可以直接编译成功，生成的是lib文件。

第三步，配置VS2022，生成Dll
CMake生成的leptonica.sln（默认是Release x64）只能编译静态库lib。按照下面的步骤配置VS2022：
打开leptonica项目的属性，做如下配置：
1、配置属性--常规--输出目录改为 $(SolutionDir)$(Configuration)\$(Platform)\
   配置属性--常规目标文件扩展名，改为.dll

2、链接器--常规--附加库目录，输入 $(SolutionDir)libs\$(Platform)

3、在项目工程目录\build中创建libs目录，在libs目录中，再创建x64和Win32（x86）两个目录，分别放入vcpkg生成的x64和x86静态库文件（共10个）：
gif.lib;jpeg.lib;openjp2.lib;libpng16.lib;zlib.lib;tiff.lib;libwebp.lib;libwebpmux.lib;libsharpyuv.lib;lzma.lib;
还有一个turbojpeg.lib，目前不需要它。

4、链接器--输入--附加依赖项，添加十个链接库:
gif.lib;jpeg.lib;openjp2.lib;libpng16.lib;zlib.lib;tiff.lib;libwebp.lib;libwebpmux.lib;libsharpyuv.lib;lzma.lib;

5、链接器--输入--忽略特定默认库，添加libcmt.lib（Release） 或 libcmtd.lib（Debug）。否则编译会报错：LNK4098	默认库“LIBCMT”与其他库的使用冲突；请使用 /NODEFAULTLIB:library	

6、C/C++预处理器定义，添加 LIBLEPT_EXPORTS，修改之后的预处理器定义如下：
%(PreprocessorDefinitions);WIN32;_WINDOWS;NDEBUG;LIBLEPT_EXPORTS;HAVE_LIBGIF=1;HAVE_LIBJPEG=1;HAVE_LIBPNG=1;HAVE_LIBTIFF=1;HAVE_LIBWEBP=1;HAVE_LIBWEBP_ANIM=1;HAVE_LIBZ=1;HAVE_LIBJP2K=1;_CRT_SECURE_NO_WARNINGS;HAVE_CONFIG_H;SW_EXPORT=__declspec(dllexport);SW_IMPORT=__declspec(dllimport);JPEG_API=;PNG_IMPEXP=;LZMA_API_STATIC;OPJ_STATIC;WEBP_API=;CMAKE_INTDIR="Release"

7、在项目的配置管理器中增加x86或Win32配置，注意要勾选leptonica项目。注意：libs的对应目录也应该是x86或Win32。

编译结果：
以Release x64编译，在bulid\src\release\x64中生成leptonica-1.84.0.dll
以Release Win32编译，在bulid\src\release\Win32中生成leptonica-1.84.0.dll

Charltsing, 2023.12.26 测试通过。

如果有其它问题，请联系我的QQ 564955427，Email: liucq@163.com
