## 本文件夹存放的是live2d的模型

### 由于文件名是游戏内dump等原因导致不好修改，在此记录对应关系如下

| Character | Folder                      | Note             | Processed |
|-----------|-----------------------------|------------------|-----------|
| 魔理沙       | object_live2d_002_101.asset | 东方Canonball      | yes       |
| 芙兰朵露      | object_live2d_011_101.asset | 东方Canonball 普通版  | no        |
| 芙兰朵露       | object_live2d_011_501.asset | 东方Canonball 活动皮肤 | no        |

注：上表最后一列为模型处理情况，模型需要处理过才能正常显示，处理步骤如下：

1. 为不同的手部创建Pose组，即：[添加pose3.json文件](https://docs.live2d.com/cubism-editor-manual/pose-setting/?locale=ja)
2. model3中需要添加Pose字段
3. model3中的motion需要整理，目前使用的是
   - Entry组（会在打开页面时调用一次, 并且闲置一段时间后也会调用一次）
   - Tap组（会在点击时调用）
