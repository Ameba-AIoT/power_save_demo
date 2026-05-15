# Ameba RTL8721Dx WiFi 省电模式（LPS）功耗测试示例（FreeRTOS）

* [English Version](./README.md)

这是一个基于 RTL8721Dx 系列 SoC 的 **WiFi 省电模式（LPS）下系统电流功耗测试** 示例程序。  
该示例展示设备在连接上 WiFi 后进入低功耗模式时的实际功耗表现以及推荐的测试方法。

- 📎 开发板链接  [🛒 淘宝](https://item.taobao.com/item.htm?id=904981157046) | [📦 Amazon](https://www.amazon.com/-/zh/dp/B0FB33DT2C/)
- 📄 [芯片详情](https://aiot.realmcu.com/cn/module/index.html)
- 📚 [WiFi 省电模式参考文档](https://aiot.realmcu.com/cn/latest/rtos/ps/powertest/index.html#wi-fi)

---

## ✨ 功能

- 连接到指定 AP 后进入 WiFi LPS 低功耗模式，示例中设定 DTIM = 10  
- 配合 AT 指令，观察进入 LPS 后的模组电流变化与功耗水平  

---

## 工作原理

1. 模组上电启动示例工程，串口侧提供 AT 命令交互能力。  
2. 用户通过 AT 指令连接到指定 WiFi AP 并等待连接稳定。  
3. 通过 `AT+WLSTATE` 和 `AT+WLRSSI` 确认连接状态、RSSI、信道等信息。  
4. 使用 `AT+WLDBG=lps_dtim_set n` 配置 LPS DTIM 周期，本示例推荐 `n = 10`（约 1 秒接收一次 Beacon）。  
5. 通过 `AT+TICKPS=R` 进入低功耗模式（WiFi 以LPS模式运行）。  
6. 使用电流表从模块端单独取电，观测在 LPS 下的 WiFi 维持连接功耗。

> 建议在 **屏蔽环境** 或电磁干扰较小的环境下进行测试，以避免外部干扰（例如：频繁重连、信道切换、干扰导致重传等）对功耗的影响。

---

### 🔌 硬件接线

- 使用官方指定 EVB（评估板）；  
- 从 **模块端直接供电**，确保量测到的是模组本身的实际功耗；  
- 接线方式参考下图：  

![alt text](image-1.png)

---

## 🚀 快速开始

1️⃣ **配置 SDK 环境**

- 设置 `env.sh`（或 `env.bat`）路径：

  ```bash
  source {sdk}/env.sh
  ```

- 将 `{sdk}` 替换为 [ameba-rtos SDK](https://github.com/Ameba-AIoT/ameba-rtos) 根目录下 `env.sh` 的绝对路径。  
- 若 SDK 路径不变，该步骤仅需执行一次。  

⚡ **注意**：本示例仅支持 SDK 版本 **≥ v1.2**

---

2️⃣ **编译示例工程**

在本 demo 工程根目录下执行：

```bash
source env.sh
ameba.py build
```

---

3️⃣ **烧录 Flash**

使用当前工程目录下编译生成的 bin：

```bash
ameba.py flash --p COMx --image boot.bin 0x08000000 0x8014000 --image app.bin 0x08014000 0x8200000
```

**或使用上一级目录中的预编译 bin（工程目录已提供）：**

```bash
ameba.py flash --p COMx --image ../boot.bin 0x08000000 0x8014000 --image ../app.bin 0x08014000 0x8200000
```

> ⚠️ **bin 文件命名说明**：bin 文件名称取决于所使用的 SDK 版本。
> 最新 SDK 编译产物为 `boot.bin` + `app.bin`；
> 旧版 SDK 编译产物为 `km4_boot_all.bin` + `km0_km4_app.bin`。
> 请根据实际编译输出修改上方命令中的文件名。

> 将 `COMx` 替换为实际串口号，例如 `COM5`。

---

4️⃣ **打开串口监视**

```bash
ameba.py monitor --port COMx --b 1500000
```

---

5️⃣ **连接 WiFi**

通过 AT 指令连接到 AP，参考官方文档 [AT+WLCONN](https://aiot.realmcu.com/cn/latest/rtos/atcmd/at_command_wifi.html#at-wlconn)。

示例命令：

```bash
AT+WLCONN=ssid,Xiaomi_Pro_2G,pw,12345678
```

---

6️⃣ **查询当前连接状态及信号强度**

- 使用 `AT+WLSTATE` 查询当前连接信息（SSID、BSSID、Channel 等）  
- 使用 `AT+WLRSSI` 查询当前连接的信号强度  

```bash
   AT+WLSTATE
   AT+WLRSSI
```

参考文档：  
- [AT+WLRSSI](https://aiot.realmcu.com/cn/latest/rtos/atcmd/at_command_wifi.html#at-wlrssi)  
- [AT+WLSTATE](https://aiot.realmcu.com/cn/latest/rtos/atcmd/at_command_wifi.html#at-wlstate)

> **注意**：WiFi 的 RSSI、所处信道、周围干扰情况等都会显著影响 LPS 下的功耗表现。  
> 建议在屏蔽箱或较为干净的射频环境中进行测试，避免因重连、重传导致的测试结果偏高。

---

7️⃣ **配置 DTIM = 10**

连接稳定后（通常需要等待 10–20 秒），通过以下指令设置 LPS 下的 DTIM 周期：

- 命令格式：`AT+WLDBG=lps_dtim_set n`  
- 含义：配置 LPS 模式下，每经过 `n` 个 Beacon 周期唤醒一次；  
- 例如：`n = 10` 时约为 **1 秒唤醒接收一次 Beacon**（具体取决于 AP 的 Beacon 周期配置）。

示例：

```bash
AT+WLDBG=lps_dtim_set 10
```

---

8️⃣ **进入低功耗模式**

使用以下命令进入超低功耗模式：

```bash
AT+TICKPS=R
```

参考文档：  
- [AT+TICKPS](https://aiot.realmcu.com/cn/latest/rtos/atcmd/at_command_common.html#at-tickps)

---

此时可通过电流表观察模组在该状态下的平均电流变化，评估 WiFi 连接维持状态的功耗。
![alt text](image.png)

---

## 📝 日志示例

```plaintext
log：
14:28:17.631  ROM:[V1.1]
14:28:17.631  FLASH RATE:1, Pinmux:0
14:28:17.631  IMG1(OTA1) VALID, ret: 0
14:28:17.631  IMG1 ENTRY[f800779:0]
14:28:17.631  [BOOT-I] KM4 BOOT REASON 0: Initial Power on
14:28:17.632  [BOOT-I] KM4 CPU CLK: 240000000 Hz
14:28:17.632  [BOOT-I] KM0 CPU CLK: 96000000 Hz
14:28:17.632  [BOOT-I] PSRAM Ctrl CLK: 240000000 Hz 
14:28:17.632  [BOOT-I] IMG1 ENTER MSP:[30009FDC]
14:28:17.632  [BOOT-I] Build Time: Mar  3 2026 16:51:41
14:28:17.632  [BOOT-I] IMG1 SECURE STATE: 1
14:28:17.632  [FLASH-I] FLASH CLK: 80000000 Hz
14:28:17.632  [FLASH-I] Flash ID: c8-40-17 (Capacity: 64M-bit)
14:28:17.632  [FLASH-I] Flash Read 4IO
14:28:17.632  [FLASH-I] FLASH HandShake[0x2 OK]
14:28:17.632  [BOOT-I] Init APM PSRAM
14:28:17.632  [PSRAM-I] Cal win size 31
14:28:17.632  [BOOT-I] KM0 XIP IMG[0c000000:53c00]
14:28:17.632  [BOOT-I] KM0 SRAM[20068000:30e0]
14:28:17.632  [BOOT-I] KM0 PSRAM[0c056ce0:20]
14:28:17.632  [BOOT-I] KM0 ENTRY[20004d00:60]
14:28:17.632  [BOOT-I] KM4 XIP IMG[0e000000:66f20]
14:28:17.632  [BOOT-I] KM4 SRAM[2000b000:1ee0]
14:28:17.632  [BOOT-I] KM4 PSRAM[0e068e00:20]
14:28:17.632  [BOOT-I] KM4 ENTRY[20004d80:40]
14:28:17.632  [BOOT-I] IMG2 BOOT from OTA 1, Version: 1.1 
14:28:17.632  [BOOT-I] Image2Entry @ 0xe00d43d ...
14:28:17.632  [APP-I] KM4 APP START 
14:28:17.632  [APP-I] VTOR: 30007[000LOC, VTOR_NKS-I] KMS:3000700 init_retarget_00
14:28:17.632  locks
14:28:17.632  [APP-I] VTOR: 30007000, VTOR_NS:30007000
14:28:17.632  [APP-I] IMG2 SECURE STATE: 1
14:28:17.632  [MAIN-I] IWDG refresh on[!C
14:28:17.632  LK-I] [CAL4M]: delta:[M0A tINar-gIet]: 3K2M00 P POMS:  0S PTARTPM_Limit:30000 
14:28:17.632   
14:28:17.632  [CLK-I] [CAL131K]: delta:27 target:2441 PPM: 11061 PPM_Limit:30000 
14:28:17.632  [LOCKS-I] KM4 init_retarget_locks
14:28:17.632  [APP-I] BOR arises when supply voltage decreases under 2.57V and recovers above 2.7V.
14:28:17.632  [MAIN-I] KM4 MAIN 
14:28:17.632  [VER-I] AMEBA-RTOS SDK VERSION: 1.2.0
14:28:17.633  [MAIN-I] File System Init Success 
14:28:17.633  [MAIN-I] KM4 START SCHEDULER 
14:28:17.633  interface 0 is initialized
14:28:17.633  interface 1 is initialized
14:28:17.633  [WLAN-I] LWIP consume heap 1312
14:28:17.633  [WLAN-A] Init WIFI
14:28:17.633  [WLAN-A] Band=2.4G&5G
14:28:17.670  [WLAN-I] NP consume heap 20400
14:28:17.670  [WLAN-A] set ssid Xiaomi_Pro_2G_5G
14:28:17.814  [WLAN-A] start auth to 50:64:2b:34:88:9f
14:28:17.846  [WLAN-A] auth success, start assoc
14:28:17.879  [WLAN-A] assoc success(7)
14:28:18.000  [WLAN-A] set pairwise key 4(WEP40-1 WEP104-5 TKIP-2 AES-4 GCMP-15)
14:28:18.000  [WLAN-A] set group key 4 1
14:28:18.000  [WLAN-I] set cam: gtk alg 4 0
14:28:18.001  [$]wifi connected
14:28:19.136  [$]wifi got ip:"192.168.32.25"
14:28:19.136  wtn dhcp success
14:28:19.136  [WLAN-I] AP consume heap 12000
14:28:19.136  [WLAN-I] Available heap after wifi init 4527392
14:28:33.855  AT+WLDBG=lps_dtim_set 10
14:28:33.855  [WLDBG]: _AT_WLAN_IWPRIV_
14:28:33.855  [WLAN-A] [iwpriv_command] cmd name: lps_dtim_set
14:28:33.855  [WLAN-A] lps_dtim=10
14:28:33.855  
14:28:33.855  OK
14:28:33.855  
14:28:33.855  [MEM] After do cmd, available heap 4528000
14:28:33.855  
14:28:33.855  
14:28:33.855  #
14:28:42.288  AT+TICKPS=R
14:28:42.288  
14:28:42.288  [MEM] After do cmd, available heap 4528000
14:28:42.288  
14:28:42.288  
14:28:42.288  #
14:28:42.288  APPG
14:28:42.288  NPPG
14:28:43.136  NPPW
14:28:43.152  NPPG
14:28:43.248  NPPW
14:28:43.248  NPPG
14:28:43.344  NPPW
14:28:43.344  NPPG
14:28:43.440  NPPW
14:28:43.440  NPPG
14:28:43.552  NPPW
14:28:43.552  NPPG
14:28:43.648  NPPW
14:28:43.648  NPPG
14:28:43.759  NPPW
14:28:43.759  NPPG
14:28:43.856  NPPW
14:28:43.856  NPPG
14:28:43.952  NPPW
14:28:43.952  NPPG
14:28:44.064  NPPW
14:28:44.064  NPPG
14:28:44.159  NPPW
14:28:44.159  NPPG
14:28:44.272  NPPW
14:28:44.272  NPPG
14:28:44.368  NPPW
14:28:44.368  NPPG
14:28:44.464  NPPW
14:28:44.464  NPPG
14:28:44.576  NPPW
14:28:44.576  NPPG
14:28:44.672  NPPW
14:28:44.672  NPPG
14:28:44.784  NPPW
14:28:44.784  NPPG
14:28:44.880  NPPW
14:28:44.880  NPPG
14:28:44.975  NPPW
14:28:44.991  NPPG
14:28:45.088  NPPW
14:28:45.088  NPPG
14:28:45.183  NPPW
14:28:45.183  NPPG
14:28:45.296  NPPW
14:28:45.296  NPPG
14:28:45.392  NPPW
14:28:45.392  NPPG
14:28:45.488  NPPW
14:28:45.503  NPPG
14:28:45.599  NPPW
14:28:45.599  NPPG
14:28:45.696  NPPW
14:28:45.696  NPPG
14:28:45.807  NPPW
14:28:45.807  NPPG
14:28:45.903  NPPW
14:28:45.903  NPPG
14:28:45.999  NPPW
14:28:46.015  NPPG
14:28:46.111  NPPW
14:28:46.111  NPPG
14:28:46.207  NPPW
14:28:46.207  NPPG
14:28:46.320  NPPW
14:28:46.320  NPPG
14:28:46.415  NPPW
14:28:46.415  NPPG
14:28:46.511  NPPW
14:28:46.527  NPPG
14:28:46.623  NPPW
14:28:46.623  NPPG
14:28:46.719  NPPW
14:28:46.719  NPPG
14:28:46.831  NPPW
14:28:46.831  NPPG
14:28:46.927  NPPW
14:28:46.927  NPPG
14:28:47.022  NPPW
14:28:47.039  NPPG
14:28:47.136  NPPW
14:28:47.136  NPPG
14:28:47.231  NPPW
14:28:47.231  NPPG
14:28:47.343  NPPW
14:28:47.343  NPPG
14:28:47.439  NPPW
14:28:47.439  NPPG
14:28:47.535  NPPW
14:28:47.554  APPW
14:28:47.554  [PMC-I] SOCPS_KM0WKM4_ipc_int 
14:28:47.554  [PMC-I] FW wakeup KM4 via IPC 
14:28:47.554  APPG
14:28:47.554  APPW
14:28:47.554  APPG
14:28:47.570  NPPG
14:28:47.650  NPPW
14:28:47.650  NPPG


```


