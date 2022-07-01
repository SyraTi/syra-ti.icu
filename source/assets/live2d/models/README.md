## 本文件夹存放的是live2d的模型

### 由于文件名是游戏内dump等原因导致不好修改，在此记录对应关系如下

| Character | Folder                      | Note             | Processed |
|-----------|-----------------------------|------------------|-----------|
| 魔理沙       | object_live2d_002_101.asset | 东方Canonball      | yes       |
| 芙兰朵露      | object_live2d_011_101.asset | 东方Canonball 普通版  | no        |
| 芙兰朵露       | object_live2d_011_501.asset | 东方Canonball 活动皮肤 | no        |

注：上表最后一列为模型处理情况，模型需要处理过才能正常显示，处理步骤见[模型处理步骤](#模型处理步骤)

### motion3文件的命名规则
目前能够判明的编号命名规则如下
- 序号为 01-08 的motion3文件： 表情文件，需要生成exp3文件使用
- 序号为 [1-5]0 的motions3文件： 应该只包含了手部的样式切换
- 序号为 [1-5][1-5] 的motion3文件： 代表了动作的过度，十位数为起始动作，个位数为目标动作。
- 十位数与个位数相同（11/22/33/44/55）的motion3文件：目前作用不是很明确，似乎是不含有过度动作的切换，目前暂作为未知起始动作时的切换，如：在任意情况下切换到Entry状态，会使用到该类文件。

### 模型处理步骤

1. 为不同的手部创建Pose组，即：[添加pose3文件](https://docs.live2d.com/cubism-editor-manual/pose-setting/?locale=ja)
2. model3中需要添加Pose字段，指定使用的pose3文件。
3. model3中的motion需要整理，目前使用的是：
   - Entry组（会在打开页面时调用一次, 并且闲置一段时间后也会调用一次），这里请使用十位数与个位数相同的motion文件。即：11/22/33/44/55其中之一，来确保模型的动作不会崩坏。
   - "1" 至 "5" 组，代表当前动作所对应的切换pose，组名为当前动作，下标为目标动作。 例如：想要由动作1切换到动作5，应当调用"1"组下标为5的motion，对应的也就是上文所示的序号为15的motion3文件。
4. 将motion序号为01-08的motion3文件，导出为exp3表情文件，具体步骤如下
   1. 使用Cubism Viewer点击［ファイル］メニュー→［読み込み］→［表情用モーション(motion3.json / exp3.json)］
   2. 导出expression文件夹
   3. 在model3.json中添加Expression字段，指定使用的exp3文件
   - 具体可参见[表情の設定と書き出し](https://docs.live2d.com/cubism-editor-manual/setting-and-exporting-facial-expressions/)
