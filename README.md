# arm-cvd-gpu
ARM CVD GPU加速

# GPU加速
实现GPU加速实际上主要只有两个方向，

1. UEFI+mainline kernel+标准显卡，我们今天主要说这个
2. ARM soc的核显，这个也行，但是我们暂时不谈
## CP8180 + RTX
这个组合目前最省事的有这么一个方案，铭凡MS-R1 + 单槽无独立供电独显

我主要参考了[geerlingguy/sbc-reviews](https://github.com/geerlingguy/sbc-reviews/issues/89) 这篇文章


这篇文章，测试了使用RTX a2000 LP 刀卡，是可以的，但是intel A310并没有成功启动我为了少走弯路，也买了同样显卡他自己自带的debian系统无法正常安装与编译内核驱动，根据文章内容，需要下载最新的ubuntu 25.10 aarch64 镜像，进行安装。安装步骤其实比较离谱：

1. 正常的制作aarch64 ubuntu 25.10的启动U盘
2. 正常的调整第一启动项为U盘
3. 正常的进入usb live。并不正常。这里如果你用核显的hdmi进去之后，他转完圈是卡住的。
4. 此时你把hdmi插到独显的插口中，画面反而出来了，此时是可以正常执行完成安装。
5. 安装完成之后，他那里是说nvidia驱动已经安装而且nvidia-smi也可用。但是我这里并不都是这样的。
我有两台ms-r1有一台按照这个流程是走通了的，另一台出了点问题。下面讲一下第二台的问题

1. ubuntu 25.10 并无发正常安装，会卡住，我没深究，直接改成ubuntu 26.04 每日构建版。一次安装成功。
2. 进去之后发现并没有n卡驱动，没关系，直接从官网下载aarch64版本的即可。
3. 安装完成正常的安装n卡相关的软件即可，nvtop也可以看到已经工作：
<img width="898" height="521" alt="截屏2026-04-02 22 27 31" src="https://github.com/user-attachments/assets/fa686108-ab1a-409a-8494-42a13fd1781e" />
<img width="1672" height="552" alt="截屏2026-04-02 22 28 53" src="https://github.com/user-attachments/assets/15ed1f09-4b02-49ab-a2df-c52d79f6be5e" />


其实到了这一步，你再正常的安装cvd，并使用参数 --gpu_mode=gfxstream 去启动已经可以正常加速了，效果如下： 

<img width="591" height="1075" alt="截屏2026-04-02 22 31 11" src="https://github.com/user-attachments/assets/8b932a76-a510-443b-af17-d863a4d7ade9" />

我们可以看一下跑分： 

<img width="401" height="596" alt="截屏2026-04-02 22 39 20" src="https://github.com/user-attachments/assets/3cce7e65-cf79-485b-af41-7547a193b0b7" />


GPU的跑分是8E的两倍还多，直接爽玩...

么？

是的，驱动起来这并不代表着万事大吉了。

## 问题1: 特征过于明显
我们看一下最下面的拓展这一栏: 50只有50个拓展，缺少的东西非常多。 

<img width="507" height="726" alt="截屏2026-04-02 22 35 04" src="https://github.com/user-attachments/assets/bab8e5a9-7464-4738-b2c1-5a31e1fb27dc" />

而且我们看这个拓展特征 ANDROID_EMU_* 这基本就把自己是模拟器些脸上了。

## 问题2: 效率非常差
我这太机器是 12c+32G, 我按照官方文档关掉了四个小核心，因为官方文档说，这四个小核心的调度器因为不在mainline中，所以无法正常调度，还会拖累，建议关闭。

8c+32G的配置，我开了死太 4c+6G 的虚拟机，静置一段时间。

正常来说，静置一段时间，soc稳定之后，占用率应该能下来的，但是实际上，他不仅没下来，还全程满载。

此时的GPU，只有微小的波动，但是操作界面非常卡，比没有加速还卡。

与AI聊天了解到，这可能是gfxstream图形转发的延迟与开销都由cpu部分来承担了。

如果真的是这个问题，那么这将是CVD多开最大的问题。

## 展望
希望各位大佬，能够集思广益，看看这个异常占用的问题，有没有解决方案。

#结尾
之前看到一直没有公开可用的CVD GPU加速方案，我就放一个方案出来，希望能做一点微小的贡献。
