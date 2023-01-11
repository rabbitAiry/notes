首先是要完成Fyde的系统要求，把将系统设置为UEFI启动方式，这可以通过在Windows中，使用win+R输入msinfo32，然后在系统摘要中查看BIOS模式

支持UEFI与否和主板相关，因为UEFI启动方式需要硬盘AHCI模式，而家里的电脑一直是硬盘ide模式，所以我决定重装win10系统。首先要准备一个重装系统的U盘，然后先在BIOS中更改配置参数，最后启动U盘的PE系统并安装Windows。

注意的是FydeOS还要求硬盘以GPT方式分区，应该对应的是Guid的分区表类型。这一步可以在重装系统的时候顺手完成

酷安里面不乏重装系统的教程，这里就不重复了。而更改BIOS参数可以通过在百度搜索如“昂达H61设置以UEFI方式启动”的关键字，应该能找到操作指南，这里记录下我电脑的设置步骤
- 进入Chipset>>SB Configuration>>SATA Configuration，选择AHCI

一直等待，直到看到“海内存知己，天涯若比邻”，就基本成功了

第二步是为fyde做准备准备，这一步可以根据官方教程，把镜像文件刻录到U盘中。官方使用的是balenaEtcher，刻录过程的操作十分简单。

插入U盘并启动U盘的操作系统。等待许久后出现选择语言了，随后选择多重引导安装，并选择“帮我安装及配置rEFind启动器”，跟随指引等待，关机，拔U盘，重启，大功告成。

我使用双系统的初衷是因为这台电脑并不是只有我一个人会用得到。家里人会用到Windows中一些软件，而我希望在fyde上完成自己的事情。rEFind默认会启动上次选择的操作系统，但家里人或许会并不太懂得双系统之类的概念，因此我希望rEFind会默认启动Windows。

所以我又上网百度了一下如何配置rEFind。参照的教程在[这里](https://blog.csdn.net/kuguazhiwang/article/details/127293654)。这一步操作要在Windows中进行，使用powershell或者cmd都可以，但是必须使用管理员打开（理论上两个系统都可以，可是我不会。。）。在以下操作中，powershell和cmd没有体现出太多差别，唯一要注意的是查看当前目录下文件的命令：powershell的`ls`命令对应cmd的`dir`命令
```sh
mountvol R: /S
R:
cd /EFI/refind
notepad refind.conf #使用笔记本编辑
code refind.conf #使用vsc编辑
```

在refind.conf文件中，default_selection选项决定了默认启动的系统，既可以使用数字表示显示中的第几个系统，也可以直接指明是“Microsoft”

