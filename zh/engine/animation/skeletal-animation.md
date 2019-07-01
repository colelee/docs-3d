
# 骨骼动画

目前我们的骨骼动画运行时处理流程如下：
* 动画组件驱动场景的骨骼节点树，每个骨骼节点的 Local Transform (TRS) 信息就是当前帧 骨骼空间 (bone space) 的动作；
* 对当前所有对相机可见的骨骼模型，计算每根骨骼 模型空间 (model space) 下的 Transform 信息，并与骨骼 bindpose 相乘；
* 将当前帧的骨骼信息按指定蒙皮方式统一上传 GPU，在 GPU 完成蒙皮。

## 蒙皮算法

我们内置提供两种常见标准蒙皮算法，它们性能相近，只对最终表现有影响：

1. LBS（线性混合蒙皮）：骨骼信息以 3x4 矩阵形式存储，通过直接线性混合矩阵实现蒙皮，有体积损失等业界已知问题；
2. DQS（双四元数蒙皮）：（推荐）骨骼信息以双四元数和缩放向量的形式存储，能正确处理绝大多数情况。

引擎默认使用 DQS，可以通过修改引擎 skinning-model.ts 的 `updateJointData` 函数引用与 cc-skinning.inc 中的头文件引用来切换蒙皮算法。

## 骨骼信息上传 GPU 的模式

目前我们内置提供以下几种上传模式，不同上传模式之间不应有任何明显最终表现的差异：

| 上传模式 | 说明 |
|:---:|:---:|
| Uniform 模式 | 直接作为 Uniform Float 数组上传，根据 WebGL 标准<br>对 Uniform 数量的规定，我们选择限制在 30 根骨骼 |
| RGBA32 浮点纹理模式 | 每个像素保存 4 个浮点数 |
| RGBA8 普通纹理模式 | 每个像素保存 1 个浮点数，<br>CPU 无任何编码过程，直接在 GPU 解码浮点数 |

引擎默认的模式选择原则是：
* 对所有 30 根骨骼以下的模型，直接使用 Uniform 模式；
* 否则对所有支持浮点纹理的平台，使用 RGBA32 浮点纹理模式；
* 对所有其他平台，使用 RGBA8 普通纹理模式。

可以通过修改 skinning-model.ts 的 `selectJointsMediumType` 函数修改此默认行为。<br>
不同平台的驱动支持效率不同，我们鼓励开发者在不同平台多做测试，并欢迎反馈测试结果。