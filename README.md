# face_detector

用 OpenCV 写的人脸检测小工具，命令行运行，支持图片、摄像头和视频。单文件，没有界面依赖。

> 本仓库仅用于个人作品展示；未经许可不得用于商业用途。示例图为合成示意图，不含真实人物。

## 能力口径
- 默认检测器：OpenCV 自带 Haar 级联，开箱即用。
- 可选 DNN：`--dnn` 会尝试加载 Caffe SSD 模型；模型文件**未随仓库提供**，缺失时打印警告并回退 Haar。
- 因此对外描述按「Haar 默认 + 可选 DNN（需自备模型）」理解，不宣称内置双引擎模型。

## 能做什么

- 检测人脸，可以顺带标出眼睛、微笑、侧脸
- 给人脸打马赛克或做高斯模糊
- 把人脸单独裁出来存成图片
- 把检测结果导出成 JSON
- 视频逐帧处理，带进度条和 FPS

## 环境

```bash
pip install -r requirements.txt
```

需要 Python 3.8+。默认分类器用的是 opencv-python 包里自带的 Haar 级联（`cv2/data/`），不用另外下模型；DNN 模型需自备。

## 用法

```bash
# 单张图片
python face_detector.py -i people.jpg

# 标出眼睛和微笑，同时打码
python face_detector.py -i people.jpg --eyes --smile --mosaic

# 摄像头实时检测
python face_detector.py -c

# 处理视频并输出
python face_detector.py -v video.mp4 -o output.mp4

# 裁出每张脸，再导出 JSON
python face_detector.py -i people.jpg --crop --json
```

完整参数用 `-h` 看。

## 实现上的几点

- 人脸检测走 Haar 级联（`haarcascade_frontalface_alt.xml`），侧脸、眼睛、微笑各用对应分类器。
- 图宽超过 800px 时先缩小再检测，检测框再映射回原图，能省不少时间。
- 检测框用 NMS 去重，IoU 阈值 0.3，见 `NMS_IOU_THRESHOLD`。
- 摄像头和视频模式算的是平滑 FPS（取最近 30 帧的平均）。
- Linux 上检测不到 `DISPLAY` 时自动跳过窗口显示，适合丢服务器上跑。

## 已知限制

- Haar 在侧脸、遮挡、逆光下漏检明显，人脸小于 30×30 基本检测不到。选它是为了快，不是为它准。
- 视频是逐帧串行处理，1080p 下帧率掉得多，没做多进程。
- `--dnn` 可以换成 OpenCV 的 Caffe SSD 模型，精度好一些，但 `deploy.prototxt` 和 `res10_300x300_ssd_iter_140000.caffemodel` 不在仓库里，要自己放了才能用；不放会打印警告并回退到 Haar。
- 只在 Windows + Python 3.11 上完整跑过，Linux 的无头分支没实测。

## 文件

- `face_detector.py`：主程序
- `people.jpg`：示例图
