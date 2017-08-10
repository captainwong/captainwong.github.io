---
layout: post
title:  "使用Google开源库breadpad实现错误报告功能"
subtitle: "Google大法好"
date:   "2017-08-10" 
author: "cj"
tags:
    breadpad
    c++
    linux
    google
    bugreport
---



前阵子写的微信公众号后台服务器自动崩溃重启了一次，看日志没任何头绪，看来需要core dump。
但是搜索一阵子发现，这玩意真难用，要`ulimit -c unlimited`后才会生成dump。又搜索一番，发现Google出品的breadpad，谷歌出品，必属精品，就它了！

* 1. 按照[官方教程](https://chromium.googlesource.com/breakpad/breakpad)，下载源码

```
git clone https://chromium.googlesource.com/breakpad/breakpad
```

* 2. 有几个第三方库默认情况没有下载，按需手动下载到breakpad/src/thirdparty中

```
cd breakpad
git clone https://chromium.googlesource.com/linux-syscall-support src/third_party/lss
```

* 3. 编译安装

```
./configure
make
make check
sudo make install
```

安装目录默认为/usr/local/include/breakpad，库目录/usr/local/lib/libbreakpad.a, libbreakpad_client.a

* 4. 应用

方便起见，写了一个自动生成symbol调试信息的脚本build_symbols.sh:

```
#!/bin/bash
out=$1
sym="${out}.sym"
dump_syms $out > $sym
line=$(head -n1 ${sym})
arr=($line)
sdir="./symbols/${out}/${arr[3]}"
mkdir -p $sdir
mv $sym $sdir
```

用例./build_symbols.sh test

我把它写在了makefile中的make release段内。


崩溃后自动调用写好的脚本生成stack walk并发送邮件给自己的邮箱

```
#include <client/linux/handler/exception_handler.h>

void crash_send(const std::string& dmp_path)
{
	try {
		auto path = bfs::canonical("../tools/send_mail_.sh");
		if (bfs::exists(path)) {
			std::string cmd = path.string() + " server_crash " + dmp_path + " &";
			printf("cmd=%s\n", cmd.c_str());
			int ret = std::system(cmd.c_str());
			printf("ret=%d\n", ret);
		}
	} catch (bfs::filesystem_error& e) {
		printf("%s\n", e.what());
	} catch (std::exception& e) {
		printf("%s\n", e.what());
	}
}

static bool dumpCallback(const google_breakpad::MinidumpDescriptor& descriptor,
						 void* context, bool succeeded)
{
	printf("Dump path: %s\n", descriptor.path());
	crash_send(descriptor.path());
	return succeeded;
}

int main()
{
	google_breakpad::MinidumpDescriptor descriptor("/tmp");
	google_breakpad::ExceptionHandler eh(descriptor, nullptr, dumpCallback, nullptr, true, -1);
}
```

以下是send_mail_.sh部分代码：

```
#!/bin/bash
basedir=/home/ubuntu/wechat_server/tools
#basedir=/home/jack/projects/test
to_user_file="${basedir}/mail_to.txt"
crashed="${basedir}/crashed.sh"
dmp2html="${basedir}/dmp2html.sh"
subject=""
preifx=`echo "<html lang='zh_CN'><head><meta charset='utf8'/></head><body>"`
time=`date +"%Y年%m月%d日 %H:%M:%S"`

param=$1
domain=$2
message=""

if [ "${param}" = "server_stop" ]; then
    subject=`echo 微信后台重启`
    message=`echo "<br/>微信公众号后台服务器已停止运行，正在重启</body></html>"`
elif [ "$param" = "server_crash" ]; then
    subject=`echo 微信后台崩溃`
    crash_dump=`${crashed} $2 "${basedir}/symbols"`
    crash_dump=`${dmp2html} "${2}.txt"`
    message=`echo "<br/>Crash Dump:<br/>"${crash_dump}"<br/></body></html>"`
fi

message=${preifx}${time}${message}
echo 'message=' ${message}
#exit
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

如代码所示，当传入参数为server_crash、dump_path时，使用dmp收集脚本crashed.sh生成stackwalk信息并保存为文件，再使用dmp2html.sh将文件的换行替换为`<br/>`，然后就发送邮件。

crashed.sh内容：

```
#!/bin/bash
dmp_path=$1
sym_path=$2
minidump_stackwalk $dmp_path $sym_path &> ${dmp_path}.txt
```

dmp2html.sh内容：

```
#!/bin/bash
dmp=$1
while IFS= read -r line
do
   echo "${line}<br/>"
done <"$dmp"

```

以下是收到的邮件部分内容：

```
2017年08月09日 21:57:06
Crash Dump:
2017-08-09 21:57:06: minidump.cc:4811: INFO: Minidump opened minidump /tmp/5f4b59c2-19b1-43e2-5fec0e9c-76809ea0.dmp

...

Operating system: Linux
0.0.0 Linux 4.4.0-87-generic #110-Ubuntu SMP Tue Jul 18 12:55:35 UTC 2017 x86_64
CPU: amd64
family 16 model 6 stepping 2
2 CPUs

GPU: UNKNOWN

Crash reason: SIGABRT
Crash address: 0x43c7
Process uptime: not available

Thread 0 (crashed)
0 libc-2.23.so + 0x35428
rax = 0x0000000000000000 rdx = 0x0000000000000006
rcx = 0x00007eff4892a428 rbx = 0x00007eff4b022000
rsi = 0x00000000000043c7 rdi = 0x00000000000043c7
rbp = 0x00000000005e74de rsp = 0x00007fffe172e378
r8 = 0xfefefefefefefeff r9 = 0x0000000000000001
r10 = 0x0000000000000008 r11 = 0x0000000000000202
r12 = 0x00000000000001a4 r13 = 0x00000 000005e8bc0
r14 = 0x0000000000000001 r15 = 0x0000000000000000
rip = 0x00007eff4892a428
Found by: given as instruction pointer in context

...

17 libc-2.23.so + 0x2dc82
rsp = 0x00007fffe172e500 rip = 0x00007eff48922c82
Found by: stack scanning
18 wechat_server!wechat::open::openapi::get_template_msg_id_for_auther(std::shared_ptr const&, std::__cxx11::basic_string, std::allocator >&) [open.cpp : 420 + 0x46]
rsp = 0x00007fffe172e530 rip = 0x00000000005a36c2
Found by: stack scanning
19 wechat_server!_fini + 0x29178
rsp = 0x00007fffe172e558 rip = 0x00000000005e733c
Found by: stack scanning
```

可以清楚的看到崩溃发生在了open.cpp文件的420行，函数为get_template_msg_id_for_auther。

至于Linux发送邮件的配置，可以参考[Ubuntu下部署MediaWiKi小记](http://wangyapeng.me/2017/05/14/unbuntu-setup-mediawiki/)。

参考资料：

* [breakpad](https://chromium.googlesource.com/breakpad/breakpad)
* [compile error: fatal error: third_party/lss/linux_syscall_support.h: No such file or directory](https://bugs.chromium.org/p/google-breakpad/issues/detail?id=541)
* [How To Add Breakpad To Your Linux Application](https://chromium.googlesource.com/breakpad/breakpad/+/master/docs/linux_starter_guide.md)




