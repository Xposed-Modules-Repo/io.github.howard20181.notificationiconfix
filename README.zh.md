# 通知图标修复

[English](/README.md)

一个模块用于原生、MIUI 和 HyperOS，使用算法转换通知白块图标为可辨认的图标。

## 支持的系统

- Android 8.1 ~ AOSP 主线
- HyperOS
- MIUI 12.5

## 效果

|||
|---|---|
|![Single Notification](/docs/img/3.jpg)|![Multiple notifications with the same icon are automatically grouped](/docs/img/2.jpg)|
|![Multiple notification icons are automatically grouped](/docs/img/1.jpg)|![Multiple notification icons are automatically grouped](/docs/img/4.jpg)|

## 算法详解

1. 判断并缩小超过系统允许的图标
2. 映射图标像素为二维坐标系并计算中心坐标
3. (MIUI) 检测图标透明边缘并裁剪掉
4. (MIUI 和 HyperOS) 判断图标是否有营销横幅并替换为[完美图标](https://github.com/pzcn/Perfect-Icons-Completion-Project) (需要自行安装)
5. (HyperOS) 判断图标是否是天气图标并替换为带温度的图标 (HyperOS 本身有 bug 导致失灵)
6. 计算除了透明像素之外的像素的平均流明
7. 量化边缘像素最多的颜色
8. 计算边缘像素最多颜色与边缘像素在 Lab 颜色空间下的欧几里得距离用于判定是否有边框
9. 判断图标背景是亮色调还是暗色调
10. 使用 K-means 量化器取图标主要颜色
11. 根据背景反色前景并去边
12. 判断图标实际内容区域并基于视觉中心裁剪
13. 最终结果输出并缓存图标到 WeakHashMap
