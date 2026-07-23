# T-LoRa Pager 飞行雷达 v3 （OpenSky数据版）

适用设备：LilyGo T-LoRa Pager V1.0（ESP32-S3、16 MB Flash、8 MB PSRAM、ST7796U 480 × 222）。

<img width="1268" height="732" alt="主界面" src="https://github.com/user-attachments/assets/dab36461-9423-456e-acd7-e6d9b5537137" />


## 本版功能

- 主界面按 `S` 进入设置；`W` / `S` 移动，Enter 选择或确认，`Q` 取消或退出。
- 设置页可扫描并选择 Wi-Fi，填写密码、中心经纬度、OpenSky Client ID、OpenSky Secret 和查询半径。
- OpenSky Client ID 与 Secret 可同时留空，此时使用匿名访问；若使用认证，必须同时填写两项。OpenSky Secret 在列表和编辑时明文显示，便于核对；Wi-Fi 密码仍保持隐藏。
- 设置保存到 NVS，重新启动后仍保留。OpenSky Secret 存在普通 NVS，未加密。
- Wi-Fi 连接后通过 `pool.ntp.org` 和 `time.nist.gov` 同步中国标准时间（东八区，UTC+8）；同步期间主界面显示 `WIFI:NTP`，成功后显示 `WIFI:OK`。
- 主界面显示 BQ27220 电量百分比；飞机型号通过 ADSBDB 的 ICAO24 查询补充并在内存中缓存，每轮 OpenSky 更新最多查询 3 架未缓存飞机。
- 联网后通过 OpenStreetMap Overpass 查询中心坐标 50 km 内带 ICAO 标签的机场，并显示最近机场代码；无结果或查询失败时显示 `----`。
- 主界面按 `L` 在 25%、50%、75%、100% 四档亮度间循环，亮度保存到 NVS。
- 长按左下黄色键约 2 秒进入深度睡眠“关机”；松开后再按同一黄色键唤醒。

固件固定调用 `https://opensky-network.org/api/states/all`，根据中心坐标和半径生成 WGS-84 包围盒，再按圆形实际半径过滤。查询频率根据 OpenSky 包围盒积分成本调整，以控制匿名或认证模式下的每日积分消耗。

默认 Wi-Fi 为占位值、OpenSky 凭据为空、位置为原 Hackster 项目的 Ohrid 坐标。首次运行请按 `S` 设置。预编译固件使用中国标准时间（东八区，UTC+8）。

## 烧录完整 16 MB 镜像

1. 安装 Python 和 esptool：`py -m pip install esptool`
2. 让 T-LoRa Pager 进入下载模式，并确认串口号。
3. 在固件目录运行 `FLASH_MERGED.bat COM7`，将 `COM7` 换成实际端口。

等价命令：

```powershell
py -m esptool --chip esp32s3 --port COM7 --baud 460800 write-flash 0x0 TLoraPagerAirRadar.ino.merged.bin
```

若 460800 不稳定，改用 115200。串口日志波特率为 115200。

## 分段烧录地址

```text
0x0000  TLoraPagerAirRadar.ino.bootloader.bin
0x8000  TLoraPagerAirRadar.ino.partitions.bin
0xE000  boot_app0.bin
0x10000 TLoraPagerAirRadar.ino.bin
```
烧录工具网址：https://esp.eterill.xyz/

## 构建与验证

- Arduino CLI 1.5.1
- Arduino-ESP32 3.3.10（内含 ESP-IDF 5.5.4）
- LilyGoLib 0.2.0
- ArduinoJson 7.4.2
- 板型 `esp32:esp32:tlora_pager`
- Board Revision `Radio_SX1262`（本程序不初始化 LoRa）
- 分区 `app3M_fat9M_16MB`
- App 1,466,425 字节，占 3 MB App 分区 46%
- 全局变量 59,672 字节，占内部动态内存 18%
- App 与完整 16 MB 镜像经 esptool 5.3.0 检查：ESP32-S3、16 MB、80 MHz、DIO，校验和与 validation hash 有效

用户已确认此前雷达主界面和 OpenSky 实时数据在实机工作。本版已完成编译和镜像静态检查；机型查询、最近机场、电量计读数、最终列对齐、亮度、深睡唤醒和待机电流仍需在实机验证。OpenSky、ADSBDB 与 Overpass 的 TLS 证书验证目前关闭；“关机”是深度睡眠，不是 PMU 完全断电。
