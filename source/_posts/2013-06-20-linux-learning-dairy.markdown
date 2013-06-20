---
layout: post
title: "Linux学习笔记：SSH"
date: 2013-06-20 20:45
comments: true
categories: Linux
---

在学习和使用了Linux一段时间后，觉得有必要把一些使用笔记整理出来，算是给自己做一个备份，前面几篇主要跟bash和shell script有关，ssh其实网上相关的文章很多，不过因为使用频繁，就在这里把一些常用的配置记录下来，便于后续参考。

## 基本用法 ##

最简单也是基本的用法就是直接连接某个url地址，当然，如果用户名不同需要附加用户名：
    ssh -l user shell.cs.helsinki.fi

或者
    ssh user@shell.cs.helsinki.fi

详细的说明参见manual，这里就不再赘述了，因为自己平时主要使用的就是这种最基本的登录，另外，ssh还可以远程调用指令，这个也比较常用，将命令附加在ssh指令后即可：
    ssh user@shell.cs.helsinki.fi “ls”

下文主要叙述的是ssh的常用配置，如果时常要登录远程主机，一些配置还是十分有用的。

## 保存密码 ##

默认情况下，ssh每次登录都需要输入密码，如果经常登录的话确实比较麻烦，这里可以通过ssh-keygen命令来生成密码对，这样，未来登录时就不再需要密码了
 
### 本地生成密码对 ###

    ssh-keygen -t rsa

你也可以选择其它密钥类型，默认情况下密码对会存放到你自己的~/.ssh/路径下，分别是id_rsa和id_rsa.pub，你也可以自行设定路径和名称。另外，ssh-keygen在生成密码时会让你给出passphrase，其实就是使用这个密码对的密码，留空为不设置这个密码，如果设置了，后面会有一个问题，这里暂时不表。

### 拷贝至远程主机 ###

生成密码对后，将id_rsa.pub拷贝到远程要登录的主机，并添加到~/.ssh/authorized_keys文件中，如果没有authorized_keys文件，则在相应路径创建该文件。

    scp ~/.ssh/id_rsa.pub user@remote-server:~/.ssh/authorized_keys

现在，你在本地使用ssh登录到刚才的主机时，就不再需要输入密码了

### Passphrase ###

在上述生成RSA密码的过程中，如果没有设置passphrase，那么后续你的ssh远程连接将不再需要输入密码，这样确实方便，但也增加了密码被盗用的可能，为了增加安全性，ssh建议使用passphrase，然而如果设置了passphrase，每次登录又需要输入另一个密码，与我们最初的目标相背，有没有什么解决方法呢？

方法是有的（目前还有些问题），就是使用SSH agent，你可以将你的密码加入SSH agent，并通过它来管理，这样，你只需要在首次登录时输入passphrase，后面的session都不再需要密码了。

如果没有设置passphrase，可以通过ssh-keygen添加
    ssh-keygen -p -f ~/.ssh/id_rsa

然后，可以通过
    ssh-add ~/.ssh/your_id_rsa

将你的密码添加到ssh agent中（可用ssh-add -l检查是否添加成功），如果你的密码就叫id_rsa，则无需指名文件，ssh-add会自动找到它。

### 问题 ####

楼主在这里使用ssh-add存在问题，实际并没有真正添加成功（Ubuntu12.04, kernel 3.2.0-38-generic），添加完成后ssh连接不再需要密码，但重开系统后，发现ssh agent中没有相关的条目，每次远程登录又需要输入passphrase了。通过GUI的“Passwords and Keys"也可以发现没有添加成功。

暂时的解决方案是设置一个ssh-add的Startup Applications脚本，每次开机都自动设置一次密码（感觉好尴尬，干脆不设置passphrase了... >_<）。

这里还不确定问题所在，因为使用的密码不是默认的名字id_rsa，不知到如果不修改名字，就使用默认的id_rsa效果如何。

## 配置ssh config ##

每次使用ssh，都需要输入一长串url地址，另外有时忘记输入用户名也会导致错误，这两点都比较麻烦，其实我们可以通过配置ssh的config文件来简化输入。

在~/.ssh路径下创建一个名为config的文件（印象中默认是没有这个文件的，如果有就直接编辑它），添加配置信息，比如我希望把shell.cs.helsinki.fi配置为department，就可以在config中添加如下信息：

    Host department
        User user
        HostName shell.cs.helsinki.fi
        IdentityFile ~/.ssh/id_rsa_dpt

这样，以后ssh department就等同与ssh user@shell.cs.helsinki.fi了，其实可配置的内容很多，详细内容可以通过man ssh_config查看，参考[here](http://www.howtogeek.com/75007/stupid-geek-tricks-use-your-ssh-config-file-to-create-aliases-for-hosts/)
