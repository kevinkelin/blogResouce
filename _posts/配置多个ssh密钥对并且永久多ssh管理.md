title: 配置多个ssh密钥对并且永久多ssh管理
comment: true
categories:
  - web
date: 2018-06-16 23:53:38
tags: web
permalink: config-local-ssh

---

这两天捣腾SSH，一直对其使用一知半解，由于要把博客迁移，弄来弄去发现还是部署到国内的coding吧
之前也弄过，但是由于重新安装了git-for-windows客户端，所以一开始用hexo d命令部署的时候报错了
趁着这次迁移也好好弄了一下本地的ssh管理，虽然还有些问题，但是至少比之前清晰一些了，这里也记录一下过程中遇到的问题

<!-- more -->

我的目的，将hexo生成的静态文件同时部署到github与coding上
## 安装git-for-windows客户端
下载地址 https://git-scm.com/download/win 
无论是github还是coding都需要上传你的公钥，这两个地方可以上传相同的公钥，但也可以像我这样闲的蛋疼上传不同的公钥。

## 创建密钥对
使用`ssh-keygen` 来创建密钥对，命令为 `ssh-keygen -t rsa -C "your_email@example.com"`,其实这里-C 后面的email地址无所谓，可以随便写，只是为了你方便而已。
输入完该命令以后，首先会让你给密钥文件起个名字，这个是文件名，叫什么随心情
![生成密钥对](/image/2018/usessh1.png)
接下来让你输入密码，我这里直接回车，不用密码
然后它就会生成一个密钥对，像我这个设置就会生成yyxtest和yyxtest_pub两个文件，yyxtest.pub是公钥，谁都可以给，但是你私钥要自已保存好，谁拿到了你私钥就呵呵了。

## 上传公钥到github与coding中
根据github上的提示将公钥上传到github上你的个人账户中的ssh中
![上传公钥到github中](/image/2018/usessh2.png)

同样的操作再生成一对密钥，将公钥上传到coding中，注意我这里是为了测试，你完全没有必要重新生成，你当然可以上传刚才生成的yyxtest.pub这个公钥，当然后面会有一些问题，之后再详细说明。

## 上传文件到github上
 赶紧上传个文件到github上试试吧(使用hexo d 来部署)，其实你可以先不用上传文件，可以用`ssh -vT git@github.com` 来查看一下信息
这里你不用试了，肯定是不行的…… 抗都被我踩了
![Permission denied](/image/2018/usessh4.png)
提示Permission denied,原因是你现在用的私钥还是id_rsa，并不是刚才生成的yyxtest，
ssh默认使用id_rsa，如果连id_rsa都没有，那你还不赶紧生成一个默认的。

## 添加yyxtest私钥到git bash中
根据github上的提示[generate SSH keys](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) 先将yyxtest添加到git bash中,
    使用`ssh-add ~/.ssh/yyxtest` ,然后顺利提交代码成功，但是在提交文件到coding时又不行了，提示
>Error: git@git.coding.net: Permission denied (publickey).
fatal: Could not read from remote repository.

没办法，再用刚才的ssh-add 命令将用于coding的私钥也添加到git bash中，这次coding也可以提交了

## 配置ssh本地的config文件
试试关掉这个git bash,然后再试着用hexo d 提交一些文件，这次又不行了，还是提示Permission denied，这怎么能行，总不能每次提交更新都输入ssh-add 添加各种私钥吧，这时就要用到config这个文件了。
如果git安装是默认的话，将会把生成的公钥保存在C:\Users\username\.ssh目录中(我用的是windows，不丢人)，里面如果没有config文件，自已生成一个,里面写一些配置信息,各种字段说明如下
>   **Host**：代码托管平台的别名,但是这个别名和后面要用到的ssh链接 git@github.com:xxx/xxx.git 中的 @ 符号后面的内容要一致，而一般来说github默认提供的就是git@github.com，因此为了方便，github的Host写github.com即可，别取别名了
    **HostName**：代码托管平台真正的IP地址或域名,写域名就行，
    **IdentityFile**：对应的密钥文件路径。必须写绝对路径，windows下可以写 C:\Users\xxx\.ssh\yyxtest
    **PreferredAuthentications**：配置登录时用什么权限认证。可设为publickey，password publickey，keyboard-interactive等
    **User**：对应的用户名。
    
我这里有两个私钥，所以我的配置文件如下
``` python
Host git.coding.net
    HostName git.coding.net
    IdentityFile C:\Users\kevin\.ssh\rsa_coding
    PreferredAuthentications publickey
    User yangyanxing

Host github.com
    HostName github.com
    IdentityFile C:\Users\kevin\.ssh\yyx
    PreferredAuthentications publickey
    User kevinkelin
    
```
之前说过，你没有必要为不同的网站生成不同的密钥，用同一份也可以，如果用同一份，这里IdentityFile也要写一样的
保存之后先不用着急提交，使用ssh -vT 查看一下连接是否有问题
github 提示 `Hi kevinkelin! You've successfully authenticated, but GitHub does not provide shell access.`
coding提示 `yangyanxing，你好，你已经通过 SSH 协议认证 Coding.net 服务，这是一个个人公钥`
这样就可以直接使用`hexo d`来提交文件更新了！

注意的问题
1. config文件中的host 配置是区分大小写的，github.com 和github.COM是不同的，一定要写对
2. coding的host是git.coding.net 而不是coding.net

## 遗留问题
git bash客户端，如果在非C盘上右键的方式启动，那么还是不行，它会寻找当前盘的中的.ssh目录，根本不存在的目录，所有找私钥肯定也找不到，只能先启动git bash,然后cd 到操作的目录中，这个我再研究研究怎么回事。

## 参考文章
- [配置本地和github的ssh密钥对：永久多ssh管理(win10)](https://blog.csdn.net/Yvettre/article/details/79639700)
- [git window下配置SSH连接GitHub](https://blog.csdn.net/w410589502/article/details/53607691)
- [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)