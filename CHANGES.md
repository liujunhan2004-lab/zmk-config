# Ferris Sweep 键盘配置档案

本仓库从 [urob/zmk-config](https://github.com/urob/zmk-config) 派生,适配到本人的
**Ferris Sweep 34 键分体键盘**。本文件记录键盘的基础情况、已做改动与关键决定,
作为这份配置的说明档案。

---

## 一、键盘基础情况

### 硬件

| 项目 | 说明 |
|---|---|
| 键盘 | Ferris Sweep(也称 Cradio),34 键分体低轮廓键盘,左右各 17 键 |
| 键位形状 | 每半 5 列 × 3 行 + 2 拇指键 = 17 键 |
| 主控 | nice!nano 系列(nRF52840,ProMicro 外形,蓝牙) |
| 开关 | 低轮廓(Choc)热插拔 |

### ZMK 构建标识

| 项目 | 值 |
|---|---|
| board | `nice_nano` |
| shield | `cradio_left` / `cradio_right` |

> 34 键 = urob `base.keymap` 的基准尺寸,因此**无需键盘适配器**,直通 fallback 直接铺开。

### 采用的键位布局

urob 的 Colemak-DH 布局,共 6 层(Base / Nav / Fn / Num / Sys / Mouse),核心特性:

- **主行 mod(HRM)** — `balanced` flavor + `require-prior-idle-ms` + 位置 hold-tap,近乎"无定时"、低误触
- **组合键代替符号层** — 所有符号通过 combo 输入(`combos.dtsi`)
- **智能层** — Numword(数字自动激活/退出)、Smart-mouse(W+P 组合触发)
- **魔法拇指键** — 一键四用:Repeat / 粘滞 Shift / Shift / Caps Word
- **鼠标层** — 右半按键模拟鼠标移动/滚轮/按键(`mouse.dtsi`,需 `CONFIG_ZMK_POINTING=y`)

---

## 二、已做改动

### 1. `config/cradio.keymap`(新建)

```c
#define CONFIG_WIRELESS
#include "zmk-helpers/key-labels/34.h"
#include "base.keymap"
```

### 2. `config/cradio.conf`(新建)

```conf
# Wireless split keyboard settings (Ferris Sweep / cradio shield)

# Sleep after 30 minutes of inactivity
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000

# Enable pointing for the smart-mouse layer
CONFIG_ZMK_POINTING=y

# Bluetooth tweaks
CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y
CONFIG_BT_GATT_ENFORCE_SUBSCRIPTION=n
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
```

### 3. `build.yaml`(修改 `include` 构建矩阵)

之前:

```yaml
include:
  - board: planck@6.0.0//zmk
  - board: corneish_zen_left@2.0.0//zmk
  - board: corneish_zen_right@2.0.0//zmk
  - board: glove80_lh
  - board: glove80_rh
```

之后:

```yaml
include:
  - board: nice_nano//zmk
    shield: cradio_left
  - board: nice_nano//zmk
    shield: cradio_right
  - board: nice_nano//zmk
    shield: settings_reset
```

### 4. 精简 leader 键(后续改动)

- 删除 leader 键与 Unicode 输入:移除 `zmk-leader-key`、`zmk-unicode` 两个模块,
  删除 `config/leader.dtsi`
- `S+T` 组合键改为**全选**(`Ctrl+A`),`R+S+T` 改为**任务管理器**(`Ctrl+Shift+Esc`)
- Sys 层右半顶行最左(J 键)新增 `&out OUT_TOG`:**一键切换 USB/蓝牙输出**

---

## 三、关键决定

### 不加 `CONFIG_ZMK_PM_SOFT_OFF`(软关机)

- 无实质影响:仅缺少"按键软关机"能力,仍可用**物理电源开关** + **30 分钟自动休眠**
  替代
- urob 布局的 Sys 层本来也没有绑定 `&soft_off` 键,开了也无法触发

### 不加 `CONFIG_ZMK_STUDIO`(ZMK Studio)

- 无实质影响:固件照常工作,改键仍走"改代码 → 重新构建刷机"
- urob 布局重度依赖自定义宏(combos/adaptive-key/tri-state 等),
  ZMK Studio 对这些支持有限
- 若要启用还需额外两处改动(`build.yaml` 加 snippet + workflow 改 `zephyr-full`),
  会显著拉长构建时间,故不启用

### board 名沿用 `nice_nano`

- 与本人旧固件(NXTKB / sky-bro 配置)一致,刷机行为不变
- 若主控实为 nice!nano **v2**,可将 board 名改为 `nice_nano_v2`(一行)

---

## 四、构建与刷机

本地 nix 构建环境仅支持 Linux/macOS(Windows 不可用),因此走 **GitHub Actions 云构建**:

1. 推到自己的 GitHub fork
2. 在 fork 的 Actions 页开启一次 Actions(新 fork 默认禁用)
3. push 触发构建,下载 `cradio_left` / `cradio_right` 两个 `.uf2`
4. 先刷 `settings_reset` 清配对,再刷左右两半新固件
   (nice!nano 双击 reset 进入 `NICENANO` 盘后拖入 .uf2)

---

## 五、其他说明

- 原 urob 的 `config/corneish_zen.*`、`config/glove80.*`、`config/planck_rev6.*`
  已不被 `build.yaml` 引用,闲置无害,可保留作参考或删除。

## 六、外部工具依赖(Windows 下需额外安装)

本布局绝大部分功能内置在固件里,仅以下一个功能依赖外部软件,只需安装运行、无需改配置:

| 工具 | 对应功能 | 仓库 |
|---|---|---|
| win-11-virtual-desktop-enhancer | Fn 层桌面管理 5 键(PDesk/NDesk/PinW/PinA/DSK_MGR) | github.com/urob/win-11-virtual-desktop-enhancer |

- 桌面工具仓库已带预编译 `virtual-desktop-enhancer.exe`,直接运行即可。
- 不装它,仅桌面键失效,其余功能不受影响。
