我手头的视频生成项目现在面临交付到声音部门的“最后一公里”的问题。手动把生成的音频拉到达芬奇，转成AAF，效率不怎么高。需要手动创建轨道，手动排查clip重叠，还可能出错。当前还是整个办公室通力完成一个人物声音的生成，之后若是生成多个人物，可想而知工作量会成倍增加。

## 狗屎fcpxml
生成fcpxml的方案经过我数十小时的研究，被我放弃了。

首先达芬奇生成的fcpxml，必须是一个图像下“包裹”一个音频数组。如果没有图像，那就用Gap做占位符代替图像。这已经是fcpxml官方文档里没有提过的情形了。

其次，但凡有多于一条视频，达芬奇就会在其他音频clip产生大量诡异的时间。比如在第一条普通的视频下叠放的音频~~超量素材~~jj，`start`和`offset`是一致的：
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

但当我上手的时候，我遇到了第一个麻烦。OpenTimelineIO是跨语言编译的轮子，和达芬奇脚本一样，它默认没有存根。没有存根就意味着所有操作返回的类型都会是Unknown，意味着没有自动补全，意味着写一步就得看一步文档。有人在二月份[讨论了存根的问题](https://github.com/AcademySoftwareFoundation/OpenTimelineIO/issues/1702)，但至今未见某个commit或是pr提到过过存根或是类型提示。

很要命，但也不是不能写。先走一步看一步了。

### 生成存根
我用官方文档，喂给cursor，生成了一部分存根。

### Timeline
```python
otio.schema.Timeline(name='Generated with randomotio.py', tracks=otio.schema.Stack(name='tracks', children=[], source_range=None, metadata={
'Random OTIO': {'version': '0.1.0'}}))
```

### 第一次失败

使用randomotio.py生成文件，raven上没有显示任何clip。附代码：
```python

import opentimelineio as otio
from random import randint, choice
import os
import string
import datetime


def _rand_string(r=10):
    letters = string.ascii_lowercase
    randString = "".join(choice(letters) for i in range(r))

    return randString


def create_timeline(global_start_hour: int, fps: int) -> otio.schema.Timeline:
    """
    创建一个新的 OTIO 时间轴并设置元数据和全局起始时间。

    :param global_start_hour: 时间轴的全局起始时间（小时）。
    :param fps: 时间轴的帧率。
    :return: 一个 OTIO 时间轴实例。
    """
    # 创建时间轴实例并设置名称
    timeline = otio.schema.Timeline()
    timeline.name = "Generated with randomotio.py"

    # 设置全局起始时间
    seconds = global_start_hour * 60**2
    hour_one_frames = otio.opentime.to_frames(
        otio.opentime.RationalTime(value=seconds), rate=fps
    )
    timeline.global_start_time = otio.opentime.RationalTime(hour_one_frames, fps)

    # 添加元数据
    timeline.metadata["Random OTIO"] = {"version": "0.1.0"}
    return timeline


def create_tracks(trk_count: int) -> list[otio.schema.Track]:
    """
    创建指定数量的空 OTIO 轨道。

    :param trk_count: 要创建的轨道数量。
    :return: 一个包含 OTIO 轨道实例的列表。
    """
    # 创建指定数量的轨道
    return [otio.schema.Track() for _ in range(trk_count)]


def populate_track_with_clips(
    track: otio.schema.Track, clp_count: int, fps: int, max_tc: int
):
    """
    为指定的轨道添加剪辑和间隙。

    :param track: 要填充的 OTIO 轨道。
    :param clp_count: 要添加到轨道中的剪辑数量。
    :param fps: 剪辑的帧率。
    :param max_tc: 剪辑来源范围的最大时间码。
    """
    for _ in range(clp_count):
        clip = create_random_clip(fps, max_tc)
        track.append(clip)

        # TODO: 还要在Resource里提前算好Gap，不，直接做成dict[str, dict[Clip/ExternalReference, Gap]]
        # 创建随机间隙 (Gap)
        if randint(0, 1) == 1:
            gap = otio.schema.Gap()
            gap_duration = clip.source_range.duration
            gap.source_range = otio.opentime.TimeRange(duration=gap_duration)
            gap.name = "Black"
            track.append(gap)


def create_random_clip(fps: int, max_tc: int) -> otio.schema.Clip:
    """
    创建一个带有随机元数据和媒体引用的 OTIO 剪辑。

    :param fps: 剪辑的帧率。
    :param max_tc: 剪辑来源范围的最大时间码。
    :return: 一个 OTIO 剪辑实例。
    """
    clip = otio.schema.Clip()

    # 随机生成剪辑名称
    clip_name = _rand_string()
    clip.name = clip_name

    # 设置剪辑的来源范围
    min_dur = 1
    max_dur = int(10 * fps)
    tc_in = randint(0, max_tc)
    # TODO: duration转换
    duration = randint(min_dur, max_dur)

    clip.source_range = otio.opentime.TimeRange(
        start_time=otio.opentime.RationalTime(tc_in, fps),
        duration=otio.opentime.RationalTime(duration, fps),
    )

    # 添加虚拟媒体引用
    rand_path1 = _rand_string(8)
    rand_path2 = _rand_string(4)
    # TODO: 音频文件路径
    url = os.path.join(
        "/Volumes", "FakeProject", rand_path1, rand_path2, clip_name + ".mov"
    )

    # TODO: 从fcpxml的Resource那儿偷代码，先做一个dict[str, ExternalReference]
    clip.media_reference = otio.schema.ExternalReference(
        target_url=url,
        available_range=otio.opentime.TimeRange(
            start_time=otio.opentime.RationalTime(0, fps),
            duration=otio.opentime.RationalTime(86400, fps),
        ),
    )

    # TODO: 写上我的大名
    # 添加剪辑元数据
    timestamp = datetime.datetime.now()
    clip.metadata["Random OTIO"] = {
        "Creation Time": str(timestamp),
        "Generated By": "randomotio.py",
    }

    return clip


def set_track_source_range(
    track: otio.schema.Track, start_time: otio.opentime.RationalTime
):
    """
    将轨道的来源范围设置为与全局起始时间匹配。

    :param track: 要更新的 OTIO 轨道。
    :param start_time: 要设置的起始时间。
    """
    track.source_range = otio.opentime.TimeRange(start_time, track.duration())


def make_otio(trk_count: int, clp_count: int, global_start_hour: int, fps: int):
    """
    生成一个包含随机轨道和剪辑的 OTIO 时间轴。

    :param trk_count: 要创建的轨道数量。
    :param clp_count: 每个轨道的剪辑数量。
    :param global_start_hour: 时间轴的全局起始时间（小时）。
    :param fps: 时间轴的帧率。
    """
    timeline = create_timeline(global_start_hour, fps)
    tracks = create_tracks(trk_count)

    # 每个轨道中填充随机剪辑和间隙
    hour_one_frames = otio.opentime.to_frames(
        otio.opentime.RationalTime(global_start_hour * 60**2), rate=fps
    )
    for track in tracks:
        # TODO: 改成角色名-序号
        track.name = f"V{tracks.index(track) + 1}"
        populate_track_with_clips(track, clp_count, fps, hour_one_frames)
        set_track_source_range(track, otio.opentime.RationalTime(-hour_one_frames, fps))
        timeline.tracks.append(track)

    # 输出 OTIO 文件
    otio.adapters.write_to_file(timeline, "random.otio")

    # 可以根据需要输出其他格式
    # otio.adapters.write_to_file(timeline, 'random.xml')
    # if trk_count == 1:
    #     otio.adapters.write_to_file(timeline, 'random.edl')


def main():
    """
    主函数，用于生成具有用户定义参数的随机 OTIO 时间轴。
    """
    # 设置参数
    trk_count = 3  # 轨道数量
    clp_count = 5  # 每个轨道的剪辑数量
    global_start_hour = 1  # 时间轴全局起始时间（小时）
    fps = 24  # 帧率

    # 调用主函数生成时间轴
    make_otio(trk_count, clp_count, global_start_hour, fps)


if __name__ == "__main__":
    main()
```