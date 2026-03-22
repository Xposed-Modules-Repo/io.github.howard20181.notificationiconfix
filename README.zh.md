# 通知图标修复

[English](README.md)

一个模块用于原生、MIUI 和 HyperOS，使用算法转换通知白块图标为可辨认的图标。

## 支持的系统

- Android 8.1 ~ AOSP 主线
- HyperOS
- MIUI 10 及以上

## 效果

|||
|---|---|
|![Single Notification](docs/img/3.jpg)|![Multiple notifications with the same icon are automatically grouped](docs/img/2.jpg)|
|![Multiple notification icons are automatically grouped](docs/img/1.jpg)|![Multiple notification icons are automatically grouped](docs/img/4.jpg)|

## 算法详解

1. 判断并缩小超过系统允许的图标（最大 512×512 dp）
2. (MIUI) 检测图标透明边缘并基于视觉中心对称裁剪
3. (MIUI 和 HyperOS) 判断图标是否有营销横幅并替换为[完美图标](https://github.com/pzcn/Perfect-Icons-Completion-Project)（需要自行安装）
4. (HyperOS) 判断图标是否是天气图标并替换为带温度的图标（HyperOS 本身有 bug 导致失灵）
5. 构建全图像素亮度直方图，使用 Otsu 算法计算全局阈值，并统计亮/暗像素数量比例
6. 采集图标边缘像素，使用 K-means 量化器提取边缘主色
7. 计算所有边缘像素与边缘主色的 Lab 颜色空间欧几里得距离，判定是否存在单色边框
8. 对四条边各采样若干点，计算各边 alpha 加权代表色（跳过边框色像素）；通过"三边相近、一边突出"规则（含严格、宽松、弱三档）检测营销横幅式边缘分割；同时检测角落徽章（单角颜色与其余三角明显不同）
9. 若检测到营销横幅边缘分割、角落徽章或图标分辨率过低，则尝试从应用启动图标中提取替代图标
10. 使用 CelebiQuantizer（Android 12+）或 VariationalKMeansQuantizer 对图标进行 K-means 量化，提取图标主题色
11. 对图标应用圆形蒙版，清除超出内切圆的圆角区域像素（无条件去除圆角边框）
12. 以四条边的代表色为基准，对每个像素进行双线性插值估算背景色，并通过动态距离阈值区分前景与背景；若前景像素数量退化（过少或过多），回退为 Otsu 亮度阈值法提取前景
13. 将前景像素转为灰度；若前景占比过大（>42%），再次执行 Otsu 少数侧清除（梯度前景除外），去除亮/暗二元中的少数一方
14. 对最终灰度前景像素紧凑裁剪透明边缘，基于视觉中心对称输出最小外接矩形
15. 将处理结果（图标与主题色）以 generation ID 为键，通过 SoftReference 缓存至 ConcurrentHashMap
