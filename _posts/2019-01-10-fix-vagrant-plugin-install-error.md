---
layout: post
title:  "修复 vagrant plugin install 失败问题"
subtitle: "智障还是机智？"
date:   "2019-01-10"
author: "cj"
tags:
    vagrant
    virtual-box
    laravel
    homestead
---

# 修复 vagrant plugin install 失败问题

由于最近重装了系统，需要重新搭建homestead+laravel环境。安装Virutal Box 6.0.0, Vagrant 2.2.2, 然后执行`vagrant plugin install vagrant-winnfsd`时报错：

```txt
Installing the 'vagrant-winnfsd' plugin. This can take a few minutes...
C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:834:in `connect':
 The requested address is not valid in its context. - connect(2) for "0.0.0.0" p
ort 53 (Errno::EADDRNOTAVAIL)
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:834:
in `block in lazy_initialize'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:826:
in `synchronize'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:826:
in `lazy_initialize'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:846:
in `sender'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:525:
in `block in fetch_resource'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:1133
:in `block (3 levels) in resolv'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:1131
:in `each'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:1131
:in `block (2 levels) in resolv'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:1130
:in `each'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:1130
:in `block in resolv'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:1128
:in `each'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:1128
:in `resolv'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:520:
in `fetch_resource'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:510:
in `each_resource'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/resolv.rb:491:
in `getresource'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/rubygems/remot
e_fetcher.rb:105:in `api_endpoint'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/rubygems/sourc
e.rb:47:in `api_uri'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/rubygems/sourc
e.rb:183:in `load_specs'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/bundler.rb:424:in `block in validate_configured_sources!'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/rubygems/sourc
e_list.rb:98:in `each'
        from C:/HashiCorp/Vagrant/embedded/mingw64/lib/ruby/2.4.0/rubygems/sourc
e_list.rb:98:in `each_source'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/bundler.rb:422:in `validate_configured_sources!'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/bundler.rb:330:in `internal_install'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/bundler.rb:128:in `install'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/plugin/manager.rb:138:in `block in install_plugin'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/plugin/manager.rb:148:in `install_plugin'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/plugins
/commands/plugin/action/install_gem.rb:30:in `call'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/action/warden.rb:34:in `call'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/action/builder.rb:116:in `call'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/action/runner.rb:66:in `block in run'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/util/busy.rb:19:in `busy'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/action/runner.rb:66:in `run'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/plugins
/commands/plugin/command/base.rb:14:in `action'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/plugins
/commands/plugin/command/install.rb:70:in `block in execute'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/plugins
/commands/plugin/command/install.rb:69:in `each'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/plugins
/commands/plugin/command/install.rb:69:in `execute'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/plugins
/commands/plugin/command/root.rb:66:in `execute'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/cli.rb:54:in `execute'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/lib/vag
rant/environment.rb:291:in `cli'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.5/gems/vagrant-2.1.5/bin/vag
rant:164:in `<main>'
```

搜索`The requested address is not valid in its context. connect(2) for "0.0.0.0" port 53 (Errno::EADDRNOTAVAIL)`，看这篇文章[vagrant up fail in 1.9.3 on windows10](https://github.com/hashicorp/vagrant/issues/8395)，但这是已经添加box后的操作了，没什么帮助。

既然说连不上`0.0.0.0:53`，看下网络状态：
`netstat -ano`，
有一行

`UDP    0.0.0.0:53             *:*                                    10144`

说明是有程序在监听这个UDP端口的。思路中断，看其他文章。

再看这篇[Installing gem error (EADDRNOTAVAIL)](https://stackoverflow.com/questions/35202153/installing-gem-error-eaddrnotavail)，有网友说是DNS出问题了。

又看到这一篇[ERROR: While executing gem ... (Errno::EADDRNOTAVAIL)The requested address is not valid in its context. - connect(2) for "0.0.0.0" port 53](https://groups.google.com/forum/#!topic/rubyinstaller/iBYTkP28C_0)，作者说：

>Looks like your DNS resolver address is set to 0.0.0.0, so that the name resolution fails. Check your network settings!

蛤？

想到以前启用HyperV虚拟机、配置网络时会生成一个新的虚拟网卡 `vEthernet (New Virtual Switch)`，大部分 IPv4 设置沿用之前的网卡设置，但是 DNS 一栏会给置空。

我最近这次重装系统后也启用过 HyperV 一阵子了，可以正常上网就忘掉有这回事，或许是这个问题？打开一看果然，DNS 栏空空如也，填写两个 DNS server： `192.168.1.1`, `114.114.114.114`，再次运行 `vagrant plugin install vagrant-winnfsd`，终于成功：

```cmd
C:\Users\capta>vagrant plugin install vagrant-winnfsd
Installing the 'vagrant-winnfsd' plugin. This can take a few minutes...
Fetching: vagrant-winnfsd-1.4.0.gem (100%)
Installed the plugin 'vagrant-winnfsd (1.4.0)'!
```

这算什么破问题，搞了两个下午。。。

用过virtual box 6.0.0 和 5.2.22，Vagrant 2.2.2 和 2.1.5，结果他们都没问题，是我系统环境有问题！ 


---

>2019年1月11日00:07:30 增补

卧槽vbox和hyperv不能共存。。。那我重装系统之前咋用的？噢，很久以前玩过hyperv后来关了，所以上次搭建homestead环境没出问题。然后再启用hyperv，毕竟微软自家儿子，还可以正常用，但是！这段时间直到我重装系统，我都没有再用过vbox了！。。。

参考[hyper-v disables vt-x for other hypervisors](https://social.technet.microsoft.com/Forums/windows/en-US/118561b9-7155-46e3-a874-6a38b35c67fd/hyperv-disables-vtx-for-other-hypervisors?forum=w8itprogeneral) 和 [Turning hyper-v on and off](https://blogs.technet.microsoft.com/gmarchetti/2008/12/07/turning-hyper-v-on-and-off/)，设置机器启动时是否开启hyperv：

* 开启 `bcdedit /set hypervisorlaunchtype auto start`
* 关闭 `bcdedit /set hypervisorlaunchtype off`