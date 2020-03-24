
# 网页内看完广告白屏问题分析
# WKWebview白屏分析


<a name="gFsj7"></a>
### 介绍:

- 先简单介绍下 项目中用的WKWebview系统特性

在iOS系统上, 开发者所开发的App在生命周期内一般都是单进程, 开发者无法自行申请进程.<br />当然也存在多进程, 就比如开发者调用了系统的WKWebview,WKWebview是系统的独立进程, 内存由系统自己管理, 和我们App不在一个进程内, 这也就是WKWebview相对于UIWebview,为什么内存降低了.

- WKWebview和UIWebview的在系统管理上的区别:

WKWebview是个单独进程不占用App内存,  一定程度上减少了App自身内存警告, 会让我们应用活的更安全、更长久.<br />UIWebview是App自身进程的一部分, 所以内存申请的越多,对App本身的威胁也就更大, App申请内存都有上限(系统总内存的45% 参考值), 不过以后苹果也不让用了

业务场景,在应用内已打开的WKWebview上再弹出其他页面,经常会出现白屏
<a name="jIVrD"></a>
### 分析流程:
思路：用开发工具自带监测工具监测合成灯笼网页内存数据变化<br />步骤：<br />1，屏蔽广告预加载对内存的影响，从启动app到打开一个合成灯笼，    到连续打开4个合成灯笼网页的内存变化<br />2，不屏蔽广告预加载对内存的影响，重复1

**划横向红线的就是由贪吃蛇App申请的需要我们关心的进程, 剩下的都是系统或者其他****App****的进程, 和我们项目无关**<br />**下面是从启动干净的贪吃蛇 到 加载很多网页之后的 ****内存数据对比**
<a name="xTLvx"></a>
### 具体流程:
<a name="i1yCL"></a>
### 1. 启动之后, 注释掉所有的广告预加载, 保证不加载任何webview, 系统进程内存分配如下: 
<a name="Jf6EI"></a>
### 我们发现没有任何web方面的进程申请, 贪吃蛇自身占用了93M内存
![image.png](https://cdn.nlark.com/yuque/0/2020/png/169030/1584525062388-2db2ce99-5fa0-4ff2-8788-eeee99d88a18.png#align=left&display=inline&height=777&name=image.png&originHeight=1554&originWidth=2488&size=629505&status=done&style=none&width=1244)
<a name="Q3Y21"></a>
### ----------------------------------------------------------
<a name="Sls3g"></a>
### 2.点击合灯笼活动网页之后:
<a name="Xunmg"></a>
### 贪吃蛇进程增加了10M左右, 一个Webview分配10M无关紧要, 
<a name="Y9a1x"></a>
### 新增了一个WebKit进程, 306M, 几乎榜首
![image.png](https://cdn.nlark.com/yuque/0/2020/png/169030/1584525212006-f9cdae06-f50b-4043-be97-274f6bc90d5f.png#align=left&display=inline&height=777&name=image.png&originHeight=1554&originWidth=2488&size=642431&status=done&style=none&width=1244)
<a name="qsdbq"></a>
### ----------------------------------------------------------
<a name="eudHS"></a>
### 3.连续打开四个合灯笼活动网页之后, 每一个都是一个新的进程, 排在最上面的是最上层可见的网页, 占用内存最多,其余的有所降低
![image.png](https://cdn.nlark.com/yuque/0/2020/png/169030/1584525730567-d6e1fa95-2f94-47c9-9848-1e93794306c4.png#align=left&display=inline&height=777&name=image.png&originHeight=1554&originWidth=2488&size=649125&status=done&style=none&width=1244)
<a name="X1NMa"></a>
### ---------------------------------------------------------
<a name="kquNH"></a>
### 4.当再多打开几个之后, 我们可以看到之前打开的webview进程开始变的不稳定,processID也在变化
<a name="jQKJX"></a>
###   这是因为iOS系统内存紧张,在不停的释放那些不可见的webview, 同时也触发了我们的业务监听webview.title为空又重新reload, 所以这些系统WebKit进程内存一直在申请与释放的循环中
<a name="28gbX"></a>
### 这是张动图:![webview.gif](https://cdn.nlark.com/yuque/0/2020/gif/169030/1584525993321-8b866552-8136-4a64-81d8-378066514892.gif#align=left&display=inline&height=463&name=webview.gif&originHeight=463&originWidth=1203&size=767552&status=done&style=none&width=1203)

<a name="ZsszN"></a>
### ---------------------------------------------------------
<a name="S8DET"></a>
### 5.我们重启APP再来看看我们打开广告预加载后, 预加载(不单单是下载资源)第三方SDK有些也会采取预先渲染webview的方式
<a name="8Y5zl"></a>
### 多了好几个WebKit进程,但内存占用都很低
![image.png](https://cdn.nlark.com/yuque/0/2020/png/169030/1584524385478-be7e98b5-b445-469b-8525-104aac6ea43a.png#align=left&display=inline&height=777&name=image.png&originHeight=1554&originWidth=2488&size=654753&status=done&style=none&width=1244)
<a name="91ael"></a>
### ---------------------------------------------------------
<a name="r3iT2"></a>
### 6.在上面5的基础上, 打开了合灯笼活动网页之后, 广告的web进程都没变化, 新增了一个WebKit进程,就是我们合灯笼活动网页, 内存比系统任何一个进程占用的都要多
![image.png](https://cdn.nlark.com/yuque/0/2020/png/169030/1584524580338-4d02f4ee-882f-4aca-986d-b212dcacd0bc.png#align=left&display=inline&height=777&name=image.png&originHeight=1554&originWidth=2488&size=675222&status=done&style=none&width=1244)

<a name="OpFb7"></a>
### 疑问:
可能会有疑问?打开多个web在我们项目,不多见,可能只会有两层(小游戏和广告)叠加, 那么你这么举例抓数据合适吗?

答:<br />我这么做只是为了触发系统内存自动回收机制, 来观察系统到底做了什么. 至于我申请了什么进程其实不重要,  只是iOS系统机制原因,开发者只能申请web进程.<br />线上用户的实际场景应该是:  正常人会连续用到很多应用, 系统内存会存在很多其他类型的进程, 当在我们的web上面又弹出广告网页进程时,系统可用内存不足, 就会回收一些进程, 正好回收的是我们自己的占用内存大不可见的webview进程.

<a name="xOmPg"></a>
### 结论:

- iOS系统内存管理属于比较强硬的作风, 内存占用一直居高不下的直接杀死, 就连App自身进程超出内存限制也要被强制kill掉, Webkit进程还只是释放了下, 要是换成UIWebview, 整个app估计早就挂了
- 或者在网页收到不可见时JS方法时,  能主动释放一些不必要的内存申请
- 继续做坚持做网页业务恢复功能
- 我们网页小游戏渲染之后占用内存太高, 合灯笼活动网页 都在300M左右,是我们App自身内存三倍.
- 通过对比 300M的进程在系统进程占比非常大, 是系统首选被释放对象
- 希望能从前端入手, 建立起明确的内存指标,300M的内存占用,非常的不健康


