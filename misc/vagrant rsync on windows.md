# vagrant rsync on windows
#####起因
```
1. 缺少 rsync
   解决方法是 安装cygwin
2. /cygdrive/d/xx 
   - 解决方法是使用 cygwin 的 Terminal 进入后运行 vagrant up
   - 如果你用惯了 cmder, 你可以参考 cmder with cygwin 的方法来整合 cmder 和 cygwin


```

#####安装cygwin
```txt
1. 下载 http://www.cygwin.com/setup-x86.exe 并运行, 如果你有代理，可以设置代理(速度或许能快点)
2. 在线安装,或者离线安装,所需的 cygwin.iso 下载地址为
3. 选择安装 rsync (搜索 rsync, categories 为 Net)
4. 安装 apt-cyg。从 https://github.com/transcode-open/apt-cyg 下载 apt-cyg 的脚本放到 cygwin 的 bin 目录即可
5. apt-cyg install wget



```

#####cmder with cygwin
```
# cmder 设置
# settings... > Startup > Tasks 添加一项 Task
name: Cygwin
icon: /icon D:&#92;Workspace&#92;workspace.env&#92;cygwin&#92;Cygwin-Terminal.ico
cmds: D:&#92;Workspace&#92;workspace.env&#92;cygwin&#92;cygwin.bat -c "/bin/xhere /bin/bash.exe '%V'"

# cmder 添加 alias,修改文件 cmder&#92;config&#92;user-aliases.cmd
# 方便 cmder 切换到 cygwin terminal, 通过 cyg 切换到 cygwin terminal
cyg=D:&#92;Workspace&#92;workspace.env&#92;cygwin&#92;cygwin.bat

# D:&#92;Workspace&#92;workspace.env&#92;cygwin 为 cygwin 的安装目录
# 在 <cygwin-home>&#92;home&#92;arashicage&#92;.bash_profile 里添加
@echo off 后面
set HOME=D:&#92;Workspace&#92;workspace.env&#92;cygwin&#92;home&#92;arashicage&#92;

# 修改 D:&#92;Workspace&#92;workspace.env&#92;cygwin&#92;home&#92;arashicage&#92;.bash_profile 里添加 alias
# 这两个 alias 是为了方便 vagrant 的相关操作
alias vbox='cd /cygdrive/D/Workspace/workspace.vagrant'
alias ll='ls -la'

# vagrant 虚机启动后, 使用下面命令监控同步目录和自动同步
vagrant rsync-auto



```

#####Vagrant 文件
```
# 在 Vagrant.configure("2") 下一行添加如下一行 
# ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"

Vagrant.configure("2") do |config|
    ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"
    
# 对于需要设定 sync folder 的 vm, 设置
config.vm.synced_folder "F:/ansible/demo", "/vagrant", type: "rsync", rsync__auto: "true"



```

#####其他参考
```
- http://blog.sina.com.cn/s/blog_9d30c8430100y6yo.html cygwin 环境变量设置
- http://www.tuicool.com/articles/2MramqI cygwin 环境变量设置


```
