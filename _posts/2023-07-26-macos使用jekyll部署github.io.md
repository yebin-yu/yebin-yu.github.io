---
layout:     post
title:      macos使用jekyll部署github.io
subtitle:   macos上部署jekyll太多坑了。。。
date:       2023-07-26
author:     yebin-yu
header-img: img/post-bg-debug.png
catalog: false
tags:
    - 其他
---

# gem 配置源

需要检查一下 `gem` 的源

```shell
yebinyu@192 ~ % gem source -l
*** CURRENT SOURCES ***

https://rubygems.org/
```

增加新的地址，删除原来的地址（国内连接原来的源太慢了。。）

```shell
yebinyu@192 ~ % gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
https://gems.ruby-china.com/ added to sources
https://rubygems.org/ removed from sources
```



# 检查 gem 和 ruby 的版本

检查 `gem` 和 `ruby` 的版本

```shell
yebinyu@192 ~ % gem -v
3.0.3.1
```

如果 `gem` 的版本过低，需要升级。

> 如果显示没有permission，需要在前面加个 `sudo` ，意思是用管理员权限运行。

```shell
yebinyu@192 ~ % gem update --system
Updating rubygems-update
Fetching rubygems-update-3.4.17.gem
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.
yebinyu@192 ~ % sudo gem update --system 
Password:
Updating rubygems-update
Fetching rubygems-update-3.4.17.gem
Successfully installed rubygems-update-3.4.17
...
```



# 安装jekyll

```shell
gem install jekyll
gem install jekyll-paginate
```

检查jekyll是否安装成功

```shell
jekyll -v
```

创建项目

```shell
jekyll new myblog
```

进入项目，运行

```shell
cd myblog
jekyll serve
```





# 报错解决

### `gem install jekyll` 失败

安装的时候报错

```shell
yebinyu@192 ~ % sudo gem install jekyll                         
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0 directory.
	/Library/Ruby/Site/2.6.0/rubygems/installer.rb:714:in `verify_gem_home'
	/Library/Ruby/Site/2.6.0/rubygems/installer.rb:904:in `pre_install_checks'
	/Library/Ruby/Site/2.6.0/rubygems/installer.rb:302:in `install'
	/Library/Ruby/Site/2.6.0/rubygems/resolver/specification.rb:104:in `install'
	/Library/Ruby/Site/2.6.0/rubygems/request_set.rb:194:in `block in install'
	/Library/Ruby/Site/2.6.0/rubygems/request_set.rb:182:in `each'
	/Library/Ruby/Site/2.6.0/rubygems/request_set.rb:182:in `install'
	/Library/Ruby/Site/2.6.0/rubygems/commands/install_command.rb:214:in `install_gem'
	/Library/Ruby/Site/2.6.0/rubygems/commands/install_command.rb:230:in `block in install_gems'
	/Library/Ruby/Site/2.6.0/rubygems/commands/install_command.rb:223:in `each'
	/Library/Ruby/Site/2.6.0/rubygems/commands/install_command.rb:223:in `install_gems'
	/Library/Ruby/Site/2.6.0/rubygems/commands/install_command.rb:169:in `execute'
	/Library/Ruby/Site/2.6.0/rubygems/command.rb:327:in `invoke_with_build_args'
	/Library/Ruby/Site/2.6.0/rubygems/command_manager.rb:252:in `invoke_command'
	/Library/Ruby/Site/2.6.0/rubygems/command_manager.rb:192:in `process_args'
	/Library/Ruby/Site/2.6.0/rubygems/command_manager.rb:150:in `run'
	/Library/Ruby/Site/2.6.0/rubygems/gem_runner.rb:51:in `run'
	/usr/bin/gem:21:in `<main>'
```

 先用`sudo -i` 进入 `root` 环境，还是报错

```shell
yebinyu@192 ~ % sudo -i
192:~ root# gem install jekyll
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0 directory.
	/Library/Ruby/Site/2.6.0/rubygems/installer.rb:714:in `verify_gem_home'
	/Library/Ruby/Site/2.6.0/rubygems/installer.rb:904:in `pre_install_checks'
	/Library/Ruby/Site/2.6.0/rubygems/installer.rb:302:in `install'
	/Library/Ruby/Site/2.6.0/rubygems/resolver/specification.rb:104:in `install'
	/Library/Ruby/Site/2.6.0/rubygems/request_set.rb:194:in `block in install'
	/Library/Ruby/Site/2.6.0/rubygems/request_set.rb:182:in `each'
	/Library/Ruby/Site/2.6.0/rubygems/request_set.rb:182:in `install'
	/Library/Ruby/Site/2.6.0/rubygems/commands/install_command.rb:214:in `install_gem'
	...
```

找了很久资料，后面发现原因是：gem 要往某个神奇的目录写文件但是你的权限不够. 因为你使用的是 Apple 家自带的 ruby, 在尝试往 Apple 自家的库中塞东西, 默认那个位置是给 root 的.

所以需要安装homebrew后，用homebrew再去下安装一套gem，这样就能和mac自身的mac区分开来了。



### 安装 homebrew 后安装新的 ruby

```shell
# 这个是官网给的，国内下载一直卡着不动
/bin/bash -c"$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

# 用了这个解决的
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"

# 安装之后安装ruby
/opt/homebrew/bin/brew install ruby
```

这时候还没用到新的路径

```shell
# 但安装了之后发现ruby还是用的原来的，这里就需要更新一下搜索路径
yebinyu@Yebindebijibendiannao / % which ruby
/usr/bin/ruby
```

设置到环境变量中

```shell
# 安装后将新增的地址放到环境变量中
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc

# 使配置的路径当前就可用
PATH="/opt/homebrew/opt/ruby/bin:$PATH"
```

这样就ok了

```shell
yebinyu@Yebindebijibendiannao / % which ruby
/opt/homebrew/opt/ruby/bin/ruby
```

再重新安装一下bundle

```shell
gem install tmbundle-manager
```



### 安装成功后找不到 jekyll

```shell
yebinyu@Yebindebijibendiannao / % gem install jekyll
Successfully installed jekyll-4.3.2
Parsing documentation for jekyll-4.3.2
Done installing documentation for jekyll after 0 seconds
1 gem installed
yebinyu@Yebindebijibendiannao / % 
yebinyu@Yebindebijibendiannao / % jekyll -v
zsh: command not found: jekyll
yebinyu@Yebindebijibendiannao / % 
yebinyu@Yebindebijibendiannao / % which jekyll
jekyll not found
```

还是因为二进制工具找不到，解决：

```shell
export PATH="/opt/homebrew/lib/ruby/gems/3.2.0/bin:$PATH"
echo 'export PATH="/opt/homebrew/lib/ruby/gems/3.2.0/bin:$PATH"' >> ~/.zshrc
```



### `jekyll new myblog` 失败

```shell
yebinyu@Yebindebijibendiannao yebin-yu.github.io % jekyll new myblog
Running bundle install in /Users/yebinyu/Documents/0_CODE/1_github.io/yebin-yu.github.io/myblog... 

  Bundler: Fetching gem metadata from https://rubygems.org/............
  Bundler: Resolving dependencies...
  Bundler: Fetching concurrent-ruby 1.2.2
  Bundler: Fetching eventmachine 1.2.7
  Bundler: Fetching http_parser.rb 0.8.0
  Bundler: Fetching ffi 1.15.5
  Bundler: Fetching google-protobuf 3.23.4 (arm64-darwin)
  Bundler: Fetching rb-fsevent 0.11.2
  Bundler: Fetching public_suffix 5.0.3
  Bundler: Fetching colorator 1.1.0/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/universal-darwin23/rbconfig.rb:21: warning: Insecure world writable dir /opt/homebrew/lib in PATH, mode 040777
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/concurrent-ruby-1.2.2.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/concurrent-ruby-1.2.2.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/http_parser.rb-0.8.0.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/http_parser.rb-0.8.0.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/eventmachine-1.2.7.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/eventmachine-1.2.7.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/concurrent-ruby-1.2.2.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/concurrent-ruby-1.2.2.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/http_parser.rb-0.8.0.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/http_parser.rb-0.8.0.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/eventmachine-1.2.7.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/eventmachine-1.2.7.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/http_parser.rb-0.8.0.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/http_parser.rb-0.8.0.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/google-protobuf-3.23.4-arm64-darwin.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/google-protobuf-3.23.4-arm64-darwin.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/concurrent-ruby-1.2.2.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/concurrent-ruby-1.2.2.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/eventmachine-1.2.7.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/eventmachine-1.2.7.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (2/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/ffi-1.15.5.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/ffi-1.15.5.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/google-protobuf-3.23.4-arm64-darwin.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/google-protobuf-3.23.4-arm64-darwin.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (3/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/ffi-1.15.5.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/ffi-1.15.5.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/google-protobuf-3.23.4-arm64-darwin.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/google-protobuf-3.23.4-arm64-darwin.gem
  Bundler: Retrying download gem from https://rubygems.org/ due to error (4/4): Bundler::GenericSystemCallError There was an error accessing `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/ffi-1.15.5.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @ rb_sysopen - /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/ffi-1.15.5.gem
  Bundler: Bundler::GenericSystemCallError: There was an error accessing
  Bundler: `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @
  Bundler: rb_sysopen -
  Bundler: /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/public_suffix-5.0.3.gem
  Bundler: /Library/Ruby/Site/2.6.0/bundler/shared_helpers.rb:117:in `rescue in
  Bundler: filesystem_access'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/shared_helpers.rb:102:in `filesystem_access'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/rubygems_integration.rb:483:in `block in
  Bundler: download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/retry.rb:40:in `run'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/retry.rb:30:in `attempt'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/rubygems_integration.rb:474:in `download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:484:in `download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:446:in `fetch_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:430:in
  Bundler: `fetch_gem_if_possible'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:160:in `install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/gem_installer.rb:54:in `install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/gem_installer.rb:16:in
  Bundler: `install_from_spec'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/parallel_installer.rb:156:in
  Bundler: `do_install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/parallel_installer.rb:147:in `block
  Bundler: in worker_pool'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:62:in `apply_func'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:57:in `block in process_queue'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:54:in `loop'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:54:in `process_queue'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:90:in `block (2 levels) in
  Bundler: create_threads'
  Bundler: 
  Bundler: An error occurred while installing public_suffix (5.0.3), and Bundler cannot
  Bundler: continue.
  Bundler: 
  Bundler: In Gemfile:
  Bundler: minima was resolved to 2.5.1, which depends on
  Bundler: jekyll-feed was resolved to 0.17.0, which depends on
  Bundler: jekyll was resolved to 4.3.2, which depends on
  Bundler: addressable was resolved to 2.8.4, which depends on
  Bundler: public_suffix
  Bundler: 
  Bundler: 
  Bundler: Bundler::GenericSystemCallError: There was an error accessing
  Bundler: `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @
  Bundler: rb_sysopen -
  Bundler: /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/colorator-1.1.0.gem
  Bundler: /Library/Ruby/Site/2.6.0/bundler/shared_helpers.rb:117:in `rescue in
  Bundler: filesystem_access'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/shared_helpers.rb:102:in `filesystem_access'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/rubygems_integration.rb:483:in `block in
  Bundler: download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/retry.rb:40:in `run'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/retry.rb:30:in `attempt'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/rubygems_integration.rb:474:in `download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:484:in `download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:446:in `fetch_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:430:in
  Bundler: `fetch_gem_if_possible'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:160:in `install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/gem_installer.rb:54:in `install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/gem_installer.rb:16:in
  Bundler: `install_from_spec'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/parallel_installer.rb:156:in
  Bundler: `do_install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/parallel_installer.rb:147:in `block
  Bundler: in worker_pool'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:62:in `apply_func'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:57:in `block in process_queue'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:54:in `loop'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:54:in `process_queue'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:90:in `block (2 levels) in
  Bundler: create_threads'
  Bundler: 
  Bundler: An error occurred while installing colorator (1.1.0), and Bundler cannot
  Bundler: continue.
  Bundler: 
  Bundler: In Gemfile:
  Bundler: minima was resolved to 2.5.1, which depends on
  Bundler: jekyll-feed was resolved to 0.17.0, which depends on
  Bundler: jekyll was resolved to 4.3.2, which depends on
  Bundler: colorator
  Bundler: 
  Bundler: 
  Bundler: Bundler::GenericSystemCallError: There was an error accessing
  Bundler: `/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem`.
  Bundler: The underlying system error is Errno::EPERM: Operation not permitted @
  Bundler: rb_sysopen -
  Bundler: /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/gems/2.6.0/cache/rb-fsevent-0.11.2.gem
  Bundler: /Library/Ruby/Site/2.6.0/bundler/shared_helpers.rb:117:in `rescue in
  Bundler: filesystem_access'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/shared_helpers.rb:102:in `filesystem_access'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/rubygems_integration.rb:483:in `block in
  Bundler: download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/retry.rb:40:in `run'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/retry.rb:30:in `attempt'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/rubygems_integration.rb:474:in `download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:484:in `download_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:446:in `fetch_gem'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:430:in
  Bundler: `fetch_gem_if_possible'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/source/rubygems.rb:160:in `install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/gem_installer.rb:54:in `install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/gem_installer.rb:16:in
  Bundler: `install_from_spec'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/parallel_installer.rb:156:in
  Bundler: `do_install'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/installer/parallel_installer.rb:147:in `block
  Bundler: in worker_pool'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:62:in `apply_func'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:57:in `block in process_queue'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:54:in `loop'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:54:in `process_queue'
  Bundler: /Library/Ruby/Site/2.6.0/bundler/worker.rb:90:in `block (2 levels) in
  Bundler: create_threads'
  Bundler: 
  Bundler: An error occurred while installing rb-fsevent (0.11.2), and Bundler cannot
  Bundler: continue.
  Bundler: 
  Bundler: In Gemfile:
  Bundler: minima was resolved to 2.5.1, which depends on
  Bundler: jekyll-feed was resolved to 0.17.0, which depends on
  Bundler: jekyll was resolved to 4.3.2, which depends on
  Bundler: jekyll-watch was resolved to 2.2.1, which depends on
  Bundler: listen was resolved to 3.8.0, which depends on
  Bundler: rb-fsevent
```

问题是缺少依赖，用bundle去下载依赖，解决：

```shell
### 安装bundle
cd myblog
gem install bundle

### 这一步是用来创建bundle根目录的，否则install会报错
bundle init
bundle install
```





