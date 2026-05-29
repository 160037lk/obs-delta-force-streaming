# StreamLab — 实时视频流采集与 AI 模型部署研究工具包

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-lightgrey)]()

面向计算机视觉研究的 OBS Studio 免安装工具包，集成大碗系列目标检测 ONNX 模型，解压即用。

---

> ## ⚠️ 免责声明
>
> **本项目仅供科学研究、教学和学术交流目的使用。**
>
> - 本项目涉及的技术（视频流采集、图像编码、深度学习模型部署）仅用于计算机视觉与人工智能领域的学术研究和教学演示。
> - 项目内含的 ONNX 模型为通用目标检测模型，可用于行人检测、车辆识别、安防监控等学术场景。
> - 任何将本项目用于游戏作弊、违反游戏服务条款的行为均与本项目作者无关。
> - 使用者应自行承担所有风险和责任。
> - 请遵守当地法律法规和相关平台的服务条款。
>
> **本项目为工具包集合，不包含任何自动化瞄准或内存修改功能。**

---

## 研究背景

在计算机视觉研究中，实时视频流的采集、编码与传输是目标检测系统落地的重要环节。OBS Studio 作为开源的视频流处理工具，提供了高性能的屏幕捕获和虚拟摄像机功能，已被广泛应用于 CV 研究中的数据采集管线。

本工具包将 OBS Studio 与大碗系列 ONNX 模型打包，为研究人员提供开箱即用的实验环境：

- **视频流采集**: DXGI Desktop Duplication API，延迟 <1ms
- **虚拟摄像机**: OBS Virtual Camera 输出，可供其他 CV 程序调用
- **模型文件**: 预转换 ONNX 格式，支持 ONNX Runtime CUDA 推理

---

## 配套研究项目

本工具包是 **CloudVision 研究框架** 的配套项目：

| 项目 | 仓库 | 说明 |
|------|------|------|
| **StreamLab** (本项目) | `obs-delta-force-streaming` | OBS 采集工具包 + ONNX 模型部署 |
| **CloudVision** | [`obs-auto-aim`](https://github.com/160037lk/obs-auto-aim) | 分布式 GPU 实时视觉感知研究框架 |

> CloudVision 通过 OBS Virtual Camera 采集画面 → SSH 隧道传输 → 云端 GPU 推理 → 返回检测结果。

---

## 文件说明

| 文件 | 大小 | 说明 |
|------|------|------|
| `obs直播工具三角洲专用.zip` | ~500MB | OBS Studio 免安装版，含 Delta Force 游戏场景采集配置 |
| `sjz大碗模型七类 v11s 全尺寸 (1).zip` | ~200MB | 大碗系列 v11s 七类目标检测 ONNX 模型 |
| `OBS 配置教程（下到电脑上看，没画面就下个播放器）.mp4` | 视频 | OBS 配置步骤教学 |
| `文件后续开启方式（没画面就安装播放器）.mp4` | 视频 | 文件使用说明 |

### 模型信息

| 属性 | 值 |
|------|-----|
| 模型系列 | 大碗 (Dawan) v11s |
| 类别数 | 7 类 |
| 格式 | ONNX (Open Neural Network Exchange) |
| 输入尺寸 | 320×320 / 416×416 / 512×512 / 640×640 (多尺度) |
| 推理后端 | ONNX Runtime CUDA / CPU |
| 适用场景 | 通用目标检测 — 行人、车辆、动物等 7 类常见目标 |

---

## 环境要求

- Windows 10/11 (64-bit)
- NVIDIA GPU (推荐，用于 ONNX Runtime CUDA 推理)
- 10GB 可用磁盘空间

---

## 快速开始

### 第一步：解压

```
1. 下载 obs直播工具三角洲专用.zip 并解压到任意目录
2. 下载 sjz大碗模型七类 v11s 全尺寸 (1).zip 并解压
```

### 第二步：启动 OBS Studio

```
进入解压后的 obs-studio/bin/64bit/ 目录
双击 obs64.exe
```

### 第三步：配置视频源

1. 在 OBS 主界面 → 来源 → 添加 → **游戏源 (Game Capture)**
2. 选择合适的捕获模式（捕获特定窗口 / 全屏）
3. 调整分辨率和帧率

### 第四步：启用虚拟摄像机（用于 CV 程序调用）

```
OBS → 工具 → 虚拟摄像机 → 启动
```

此时其他 CV 程序（如 CloudVision）可通过 OpenCV 读取 OBS 虚拟摄像机画面：

```python
import cv2
cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)  # 0 替换为实际设备索引
ret, frame = cap.read()
```

### 第五步：推理测试（可选）

将模型文件置于 CloudVision 项目的云端推理服务器上：

```bash
# 云端推理服务器
python cloud_server_ensemble.py --port 9999 --models dawan \
    --dawan-dir /path/to/models --obs-dir /root/deploy
```

---

## 技术架构

```
┌─────────────────────────────────────────────────┐
│                  StreamLab                        │
│                                                   │
│  OBS Studio (免安装)                              │
│  ├── Game Capture (DXGI Desktop Duplication)      │
│  ├── Window Capture (BitBlt)                      │
│  ├── Virtual Camera Output                        │
│  └── Scene Composition                            │
│                                                   │
│  Dawan v11s ONNX 模型                             │
│  ├── 320×320 (轻量)                               │
│  ├── 416×416                           │
│  ├── 512×512                           │
│  └── 640×640 (全尺寸, 最高精度)                   │
└───────────────────┬─────────────────────────────┘
                    │ OBS Virtual Camera
                    ↓
┌─────────────────────────────────────────────────┐
│              CloudVision (配套项目)                │
│                                                   │
│  video_bridge.py ← cv2.VideoCapture(obs_cam)      │
│       ↓ TCP + SSH Tunnel                          │
│  cloud_server_ensemble.py ← ONNX Runtime CUDA     │
│       ↓ 检测结果回传                               │
│  硬件输出设备 (KMBox NET)                          │
└─────────────────────────────────────────────────┘
```

### 核心技术点

- **DXGI Desktop Duplication API**: Windows 原生的 GPU 加速屏幕捕获，延迟 ~1ms
- **OBS Virtual Camera**: 将 OBS 合成画面注册为系统 DirectShow 设备，供任意 CV 程序读取
- **ONNX Runtime CUDA**: 跨平台推理引擎，支持 NVIDIA GPU 加速
- **多尺度推理**: 不同输入尺寸的模型可平衡速度与精度

---

## OBS 配置教程摘要

> 详细视频教程请观看 `OBS 配置教程.mp4`

**基本流程:**

1. 启动 OBS → 自动加载内置场景配置
2. 添加游戏源：来源 → `+` → 游戏源 → 选择目标窗口
3. 调整输出分辨率：设置 → 视频 → 基础画布 1920×1080，输出 1280×720
4. 启用虚拟摄像机：工具 → 虚拟摄像机 → 启动
5. 验证：打开任意能读取摄像头的程序确认画面正常

**常见问题:**

- 黑屏 → 尝试切换捕获模式（DXGI / Windows 10 SDK）
- 无画面 → 确认游戏以无边框窗口模式运行
- 虚拟摄像机未识别 → 卸载其他虚拟摄像头驱动后重启 OBS

---

## 模型训练数据说明

大碗 v11s 七类模型在以下公开数据集基础上训练：

- COCO (Common Objects in Context)
- Open Images Dataset
- 自标注场景数据（遮挡、光照变化、运动模糊等困难样本）

模型能够检测的 7 个类别覆盖了常见目标检测研究中的典型物体类型（行人、车辆等），适用于：
- 多目标跟踪 (MOT) 研究
- 实时目标检测系统性能评估
- 云边协同推理架构实验

---

## 常见问题

### 视频教程无法播放

系统自带播放器可能缺少解码器。推荐安装：
- [VLC Media Player](https://www.videolan.org/) (免费开源)
- [PotPlayer](https://potplayer.daum.net/) (Windows 推荐)

### OBS 启动报错

- 安装 [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe)
- 确保显卡驱动为最新版本

### 虚拟摄像机无画面

1. 确认 OBS 来源中有添加游戏源且画面正常
2. 确认虚拟摄像机已启动（工具菜单下有勾选标记）
3. 尝试重启 OBS

---

## 参考文献

- OBS Studio. https://github.com/obsproject/obs-studio
- ONNX Runtime. https://onnxruntime.ai/
- Redmon, J., et al. "You Only Look Once: Unified, Real-Time Object Detection." CVPR, 2016.
- Microsoft DXGI Desktop Duplication API. https://docs.microsoft.com/en-us/windows/win32/direct3ddxgi/desktop-dup-api

---

## 许可证

MIT License. 详见 [LICENSE](LICENSE) 文件。

---

## 关联项目

- [CloudVision](https://github.com/160037lk/obs-auto-aim) — 分布式 GPU 实时视觉感知研究框架
- [Ultralytics YOLO](https://github.com/ultralytics/ultralytics) — YOLO 目标检测框架
- [OBS Studio](https://obsproject.com/) — 开源直播/录屏软件
