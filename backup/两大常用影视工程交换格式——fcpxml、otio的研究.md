我手头的视频生成项目现在面临交付到声音部门的“最后一公里”的问题。手动把生成的音频拉到达芬奇，转成AAF，效率不怎么高。需要手动创建轨道，手动排查clip重叠，还可能出错。当前还是整个办公室通力完成一个人物声音的生成，之后若是生成多个人物，可想而知工作量会成倍增加。

## 狗屎fcpxml
生成fcpxml的方案经过我数十小时的研究，被我放弃了。

首先达芬奇生成的fcpxml，必须是一个图像下“包裹”一个音频数组。如果没有图像，那就用Gap做占位符代替图像。这已经是fcpxml官方文档里没有提过的情形了。

其次，但凡有多于一条视频，达芬奇就会在其他音频clip产生大量诡异的时间。比如在第一条普通的视频下叠放的音频~~超量素材~~，`start`和`offset`是一致的：
> 注意以下xml片段均已变换为时间码
```xml
<asset-clip start="00:00:00.000" offset="00:00:00.000" duration="01:12:15.458" format="r1" ref="r2" enabled="1" tcFormat="NDF" name="马东锡 坏家伙.mp4">
    <adjust-transform scale="1 1" anchor="0 0" position="0 0" />
    <asset-clip start="00:14:26.583" lane="2" offset="00:14:26.583" duration="00:00:01.458" ref="r6" enabled="1" name="187_Young Libai_241221_02_4a8d_ZhangJian.wav" />
    <asset-clip start="00:14:36.500" lane="2" offset="00:14:36.500" duration="00:00:04.625" ref="r7" enabled="1" name="189_Young Libai_241220_17_847e_ZhangJian.wav" />
    <asset-clip start="00:14:42.500" lane="2" offset="00:14:42.500" duration="00:00:02.583" ref="r8" enabled="1" name="191_Young Libai_241220_21_ca86_ZhangJian.wav" />
......
	  <asset-clip start="00:21:19.167" lane="15" offset="00:21:19.167" duration="00:00:03.167" ref="r612" enabled="1" name="287_Young Libai_241223_16_e553_ZhangJian.wav" />
</asset-clip>
```
但是，倘若将目光投向下一个Clip：
```xml
<asset-clip start="01:00:00.000" offset="01:12:15.458" duration="01:02:42.792" format="r0" ref="r3" enabled="1" tcFormat="NDF" name="ChangAn_LowRes.mp4">
    <adjust-transform scale="1 1" anchor="0 0" position="0 0" />
    <asset-clip start="01:39:18.583" lane="2" offset="01:27:03.125" duration="00:00:05.375" ref="r60" enabled="1" name="1744_Mid-age Libai_241224_14_9860_ZhangJian.wav" />
```
**噔噔咚！** surprise！第一个音频clip就出现了start 不等于 offset 的诡异情形。这反直觉得很，因为start是音频文件的时间码，offset是音频Clip在轨道上的位置。通过时码同步之后，两个值应该相等。但这个`01:27:03.125`就好像是他妈的天外来客，我根本不知道这是怎么算出来的，帧数相除都是他妈一个丑陋之极的数字。这样的外星数学在生成的文件中比比皆是，令人触目惊心。

除此之外，fcpxml原生似乎是没有“轨道”这个概念的。于是达芬奇的音频clip就都多了一个属性：`lane`，对应达芬奇中音频所在的轨道。但诡异的事情就在这里，达芬奇给了音频lane，但视频，没有lane。是的，视频没有轨道。我根本没法从xml里预测画面，以及画面携带的声音会在哪个轨道上出现。

虽然目前可以将用Gap套音频Clip保证时间码正常，但我觉得如果再用这种给外星人准备的格式，会给后续维护带来成吨的负担。

## Otio 是救世主吗？

当我跟我校优秀校友白龙说，我要写otio，他锐评：
> otio支持更是一坨
> 大家的实现非常迥异
> 兼容性一塌糊涂

这虽然也有些难绷，但想想fcpxml的外星数字，以及达芬奇脚本API的诡异设计，我认为有Track对象的交换格式才是好格式（咬牙切齿）。

### 生成存根

但当我上手的时候，我遇到了第一个麻烦。OpenTimelineIO是跨语言编译的轮子，和达芬奇脚本一样，它默认没有存根。没有存根就意味着所有操作返回的类型都会是Unknown，意味着没有自动补全，意味着写一步就得看一步文档。有人在二月份[讨论了存根的问题](https://github.com/AcademySoftwareFoundation/OpenTimelineIO/issues/1702)，但至今未见某个commit或是pr提到过过存根或是类型提示。

最后我发现了[pybind11存根生成器](https://github.com/sizmailov/pybind11-stubgen)，只需要运行`pybind11-stubgen opentimelineio`，就会在`stub/opentimelineio`生成存根文件。将`opentimelineio`文件夹移动到根目录，LSP就会有类型提示了。

### 生成

使用[randomotio.py](https://github.com/IgorRidanovic/randomOTIO)生成文件，Raven上没有显示任何clip。但在Davinci Resolve中，可以正常导入。那可能说明
现版本Raven要读otio，文件路径必须有效。这点需要进一步验证。

不管怎么说，以randomotio为基础，就可以写更具体的功能。最终写成了这个[audio_otio_composer]。(https://github.com/drunkenQCat/audio_otio_composer)

### 达芬奇的方言

* 轨道通道数：
```json
{
    "OTIO_SCHEMA": "Track.1",
    "metadata": {
        "Resolve_OTIO": {
            "Audio Type": "Stereo", // 轨道类型
            "Locked": false,
            "SoloOn": false
        }
    },
...
```
* 声音通道数：
```json
{
    "OTIO_SCHEMA": "Clip.2",
    "metadata": {
        "Resolve_OTIO": {
            "Channels": [ // Stereo音频，对应数组中两个元素
                {
                    "Source Channel ID": 0, //源音频通道序号
                    "Source Track ID": 0 //轨道中对应通道序号
                },
                {
                    "Source Channel ID": 1,
                    "Source Track ID": 0
                }
            ],
            "Link Group ID": 2 // 链接组，一般是视频用
        }
    },
...
```
注意，若是Clip的通道数多于轨道，则会让Track轨道跟随Clip的通道数类型。例如：轨道设定Mono，但是因为clip有Stereo的音频，会让轨道回退到Stereo。

### RationalTime要点

* value对应的是帧数，rate默认为1（1秒1帧）

### 生成AAF时的离奇现象

出AAF没有声音。我花了两三个小时尽力让我生成的OTIO像达芬奇生成的，但就单单AAF格式，拼尽全力，没有声音，或是只有一部分有声。

最后我迂回了一下，用达芬奇单出时间线的AAF文件，用 pro tools 存档声音工程，解决了交付的问题。

### 使用otio-aaf-adapter转换为aaf的尝试

上一节中提到的方案虽然已经减少了我的工作量，但仍是复杂，我目前的交付工作流是：
`音频文件 -> otio时间线 -> 达芬奇工程 -> AAF时间线 -> Pro Tools 工程`

要有`AAF时间线 -> Pro Tools 工程`这一步是因为这条工作流中，达芬奇输出的AAF时间线文件引用是绝对路径，与常规交付流程中的AAF有异，直接交付会引起录音师困惑。故转而交付Pro Tools工程。

我决定使用[otio-aaf-adapter](https://github.com/OpenTimelineIO/otio-aaf-adapter)将我写的otio直接转换为AAF，省去达芬奇这一步。

我遇到的第一个问题是`mobid`，这是aaf的一个类似于uuid的编号。otio-aaf-adapter默认是需要在otio中给每个片段都写一个mobid的，文件中没有就会崩溃。只要在运行`opentimelineio.adapters.write_to_file`时添加入参`use_empty_mob_ids=True`即可避免。

第二个问题是帧率问题。otio-aaf-adapter会认轨道中第一个片段的帧率为轨道帧率，而我此前写的代码因为只考虑音频，所有片段帧率都是1，输出失败。

改正之后，输出的aaf文件Pro Tools仍无法打开。遂开[issue](https://github.com/OpenTimelineIO/otio-aaf-adapter/issues/55)等待开发者回应。

在等待回应的几个小时里，我又尝试运行了一个[两年前的样例](https://github.com/markreidvfx/otio_to_multi_aaf_alab_example?tab=readme-ov-file)，顺利通过不带aaf元数据的otio文件输出了aaf。这个aaf据作者视频演示，是可以在media composer中使用的。但在Pro Tools中仍然打开失败，遂放弃。