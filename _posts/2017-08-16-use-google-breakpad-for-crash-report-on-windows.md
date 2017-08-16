---
layout: post
title:  "使用Google开源库breakpad实现错误报告功能（Windows环境）"
subtitle: "Google大法好"
date:   "2017-08-16" 
author: "cj"
tags:
    breakpad
    c++
    windows
    google
    bugreport
---

Windows环境下以前使用的CrashRpt1403错误报告系统出了点问题，问题上传端口（80）被微信服务器测试环境占用了，索性使用breakpad更新之。

# 目录

1. [环境搭建](#环境搭建)
    1. [下载](#下载)
    2. [编译](#编译)
2. [部署](#部署)
    1. [生成sym并上传](#生成sym并上传)
        1. [Windows publish](#Windows publish)
        2. [Linux publish](#Linux publish)
    2. [应用程序植入breakpad](#integrate)
    3. [处理应用程序崩溃](#report)
        1. [Windows](#report_win)
        2. [Linux](#report_linux)
    4. [邮件示例](#mail)
3. [参考资料](#refs)

# <a name=install></a>环境搭建

## <a name=download></a>下载

按照[官方教程](https://chromium.googlesource.com/breakpad/breakpad)，下载源码[jump]

```bash
git clone https://chromium.googlesource.com/breakpad/breakpad
```

有几个第三方库默认情况没有下载，按需手动下载到breakpad/src/thirdparty中 

```bash
cd breakpad
git clone https://chromium.googlesource.com/linux-syscall-support src/third_party/lss
```

## <a name=build></a>编译

与linux环境下直接configure&&make不同，Windows下需要google另一个工具gyp生成visual studio 的sln文件。另外gyp依赖Python2.x版本，如果系统环境变量PATH中指定的python.exe已经是Python3.x版的话，需要手动修改。Python2.x与3.x共用方案网上一搜大把不再赘述。

安装gyp：

```bash
git clone https://chromium.googlesource.com/external/gyp
cd gyp
python setup.py install
```

生成sln解决方案：

```bash
your-path-to-gyp/gyp.bat your-path-to-breakpad/src/client/windows/breakpad_client.gyp --no-circular-check
```

打开breakpad_client.sln并编译所需的库文件：

* ommon.lib
* exception_handler.lib
* crash_generation_client.lib


# <a name=deploy></a>部署

在Windows下编译好应用程序后，使用breakpad的工具dump_syms.exe生成sym符号表，并上传至服务器。当客户环境中的应用程序崩溃时，调用一个脚本将dump信息上传至服务器，服务器端处理一下并生成stack walk信息发送到自己的邮箱。思路与上一篇linux版完全一样，只不过生成sym的环境换成了Windows，并增加了web服务以便处理。

下面以分为新版本发布时的“生成sym”和崩溃发生时的“处理dmp”记录一下。

## <a name=gen_sym></a>生成sym并上传

新版本发布时，自动生成sym文件并上传

### <a name=gen_sym_win></a>Windows环境运行

参考了这篇[文章](https://www.chromium.org/developers/decoding-crash-dumps)，摘录如下：

>Decoding Windows crash dumps on Linux
>
>Windows crash dumps can be decoded the same way as Linux crash dumps. The issue is mainly getting the debugging symbols as a .sym file instead of a .pdb file.
>
>To convert a .pdb file to a .sym file:
> 1. Obtain the .pdb file and put it on a Windows machine. (It may be possible to do this with Wine, YMMV.)
> 2. Download dump_syms.exe.
> 3. Run: dump_syms foo.pdb > foo.sym
>       * If no error messages, then you are done.
>       * If you get: CoCreateInstance CLSID_DiaSource failed (msdia80.dll unregistered?), go to step 4.
> 4. Get a copy of msdia80.dll and put it in c:\Program Files\Common Files\Microsoft Shared\VC\.
> 5. As Administrator, run: regsvr32 c:\Program Files\Common Files\Microsoft Shared\VC\msdia80.dll.
>       * On success, retry step 3.
>       * If you get error 0x80004005, you did not run as Administrator.
* publish.bat

懒人的解决方案：

```bash
@echo off
rem 引用dump_sym.exe
set dump_syms="D:\dev_libs\google\breakpad\src\tools\windows\binaries\dump_syms.exe"

rem 需要使用7z进行压缩
set the_7z="..\7-Zip\7z.exe"

rem fciv是Windows发布的一个生成文件MD5值的小工具
set fciv="fciv.exe"

rem 需要使用curl上传文件
set curl="curl.exe"

rem 生成sym
%dump_syms% ..\..\Release\your-app.pdb > your-app.sym

rem 压缩
%the_7z% a publish.7z your-app.sym ..\ChangeLog.txt
del your-app.sym

rem 获取压缩文件MD5值
for /f %%i in ('%fciv% -md5 publish.7z') do set md5=%%i

rem 获取文件版本号
set /p pub_version=<..\..\Release\VersionNo.ini

rem 上传压缩文件至服务器
%curl% -F "action=upload" -F "md5=%md5%" -F "pub_version=%pub_version%" -F "publish=@publish.7z;type=application/7z" https://www.your-domain.com/crash/publish.php

del publish.7z
echo Done!
```

### <a name=gen_sym_linux></a>Linux服务器环境运行

* publish.php

```php
<?php

// Specify the directory where to save error reports
$file_root = "/var/www/html/crash/app/";

// This is to avoid PHP warning
date_default_timezone_set('UTC');

// Writes error code and text message and then exits
function done($return_status, $message)
{
    // Write HTTP responce code
    header("HTTP/1.0 ".$return_status." ".$message);
    // Write HTTP responce body (for backwards compatibility)
    echo $return_status." ".$message; 
    exit(0);
}

// Checks that text fild doesn't contain inacceptable symbols
function checkOK($field)
{
    if (stristr($field, "\\r") || stristr($field, "\\n")) 
    {
        done(450, "Invalid input parameter.");
    }
}

$md5_hash = "";    // MD5 hash for error report ZIP
$file_name = "";   // Destination file name                                  
$pub_version = "";  // Publish version

// Check that MD5 hash exists 
if(!isset($_POST['md5']))
{
    done(450, "MD5 hash is missing.");
}

// Get MD5 hash
$md5_hash = $_POST['md5'];
checkOK($md5_hash);
if(strlen($md5_hash)!=32)
{
    done(450, "MD5 hash value has wrong length.");
}

// Get CrashGUID
if(!array_key_exists("pub_version", $_POST))
{
    done(450, "Publish version missing.");  
}

$pub_version = $_POST["pub_version"];
checkOK($pub_version);
if(strlen($pub_version)==0)
{
    done(450, "Publish version has wrong length.");
}  

// Get file attachment
if(array_key_exists("publish", $_FILES))
{
    // Check upload error code
    $error_code = $_FILES["publish"]["error"];
    if($error_code!=0)
    {
        done(450, "File upload failed with code $error_code.");
    }

    // Get temporary name uploaded file is stored currently
    $tmp_file_name = $_FILES["publish"]["tmp_name"];
    checkOK($tmp_file_name);

    // Check that uploaded file data have correct MD5 hash
    $my_md5_hash = strtolower(md5_file($tmp_file_name));
    $their_md5_hash = strtolower($md5_hash);
    if($my_md5_hash!=$their_md5_hash)
    {
        done(451, "MD5 hash is invalid (yours is ".$their_md5_hash.", but mine is ".$my_md5_hash.")");
    }

    // Use crash GUID as file name
    $file_name = $file_root.$pub_version.".7z";

    // Move uploaded file to an appropriate directory
    if(!move_uploaded_file($tmp_file_name, $file_name))
    {
        done(452, "Couldn't save data to local storage"); 
    }

    // 调用auto_build.sh进行处理
    $output = shell_exec($file_root."auto_build.sh ".$file_name);
    done(200, $output);
}
else
{
  done(452, "File attachment missing"); 
}

// OK.
done(200, "Success.");

?>
```

* auto_build.sh 解压

```bash
#!/bin/bash
if [ -z "$1" ]; 
then echo Usage:$0 [7z file]
exit
fi

cd /var/www/html/crash/app
s=$1
s=${s##*/}
name=${s%.7z}
echo name=$name
if [ -d "$name" ]; then
echo This version of 7z file had already been extracted!
exit
fi

#mkdir $name
7z x $1 -o$name
chmod 755 $name
cd $name
../build_symbols.sh your-app.pdb

```

* build_symbols.sh 将sym文件按照breakpad指定的方式存放

```bash
#!/bin/bash
line=$(head -n1 your-app.sym)
echo line
arr=($line)
sdir="./symbols/your-app.pdb/${arr[3]}"
mkdir -p $sdir
mv your-app.sym $sdir
echo Build symbols OK
```

## <a name=integrate>应用程序植入breakpad

按照breakpad官方文档[Windows Integration overview](https://chromium.googlesource.com/breakpad/breakpad/+/master/docs/windows_client_integration.md)，应用程序中如此这般：

```c++
#include <client/windows/handler/exception_handler.h>
#ifdef _DEBUG
#pragma comment(lib, "client/windows/Debug/lib/common.lib")
#pragma comment(lib, "client/windows/Debug/lib/exception_handler.lib")
#pragma comment(lib, "client/windows/Debug/lib/crash_generation_client.lib")
#else
#pragma comment(lib, "client/windows/Release/lib/common.lib")
#pragma comment(lib, "client/windows/Release/lib/exception_handler.lib")
#pragma comment(lib, "client/windows/Release/lib/crash_generation_client.lib")
#endif // _DEBUG

bool ShowDumpResults(const wchar_t* dump_path,
    const wchar_t* minidump_id,
    void* context,
    EXCEPTION_POINTERS* exinfo,
    MDRawAssertionInfo* assertion,
    bool succeeded) {

    if (minidump_id) {
        auto crashguid = utf8::w2a(minidump_id);
        JLOG_INFO("minidump_id={}", crashguid);
        std::string cmd = get_exe_path_a() + "/tools/report.bat " + app_version + " " + crashguid;
        jlib::daemon(cmd, false, false);

        // 调用report.bat，参数为版本号、crushguid
    }

    return succeeded;
}

int main()
{
    auto handler = std::make_unique<ExceptionHandler>(
        dmp_path, // dump_path
        nullptr, // FilterCallback
        ShowDumpResults, // MinidumpCallback
        nullptr, // callback_context
        ExceptionHandler::HANDLER_ALL // handler_types
        );
}
```

应用程序安装后的部分文件结构为：

```txt
+-- your-app-installation-path/
    +-- 7-zip/
        +-- 7z.exe
        +-- 7z.dll
        +-- 7-zip.dll
    +-- your-app.exe
    +-- dumps/
        +-- here is the dmp files' location
    +-- tools/
        +-- curl.exe
        +-- curl-ca-bundle.crt
        +-- fciv.exe
        +-- libcurl.dll
        +-- report.bat
```

## <a name=report>处理应用程序崩溃
### <a name=report_win>Windows

* report.bat

```bash
@echo on
set the_7z="..\7-Zip\7z.exe"
set fciv="fciv.exe"
set curl="curl.exe"

set appversion=%1
set crashguid=%2
cd ..\dumps
copy %crashguid%.dmp crashdump.dmp
%the_7z% a -tzip %crashguid%.zip crashdump.dmp
for /f %%i in ('%fciv% -md5 %crashguid%.zip') do set md5=%%i
cd ..\tools
%curl% -F "action=upload" -F "md5=%md5%" -F "appversion=%appversion%" -F "crashguid=%crashguid%" -F "crashrpt=@..\dumps\%crashguid%.zip;type=application/zip" https://www.your-domain.com/crash/report.php

echo Done!
```

### <a name=report_linux>Linux

1. report.php

```php
<?php

// Specify the directory where to save error reports
$file_root = "/var/www/html/crash/";

// This is to avoid PHP warning
date_default_timezone_set('UTC');

// Writes error code and text message and then exits
function done($return_status, $message)
{
    // Write HTTP responce code
    header("HTTP/1.0 ".$return_status." ".$message);
    // Write HTTP responce body (for backwards compatibility)
    echo $return_status." ".$message; 
    exit(0);
}

// Checks that text fild doesn't contain inacceptable symbols
function checkOK($field)
{
    if (stristr($field, "\\r") || stristr($field, "\\n")) 
    {
        done(450, "Invalid input parameter.");
    }
}

$app_version = ""; // Application Version
$md5_hash = "";    // MD5 hash for error report ZIP
$file_name = "";   // Destination file name                                  
$crash_guid = "";  // Crash GUID

if(!isset($_POST['appversion']))
{
    done(450, "AppVersion is missing.");
}
$app_version = $_POST['appversion'];

// Check that MD5 hash exists 
if(!isset($_POST['md5']))
{
    done(450, "MD5 hash is missing.");
}

// Get MD5 hash
$md5_hash = $_POST['md5'];
checkOK($md5_hash);
if(strlen($md5_hash)!=32)
{
    done(450, "MD5 hash value has wrong length.");
}

// Get CrashGUID
if(!array_key_exists("crashguid", $_POST))
{
    done(450, "Crash GUID missing.");  
}

$crash_guid = $_POST["crashguid"];
checkOK($crash_guid);
if(strlen($crash_guid)!=36)
{
    done(450, "Crash GUID has wrong length.");
}  

// Get file attachment
if(array_key_exists("crashrpt", $_FILES))
{
    // Check upload error code
    $error_code = $_FILES["crashrpt"]["error"];
    if($error_code!=0)
    {
        done(450, "File upload failed with code $error_code.");
    }

    // Get temporary name uploaded file is stored currently
    $tmp_file_name = $_FILES["crashrpt"]["tmp_name"];
    checkOK($tmp_file_name);

    // Check that uploaded file data have correct MD5 hash
    $my_md5_hash = strtolower(md5_file($tmp_file_name));
    $their_md5_hash = strtolower($md5_hash);
    if($my_md5_hash!=$their_md5_hash)
    {
        done(451, "MD5 hash is invalid (yours is ".$their_md5_hash.", but mine is ".$my_md5_hash.")");
    }

    // Use crash GUID as file name 
    // /var/www/html/crash/report/$APPVERSION/$CRASHGUID.zip
    $file_name = $file_root."app/".$app_version."/".$crash_guid.".zip";

    // Move uploaded file to an appropriate directory
    if(!move_uploaded_file($tmp_file_name, $file_name))
    {
        done(452, "Couldn't save data to local storage:".$file_name); 
    }

    $output = shell_exec($file_root."app/process.sh ".$app_version." ".$crash_guid." ".$_SERVER['REMOTE_ADDR']);
    done(200, $output);
}
else
{
    done(452, "File attachment missing"); 
}

// OK.
done(200, "Success.");

?>
```

2. process.sh

```bash
#!/bin/bash
basedir=/var/www/html/crash/app
cd $basedir

if [ -z "$1" ] || [ -z "$2" ] || [ -z "$3" ];then
    echo Usage: $0 [app versin] [crash guid] [ip address]
    exit
fi

app_version=$1
crashguid=$2
ip_addr=$3
echo app_version=$app_version
echo crashguid=$crashguid
echo ip_addr=$ip_addr

if [ -d "$crashguid" ]; then
    echo This version of crashguid had already been extracted!
else
    7z x ${basedir}/$app_version/${crashguid}.zip -o${basedir}/$app_version/${crashguid}
    chmod 755 ${basedir}/$app_version/${crashguid}
fi

basedir=${basedir}/${app_version}/${crashguid}

# 这里使用了聚合的获取IP位置服务。吐个槽，干嘛都要实名注册！
# get address info by ip
ip_xml="${basedir}/ip.xml"
result_code=""
area=""
location=""
url="http://apis.juhe.cn/ip/ip2addr?ip="${ip_addr}"&dtype=xml&key=your-key"
echo url=$url
echo ip_xml=$ip_xml
curl $url > $ip_xml
result=$(cat $ip_xml)
echo result:$result

read_dom() {
    local IFS=\>
    read -d \< ENTITY CONTENT
    local RET=$?
    TAG_NAME=${ENTITY%% *}
    ATTRIBUTES=${ENTITY#* }
    return $RET
}

parse_dom_ip() {
    if [[ $TAG_NAME = "resultcode" ]]; then
        result_code=$CONTENT
        elif [[ $TAG_NAME = "area" ]]; then
        area=$CONTENT
        elif [[ $TAG_NAME = "location" ]]; then
        location=$CONTENT
    fi
}

while read_dom; do
    parse_dom_ip
done < "$ip_xml"
rm $ip_xml

if [ -z "$result_code" ] || [ "$result_code" != "200" ]; then
    echo Get address for ip $ip_addr failed!
    elif [ "$result_code" = "200" ]; then
    echo Get address for ip $ip_addr success! Area: $area, Location: $location
fi

./send_mail_.sh $app_version $crashguid $ip_addr $area $location

rm $ip_xml
echo Done!
```

3. send_mail_.sh

```bash
#!/bin/bash
basedir=/var/www/html/crash/app
to_user_file="${basedir}/mail_to.txt"
dmp2html="${basedir}/dmp2html.sh"

subject=`echo 应用程序崩溃报告`
preifx=`echo "<html lang='zh_CN'><head><meta charset='utf8'/></head><body>"`
time=`date +"%Y年%m月%d日 %H:%M:%S"`
message=""

app_version=$1
crash_guid=$2
ip_addr=$3
area=$4
location=$5

# build ip info
ip_info="<p>Client IP:"${ip_addr}"<br/>Area:"${area}"<br/>Location:"${location}"</br></p>"
echo ip_info=$ip_info

# build stack walk info
dump=${basedir}/${app_version}/${crash_guid}/crashdump.dmp
symbols=${basedir}/${app_version}/symbols
crash_dump=`minidump_stackwalk $dump $symbols &> ${dump}.txt`
crash_dump=`${dmp2html} ${dump}.txt`
rm ${dump}.txt

# build message
message=`echo "<br/>Crash Dump:<br/>"${ip_info}"<br/>"${crash_dump}"<br/></body></html>"`
message=${preifx}${time}${message}
echo 'message=' ${message}
echo $message > message.html

#send mail
while IFS= read -r line
do
    echo "Sending email to $line"
    (
        echo "From: admin";
        echo "To: $line";
        echo "Subject: ${subject}";
        echo "Content-Type: text/html";
        echo "MIME-Version: 1.0";
        echo "";
        echo "${message}";
    ) | /usr/sbin/sendmail -t
    echo "Done!"
done <"$to_user_file"

```

### <a name=mail></a>邮件示例

以下是收到的邮件部分内容：

```html
2017年08月16日 23:43:26
Crash Dump:
Client IP:113.140.*.*
Area:陕西省西安市
Location:电信
...

2017-08-16 23:43:26: minidump.cc:4811: INFO: Minidump opened minidump /var/www/html/crash/app/1.5.42.12672/51c3e708-4c60-4334-9faf-f7f1fb9523f9/crashdump.dmp
2017-08-16 23:43:26: minidump.cc:4931: INFO: Minidump not byte-swapping minidump
2017-08-16 23:43:26: minidump.cc:5414: INFO: GetStream: type 1197932546 not present
...

Operating system: Windows NT
10.0.14393 
CPU: x86
GenuineIntel family 6 model 60 stepping 3
4 CPUs

GPU: UNKNOWN

Crash reason: EXCEPTION_ACCESS_VIOLATION_WRITE
Crash address: 0x0
Process uptime: 1 seconds

Thread 0 (crashed)
0 your-app.exe!CLoginDlg::OnBnClickedOk() [logindlg.cpp : 71 + 0x0]
eip = 0x00b60ef0 esp = 0x014fb608 ebp = 0x014fb64c ebx = 0x014ff4c0
esi = 0x014ff4c0 edi = 0x00000000 eax = 0x014fb640 ecx = 0x014ff4c0
edx = 0x00000000 efl = 0x00010286
Found by: given as instruction pointer in context
1 your-app.exe!_AfxDispatchCmdMsg(CCmdTarget *,unsigned int,int,void ( CCmdTarget::*)(void),void *,unsigned int,AFX_CMDHANDLERINFO *) [cmdtarg.cpp : 77 + 0x5]
eip = 0x00c64503 esp = 0x014fb654 ebp = 0x014fb660
Found by: call frame info
2 your-app.exe!CCmdTarget::OnCmdMsg(unsigned int,int,void *,AFX_CMDHANDLERINFO *) [cmdtarg.cpp : 372 + 0x18]
eip = 0x00c64324 esp = 0x014fb668 ebp = 0x014fb698
Found by: call frame info
...
```

可以清楚的看到崩溃发生在了logindlg.cpp文件的71行，函数为CLoginDlg::OnBnClickedOk。

# <a name=refs></a>参考资料

* [breakpad](https://chromium.googlesource.com/breakpad/breakpad)
* [How to build google google-breakpad for windows?](https://stackoverflow.com/questions/3618721/how-to-build-google-google-breakpad-for-windows)
* [Windows Integration overview](https://chromium.googlesource.com/breakpad/breakpad/+/master/docs/windows_client_integration.md)
* [Decoding Crash Dumps](https://www.chromium.org/developers/decoding-crash-dumps)
