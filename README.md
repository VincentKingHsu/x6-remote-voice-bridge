# X6 Remote Voice Bridge for macOS

把 X6 Pro 蓝牙语音遥控器的麦克风接入 macOS，并通过 `MiRemoteV 2ch`
提供给豆包输入法等只接受“硬件型”输入设备的应用。

## 当前基线

版本：`2.0.0-beta.1`（2.0 Beta V1）

2026-08-06 在 Apple Silicon Mac、X6 Pro 和豆包输入法上完成真实设备验收：

- X6 语音键短按一次开始录音，再短按一次结束录音。
- Mac 键盘 Option 短按一次开始，再短按一次结束。
- Mac Option 开始、X6 结束连续三轮通过。
- X6 开始、Mac Option 结束连续三轮通过。
- 清理旧版停用实现后，再次完成两个方向的交叉回归。

## 已知限制

X6 当前固件在第一次启动 ATVV 音频时，不向 macOS 的 HID 或 ATVV 通道
上报语音键松手事件。独占 HID 探针能够收到音量键的按下与抬起，但长按语音键
松手后既没有 HID release，也没有 `AUDIO_STOP`。因此本基线只承诺稳定的
“短按切换”模式，不用延迟或静音检测伪造长按松手。

## 运行要求

- macOS 12 或更高版本。
- X6 Pro 已在系统蓝牙设置中配对。
- 已安装 BlackHole 2ch，供 `Driver/build-driver.sh` 构建独立驱动。
- 已安装并启用生成的 `MiRemoteVoice.driver`。
- 豆包输入法选择 `MiRemoteV 2ch`。
- 首次启动时按系统提示授予蓝牙、辅助功能、输入监控和麦克风权限。

## 构建与安装音频驱动

驱动构建脚本复制本机已安装的 `BlackHole2ch.driver`，生成并签名独立的
`MiRemoteVoice.driver`；它不会修改现有 BlackHole。

```bash
./Driver/build-driver.sh
sudo ./Driver/install-driver.sh
```

安装后需要重启 `coreaudiod`；安装脚本会自动执行。驱动详情及限制见
[`Driver/README.md`](Driver/README.md)。

## 构建与安装应用

```bash
./run-self-tests.sh
./package-app.sh
./install-app.sh
open ~/Applications/MiRemoteBridgeV2Beta.app
```

应用采用稳定的本地 ad-hoc designated requirement，避免每次重编译都被 macOS
当作全新的权限主体。它是菜单栏应用，同时提供一个简单前台控制台。

## 数据与日志

- X6 UUID：`~/Library/Application Support/mi-remote-bridge/x6-uuid.txt`
- 日志：`~/Library/Logs/MiRemoteBridge/MiRemoteBridge.log`
- 调试录音：`~/Library/Application Support/MiRemoteBridge/Recordings/`

UUID、日志、录音、构建目录、应用包和驱动包均被 `.gitignore` 排除。日志当前在
调试阶段保持开启，但最多 5 MB；调试录音默认关闭。

## 目录

- `Sources/MiRemoteBridge/`：X6 HID、BLE/ATVV、豆包状态与音频桥接。
- `SelfTests/`：ATVV/ADPCM 状态回归测试。
- `Driver/`：从 BlackHole 2ch 生成独立 MiRemoteV 驱动的可审计脚本。
- `Packaging/`：macOS App 元数据。

## 许可

应用代码采用 MIT License。`MiRemoteVoice.driver` 基于 BlackHole 0.4.1，
遵循 GNU GPL v3；详见 [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)。
