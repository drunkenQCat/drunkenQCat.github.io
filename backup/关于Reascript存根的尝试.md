使用`reapy.reascript_api`，pyright频频报错。查看代码，发现这是个运行时才有方法的模块。寻互联网有无存根，无果。遂自行探索。

1. 尝试使用`inspect`模块探查方法以及签名，但只有方法名，无签名
2. 尝试在Reaper内创建脚本，再度inspect，只能输出参数数量，签名不可见
3. vscode有reascript插件，但似乎只支持lua和eel，遂放弃。


又见Reaper会自动生成运行时所有方法的文档，故觉可做爬虫取之为存根。