<a id="chinese"></a>
<a id="english"></a>

**简体中文** | [English](README_EN.md)

# A1Z — 6-DOF 机械臂 Python SDK（可选 G1Z 夹爪）

<p align="center">
  <img src="assets/A1Z_G1Z.png" alt="A1Z 机械臂（带 G1Z 夹爪）" width="500"/>
</p>

A1Z 六轴机械臂的开源 Python SDK，面向机器人研发、算法验证和上层应用集成。项目提供跨平台 CAN 通信、电机控制、Pinocchio 动力学与运动学、重力补偿、零力示教、位置保持及可选 G1Z 夹爪控制。

代码仓库：[https://github.com/userguide-galaxea/galaxea_a1z](https://github.com/userguide-galaxea/galaxea_a1z)

## 快速导航

- [URDF 与 CAD](#urdf-与-cad)
- [功能特性](#功能特性)
- [平台与硬件支持](#平台与硬件支持)
- [安装](#安装)
- [快速开始](#快速开始)
- [CAN 通信故障排查](docs/can-troubleshooting.md#can-通信故障排查)
- [SDK 使用指南](docs/sdk-guide.md#zh)
- [更新日志](CHANGELOG.md#中文)
- [安全注意事项](#安全注意事项)
- [开发与贡献](#开发与贡献)

<a id="urdf-and-cad"></a>

## URDF 与 CAD

- 裸法兰、G1Z 夹爪及 G1Z 带腕部相机三种模型，附外观预览。[A1Z URDF](https://github.com/userguide-galaxea/robot_descriptions/tree/galaxea/main/a1z_description)

  <a href="https://raw.githubusercontent.com/userguide-galaxea/robot_descriptions/galaxea/main/a1z_description/a1z_g1z_description/previews/a1z_g1z.png"><img src="https://raw.githubusercontent.com/userguide-galaxea/robot_descriptions/galaxea/main/a1z_description/a1z_g1z_description/previews/a1z_g1z.png" alt="A1Z 带 G1Z 夹爪的 URDF 外观" width="420"></a>

- 基座、腕部相机支架、J6 输出法兰的 STEP/STP 模型，以及基座和法兰的 PDF 图纸。[A1Z CAD](https://github.com/userguide-galaxea/CAD_files/tree/main/A1Z)
- G1Z 夹指的标准版和加强筋版 STEP 模型，附外观预览。[G1Z 夹指 3D 打印件](https://github.com/userguide-galaxea/CAD_files/tree/main/G1Z)

| 标准版 | 加强筋版 |
| :---: | :---: |
| <a href="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print.png"><img src="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print.png" alt="标准版 G1Z 夹指" width="220"></a> | <a href="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print_reinforced.png"><img src="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print_reinforced.png" alt="加强筋版 G1Z 夹指" width="220"></a> |

## 功能特性

- **跨平台 CAN 后端**：Linux 使用 SocketCAN，macOS/Windows 可通过 `gs_usb` 用户态后端连接 Galaxea USB2CANFD 适配器，也可选 PEAK PCAN-USB。
- **六轴机械臂控制**：MotorA/MotorB 混合链、MIT 力位混控、250 Hz 默认控制循环。
- **动力学与运动学**：基于 Pinocchio 的 RNEA 重力补偿、FK 和 IK。
- **交互式示例**：重力补偿、位置保持、零力示教与回放、夹爪测试。
- **诊断与安全收尾**：CAN 检查、被动监听、电机扫描、错误码解析、中断后重复失能。
- **可配置夹爪模型**：在 `config.yaml` 中启用 G1Z 夹爪时自动选用对应的 URDF 质量模型。

## 平台与硬件支持

| 平台 | 默认后端 | 适配器/运行时 | 说明 |
|------|----------|----------------|------|
| Linux | SocketCAN | SocketCAN 兼容 CAN 接口 | 需配置 1 Mbps 接口；部分内核版本可能需要 `gs_usb` 补丁 |
| macOS | `gs_usb` | Galaxea USB2CANFD 适配器 | 用户态 USB 后端；当前版本已通过 250 Hz 真机控制验证 |
| Windows | `gs_usb` | Galaxea USB2CANFD 适配器+ libusb 兼容驱动 | 先使用现有驱动；无法打开时再考虑 Zadig/WinUSB |
| macOS / Windows | `pcan` | PEAK PCAN-USB + 官方运行时 | 通过 `--bustype pcan` 显式选择 |
| macOS / Windows / Linux | `cansub` | CSS Electronics CANsub（USB 或以太网） | 通过 `--bustype cansub` 显式选择 |

### 验证层级

为避免把“适配器可打开”误解为“机械臂可安全控制”，本项目将验证分为三层：

1. **适配器层**：USB-CAN 设备可识别、打开并提交发送请求。Windows 可用 `tools/verify_can_win.py`。
2. **电机通信层**：机械臂已上电、接线和终端电阻正确，所有关节能返回有效反馈。使用 `tools/motor_diag.py --scan`。
3. **控制层**：在真实负载下验证频率、丢帧/延迟、稳定性、限位和重力补偿方向。该层必须在本地硬件上完成，前两层通过不代表本层通过。


## 项目结构

```
a1z/
├── pyproject.toml                 # 构建配置 (flit)
├── setup.py                       # setuptools 后备
├── README.md
├── LICENSE                        # MIT 开源许可
├── .python-version                # 统一的 Python 3.11 环境提示
├── a1z/                       # SDK 主包
│   ├── dynamics/
│   │   └── gravity_model.py       # Pinocchio RNEA 重力补偿
│   ├── motor_drivers/
│   │   ├── can_backend.py         # 跨平台 CAN 后端选择（Linux/macOS/Windows 自动适配）
│   │   ├── can_interface.py       # CAN 总线封装
│   │   ├── motor_a_driver.py      # MotorA 驱动 (MIT 混控)
│   │   ├── motor_b_driver.py      # MotorB 驱动 + MixedMotorChain
│   │   └── utils.py               # 数据结构, float↔uint 转换
│   ├── robots/
│   │   ├── robot.py               # Robot Protocol (抽象接口)
│   │   ├── arm_robot.py           # ArmRobot 实现 (控制回路+重力补偿+安全状态机)
│   │   ├── get_robot.py           # 工厂函数 get_a1z_robot()
│   │   ├── gripper.py             # Gripper 控制 (MotorB CAN ID 0x07)
│   │   ├── server.py              # 跨平台本机控制服务端（Unix socket / Windows TCP）
│   │   └── kinematics.py          # FK/IK (Pinocchio)
│   ├── robot_models/
│   │   └── a1z/               # URDF 模型文件 (A1Z_Flange.urdf 默认，A1Z_G1Z.urdf 用于夹爪配置)
│   └── utils/
│       └── utils.py               # RateRecorder, 日志工具
├── examples/
│   ├── gravity_comp.py            # 重力补偿示例
│   ├── position_hold.py           # 位置保持示例
│   ├── teach_and_play.py          # 零力示教录制与回放
│   └── gripper_hybrid_test.py     # 夹爪测试：自由行程 + 力矩饱和验证
└── tools/
    ├── a1zctl                     # 机械臂控制 CLI（serve/move/gripper/dance/stop）
    ├── gripper_set_zero.py        # 夹爪零点标定（出厂已完成，一般无需执行）
    ├── motor_diag.py              # 电机通信诊断与故障排查
    ├── set_zero.py                # 电机零点标定
    ├── setup_can.sh               # Linux SocketCAN 一键配置
    ├── verify_can_mac.sh          # macOS CAN 通信验证
    └── verify_can_win.py          # Windows CAN 通信验证
```


## 安装

### 依赖

- Python >= 3.11（所有平台统一支持基线）
- **Linux**：SocketCAN（需硬件 CAN 接口）
- **macOS / Windows**：通过 Galaxea USB2CANFD 适配器连接，SDK 自动使用 gs_usb 用户态后端
- URDF 模型文件随 SDK 提供，并从 [robot_descriptions](https://github.com/userguide-galaxea/robot_descriptions/tree/galaxea/main/a1z_description) 同步。默认使用 `A1Z_Flange.urdf`，启用夹爪时使用 `A1Z_G1Z.urdf`。

> 跨平台支持详情见 [跨平台 CAN 配置](#跨平台-can-配置) 章节。

### 安装 SDK

```bash

# 克隆 SDK
git clone --branch main https://github.com/userguide-galaxea/galaxea_a1z.git GALAXEA-A1Z

cd GALAXEA-A1Z

# Linux/macOS 使用 Python 3.11 创建虚拟环境；Windows 请使用下文的 Conda 流程
python3.11 -m venv .venv
source .venv/bin/activate

# 开发模式安装（推荐）
python -m pip install -e .

# 或直接安装
python -m pip install .
```

依赖会自动安装：`numpy`、`python-can>=4.0`、`pyyaml`，Linux/macOS 还会通过 pip 安装 `pin`（Pinocchio）。Windows 上不由 pip 处理 `pin`，请按下文使用 Conda 安装 `pinocchio`。

macOS / Windows 用户请改用以下命令安装（含 gs_usb 用户态后端和 libusb 运行时）：

```bash
python -m pip install -e ".[gs_usb]"
```

> 注意：macOS 默认的 zsh 会把 `[...]` 当作通配符并报 `no matches found`，所以引号不能省。Windows 还需要允许 libusb 访问设备的驱动；现有驱动可用时不要强制替换。

### 配置文件

示例、诊断和标定工具及 `a1zctl serve` 会自动读取仓库根目录的 `config.yaml`。请在运行前编辑此文件。

- `with_gripper: false`（默认）：无夹爪，使用 `A1Z_Flange.urdf`。
- `with_gripper: true`：启用 G1Z 夹爪，自动选用 `A1Z_G1Z.urdf`。

```yaml
can_channel: can0
control_freq_hz: 250
gravity_comp_factor: 1.0
with_gripper: false
```

| 参数 | 说明 |
|---|---|
| `gripper_max_torque` | G1Z 最大夹持力矩，单位 N·m。 |
| `urdf_path` | 取消注释并填写路径，可覆盖按夹爪配置自动选用的 URDF。 |
| `can_channel`, `bustype` | CAN 接口和后端。后端可选 `socketcan`、`gs_usb`、`pcan`、`slcan`、`cansub`。自动选择时，Linux 用 `socketcan`，macOS 和 Windows 用 `gs_usb`。 |
| `motor_a_use_new_enable_protocol` | 仅在确认 MotorA 固件支持 `0x7FF` 使能协议时设为 `true`。 |
| `control_freq_hz`, `min_freq_hz` | 控制频率和低频保护阈值，单位 Hz。频率持续低于阈值会触发保护停机。 |
| `gravity_comp_factor` | 重力补偿系数，`0` 关闭，`1` 全额补偿。 |
| `zero_gravity_mode` | `true` 为零力漂浮，`false` 为位置保持。省略时按命令默认模式运行。 |
| `default_kp`, `default_kd` | 按 J1–J6 排列的位置和速度增益。取消对应行的注释并修改数值，可覆盖[代码默认值](#默认控制参数)。 |

使用示例：

> [!CAUTION]
> 首次运行前，请固定机械臂、清空工作空间，并确保物理急停可触及。

```bash
python examples/position_hold.py
python tools/a1zctl serve
python tools/motor_diag.py --scan
python tools/set_zero.py --all
```

### 配置 CAN 总线（SocketCAN 模式）

注意：检查can盒电阻是否正确安装！

使用 Galaxea USB2CANFD 适配器：

```bash
# 1. 加载驱动
sudo modprobe gs_usb

# 2. 确认接口出现（单适配器通常为 can0）
ip link show type can

# 3. 配置并启动（1 Mbps）
sudo ip link set can0 type can bitrate 1000000
sudo ip link set can0 up
```

如果没有出现 CAN 接口，参见 [Linux 驱动排障](docs/can-troubleshooting.md#3-linux检查内核版本并修复)。

#### macOS（gs_usb 用户态模式）

macOS 没有 SocketCAN，SDK 会自动检测平台并使用 gs_usb 用户态后端连接 Galaxea USB2CANFD 适配器，**无需手动配置 CAN 接口**。

```bash
# 1. 安装 SDK + gs_usb extras（含内置 libusb，通常无需 brew）
python -m pip install -e ".[gs_usb]"

# 2. 验证 CAN 通信（可选）
bash tools/verify_can_mac.sh
```

#### Windows（gs_usb 用户态模式）

Windows 同样走 gs_usb 用户态后端。`libusb-package` / `libusb1` 由 `.[gs_usb]` extras 安装。如果适配器现有驱动已允许 libusb 访问，无需使用 Zadig；只有在设备无法打开且确认是驱动不兼容时，才需要将设备绑定到 WinUSB。

```powershell
# 0. 如果当前终端中文乱码，切换为 UTF-8（推荐使用 Windows Terminal）
chcp 65001

# 1. 创建并激活独立 Conda 环境
# 项目统一使用 Python 3.11
conda create -n a1z python=3.11 -y
conda activate a1z

# 2. 用 Conda 安装 Pinocchio（Windows 包名为 pinocchio）
conda install -c conda-forge pinocchio libusb -y

# 3. 在同一环境中安装 SDK + gs_usb extras
python -m pip install -e ".[gs_usb]"

# 4. 确认 python/pip 都指向当前 Conda 环境，并检查依赖
where.exe python
python --version
python -m pip --version
python -c "import pinocchio, can, usb, usb1, gs_usb; print('Environment OK')"

# 5. 验证 USB-CAN 适配器能否打开和发送
python tools\verify_can_win.py
```

#### 跨平台 CAN 配置

SDK 默认按平台自动选择 CAN 后端：

| 平台 | 适配器 | 安装 | 使用 |
|------|--------|------|------|
| Linux | SocketCAN 内核驱动 | `python -m pip install -e .` | `--can can0` |
| **macOS** | Galaxea USB2CANFD 适配器 | `python -m pip install -e ".[gs_usb]"` | （默认）`gs_usb` |
| **macOS** | **PEAK PCAN-USB** | `python -m pip install -e ".[pcan]"` + MacCAN PCBUSB 运行时 | `--bustype pcan --can PCAN_USBBUS1` |
| **Windows** | Galaxea USB2CANFD 适配器 | `python -m pip install -e ".[gs_usb]"`；仅驱动不兼容时使用 Zadig/WinUSB | （默认）`gs_usb` |
| **Windows** | **PEAK PCAN-USB** | `python -m pip install -e ".[pcan]"` + PEAK 官方驱动 | `--bustype pcan --can PCAN_USBBUS1` |
| macOS / Windows / Linux | CSS Electronics CANsub | `python -m pip install -e ".[cansub]"`，无需驱动 | `--bustype cansub --can aabbccdd-usb.local@1` |

不传 `--bustype` 时使用 OS 默认值：Linux 默认 `socketcan`/`can0`，macOS/Windows 默认 `gs_usb`/设备序号 `0`。

## 快速开始

### 使用 example 脚本

> 装有 G1Z 夹爪时，在 `config.yaml` 中设置 `with_gripper: true`，使重力补偿计入夹爪质量。

```bash
# 首次调试从较低的重力补偿系数开始
python examples/gravity_comp.py --gravity_factor 0.3

# 确认补偿方向正确后提升到全补偿
python examples/gravity_comp.py --gravity_factor 1.0

# 位置保持模式
python examples/gravity_comp.py --mode hold

# 位置保持 + 低速移动到目标（with_gripper: false）
python examples/position_hold.py --q_target_deg 0,30,-20,-15,0,0 --speed 0.5
```

> ⚠️ **如果上述命令报错或机械臂无反应**，请跳转到 [CAN 通信故障排查](docs/can-troubleshooting.md#can-通信故障排查)。

更多用法  [零力示教与回放](docs/sdk-guide.md#零力示教与回放) · [夹爪](docs/sdk-guide.md#夹爪) · [a1zctl](docs/sdk-guide.md#使用-a1zctl-服务端可用于openclaw交互) · [API 参考](docs/sdk-guide.md#api-参考)

## 工具

### 电机通信诊断与故障排查

```bash
# 检查 CAN 接口是否正常
python tools/motor_diag.py --check-can

# 扫描机械臂电机（with_gripper 为 true 时包含夹爪）
python tools/motor_diag.py --scan

# 详细探测某个关节（完整收发流程 + 反馈解析）
python tools/motor_diag.py --probe 3

# 持续监控所有电机状态（位置/速度/温度/错误码）
python tools/motor_diag.py --monitor

# 被动监听 CAN 总线（不发任何指令，用于排查总线冲突）
python tools/motor_diag.py --listen --duration 10

# 清除 MotorB 错误码
python tools/motor_diag.py --clear-error
python tools/motor_diag.py --clear-error --joints 3 4
```

> ⚠️ `--scan`、`--monitor` 和 `--probe` 不是被动诊断：它们会使能电机并发送控制帧，机械臂可能短暂动作。脚本会要求输入 `YES` 进行确认。执行前请清空周围空间、稳定支撑机械臂、确保急停可触及，运行期间不要接触机械臂。自动化环境只能在已完成等价安全检查时使用 `--yes`。

诊断脚本会自动检测并给出常见问题的排查建议：
- 电机无响应（未上电 / CAN 线反接 / ID 错误 / 固件模式）
- MotorB 错误码（过压/欠压/过流/过温/通信丢失/过载）
- CAN 总线异常（bus-off / error-passive / 重启次数）
- 温度预警

### 电机零点标定

```bash
# 标定所有电机（当前位置设为零点）
sudo python tools/set_zero.py --all

# 标定指定关节
sudo python tools/set_zero.py --joints 0 3
```

## 控制原理

### MIT 力位混控

每个控制周期，SDK 通过 `send_mit_command` 向电机下发五元组（`pos`, `vel`, `kp`, `kd`, `torque`），电机固件在内部执行 PD + 前馈合力：

```python
# motor_a_driver.py / motor_b_driver.py — send_mit_command
def send_mit_command(self, pos: float, vel: float, kp: float, kd: float, torque: float) -> None:
    pos_u16    = float_to_uint(pos,    r.pos_min,    r.pos_max,    16)
    vel_u12    = float_to_uint(vel,    r.vel_min,    r.vel_max,    12)
    kp_u12     = float_to_uint(kp,     r.kp_min,     r.kp_max,     12)
    kd_u9      = float_to_uint(kd,     r.kd_min,     r.kd_max,      9)
    torque_u12 = float_to_uint(torque, r.torque_min, r.torque_max, 12)
    # 打包成 8 字节 CAN 帧下发
```

SDK 在每个控制周期（默认 250 Hz）的 `_update` 中执行（发送路径位于 `_send_command_and_cache_hold_locked`）：

```python
# arm_robot.py — _update()

# 1. 读取电机反馈，并做每周期安全检查（反馈新鲜度 / 电机错误 / 温度 / 速度）
self._read_state()
feedback_fault = self._check_runtime_safety()
# 反馈失效时：先重发缓存的纯重力补偿 MIT 保持帧，再抛可恢复故障（见"故障策略"）；
# 过温 / 过速 / 电机错误等硬故障直接失能电机

# 2. Pinocchio RNEA 逆动力学（重力 + 科氏力 + 惯性）
tau_id = self._gravity_model.compute_inverse_dynamics(q, cmd.vel, cmd.acc)

# 3. 合成最终扭矩（PD + 重力补偿：torque_ff + 逆动力学补偿）
torques_urdf  = cmd.torque_ff + tau_id * self._gravity_torque_scale * self.gravity_comp_factor
motor_torques = np.clip(torques_urdf * self._joint_sign, -self._torque_clip, self._torque_clip)

# 4. 经电机链下发，并缓存纯重力补偿保持帧供反馈失效时重发
self._motor_chain.send_commands(
    pos=cmd.pos * self._joint_sign,
    vel=cmd.vel * self._joint_sign,
    kp=cmd.kp,
    kd=cmd.kd,
    torque=motor_torques,
)
```

### 零力漂浮模式

```python
# get_a1z_robot(zero_gravity_mode=True) 启动时初始化：
self._command.kp = np.zeros(self._num_joints)        # 无位置刚度
self._command.kd = self._default_kd.copy() * 0.5    # 小阻尼
# 仅靠 tau_g 抵消重力，机械臂可自由拖拽
```

### 位置保持模式

```python
# get_a1z_robot(zero_gravity_mode=False) 启动时初始化：
self._command.kp = self._default_kp.copy()  # [146.8988, 62.9454, 89.2416, 120.0, 40.0, 100.0]
self._command.kd = self._default_kd.copy()  # [5.0, 5.0, 5.0, 2.0776, 1.5059, 1.2553]
# PD 控制 + 重力补偿，锁定到当前位置
```

## 安全注意事项


- 请将机械臂牢固安装在稳定基座上，清空其全部可达空间，并确保操作者可随时按下物理急停。
- 装有 G1Z 夹爪时，在 `config.yaml` 中设置 `with_gripper: true`，使动力学模型包含夹爪质量。
- `--scan`、`--monitor`、`--probe`、示例脚本和 `a1zctl` 都可能使电机动作；不要在机械臂工作空间内停留。
- 重力扭矩阈值、软限位和平滑停机只是软件保护，不能替代物理限位、防护围栏、急停回路或风险评估。
- 通信中断、程序异常或行为不符合预期时，立即使用物理急停，然后确认所有电机已失能。
- 不要将未经本地硬件验证的脚本用于无人值守、生产环境或与人直接协作的场景。
- SDK 由显式安全状态机管理急停与故障行为（见下文"安全状态机"与"故障策略"）
- 反馈丢失 / 总线瞬态错误时自动重发缓存的安全保持帧并继续尝试恢复；持续 0.2 s 未恢复则锁存位置保持；过温 / 过速 / 电机错误直接失能电机
- 所有目标关节角都会被裁剪到 URDF 限位范围内（见下文"关节限位处理"）

### 安全状态机

控制回路运行在显式状态机上：`STOPPED` / `RUNNING` / `SOFT_ESTOP` / `COMMAND_HOLD` / `FAULT_HOLD` / `HARD_DISABLED` / `HARD_DISABLE_UNCONFIRMED`。

- `start()` 仅允许从 `STOPPED` 状态启动（无存活控制线程、无未清除的致命故障），否则抛 `RuntimeError`。进入 `HARD_DISABLED` / `HARD_DISABLE_UNCONFIRMED` 后对象不可重启，必须重建 `ArmRobot` 实例。
- `estop()` 为软急停：kp 清零、kd 减半、命令位置锁定到当前测量位置；**电机保持使能**，重力补偿继续运行，机械臂在负载下不会坠落。从 `STOPPED` / `FAULT_HOLD` / `HARD_*` 状态调用返回 `False`。
- `estop()` / `release()` 均返回 `bool`。`release(expected_fault_code=None)` 仅在控制回路运行、反馈新鲜完整、状态匹配（`SOFT_ESTOP` / `COMMAND_HOLD` / `FAULT_HOLD`）时恢复 `RUNNING`；否则返回 `False` 并保持现状。`expected_fault_code` 可防止误释放其他来源的故障。
- 急停 / 保持期间，`command_joint_pos` / `command_joint_state` / `command_gripper` / `move_joints` 被拒绝（返回 `False` 或提前返回，并记日志）。
- `stop()` 停止控制回路并失能所有电机；**不再关闭 CAN 总线**——总线在对象析构时关闭（或由外部调用方管理），因此 `stop()` 后可用同一对象再次 `start()`。
- `get_fault_status()` 返回线程安全的状态快照（`state` / `code` / `reason` / 故障时长 / 逐关节反馈健康度 / `restart_allowed` 等）。

### 故障策略

| 分类 | 触发条件 | 行为 |
|------|----------|------|
| 可恢复（recoverable） | 反馈超时（stale）、瞬态 CAN 发送错误（EAGAIN/EWOULDBLOCK/EINTR/ENOBUFS/TX 缓冲区满）、逆动力学计算异常 | 保持最后一次命令并重发缓存的纯重力补偿 MIT 保持帧（电机**不**失能）；持续 0.2 s 未恢复 → 锁存 `FAULT_HOLD`（全增益位置保持），反馈恢复后可 `release()` |
| 硬故障（hard） | 过温、过速、电机错误码、非有限反馈 | 立即失能所有电机（`HARD_DISABLED`），锁存故障且对象不可重启 |

### 关节限位处理

- 流式命令（`command_joint_pos` / `command_joint_state`，teleop / 轨迹回放场景）：越界目标**始终裁剪**到物理限位，不再拒绝整帧或触发急停，未越界关节继续跟踪。裁剪前按 S¹ 圆拓扑将目标 unwrap 到距当前命令最近的等价角，避免 ±π 回绕导致满行程跳变。超出每关节容差（默认 0.05 rad）的大越界记 ERROR 日志，容差内小越界记 WARNING。
- `move_joints` 单发运动：目标越界超出容差仍抛 `ValueError` 拒绝执行；新增 `max_jump_rad` 参数，可拒绝距当前测量位置过远的目标（如 IK 翻肘解）。
- 运行中测量位置漂出软限位只记日志（1 Hz 限频），不再触发急停。

## 关节限位

| 关节 | 名称 | 机械限位 (°) | 机械限位 (rad) | 软限位 (°) | 软限位 (rad) |
|------|------|-------------|--------------|-----------|-------------|
| 0 | arm_joint1 | [-130°, 130°] | [-2.269, 2.269] | [-120°, 120°] | [-2.094, 2.094] |
| 1 | arm_joint2 | [-1.94°, 192.78°] | [-0.034, 3.365] | [0°, 180°] | [0.000, 3.142] |
| 2 | arm_joint3 | [-200.38°, 0°] | [-3.497, 0.000] | [-180°, 0°] | [-3.142, 0] |
| 3 | arm_joint4 | [-91.88°, 110.38°] | [-1.604, 1.926] | [-85°, 85°] | [-1.484, 1.484] |
| 4 | arm_joint5 | [-90°, 90°] | [-1.571, 1.571] | [-85°, 85°] | [-1.484, 1.484] |
| 5 | arm_joint6 | [-120°, 120°] | [-2.094, 2.094] | [-115°, 115°] | [-2.007, 2.007] |

## 默认控制参数

| 参数 | 值 |
|------|------|
| 默认 KP | `[146.8988, 62.9454, 89.2416, 120.0, 40.0, 100.0]` |
| 默认 KD | `[5.0, 5.0, 5.0, 2.0776, 1.5059, 1.2553]` |
| 命令增益上限 | kp ≤ 200，kd ≤ 5（`command_joint_state` / `move_joints` 自定义增益越界拒绝） |
| 关节坐标系符号 | `[1, 1, -1, 1, -1, 1]` (关节3，5与URDF方向相反) |
| 重力扭矩缩放 | `[1, 1, 1, 1, 1, 1]` |
| 最大重力扭矩 | `[50, 50, 50, 24, 10, 10]` Nm |
| 扭矩限幅 | `[70, 70, 70, 27, 10, 10]` Nm |
| MotorA KT | 2.8 (电流→扭矩转换系数) |
| 控制频率 | 250 Hz |

## 开发与贡献

欢迎提交 Issue 和 Pull Request（PR）。建议使用小而可审查的变更，并将平台差异、硬件假设和安全影响写清楚。

提交 PR 前请确认：

- 变更聚焦于单一问题，不包含无关的格式化或生成文件。
- 无法自动测试的硬件路径需说明完整的人工验证步骤和结果。
- 修改公开 API、CLI、默认参数、配置文件或平台支持时，同步更新中英文 README。
- 不要仅根据“测试帧发送成功”宣称电机通信或控制稳定性已验证。

## 获取帮助与报告问题

报告问题前，请先查阅 [CAN 通信故障排查](docs/can-troubleshooting.md#can-通信故障排查) 并搜索现有 [GitHub Issues](https://github.com/userguide-galaxea/GALAXEA-A1Z/issues)。新 Issue 建议包含：

- 操作系统及版本、Python 版本、仓库分支/提交号。
- CAN 适配器型号和 VID:PID、后端类型、通道及波特率。
- 机械臂是否带 G1Z 夹爪、使用的配置文件和完整命令。
- 失败所在的验证层级、完整错误输出、可稳定复现的最小步骤。
- 硬件供电、CANH/CANL 接线和终端电阻的检查结果。

公开日志前请删除用户名、本地路径、设备序列号及其他敏感信息。

## 开源许可

本项目基于 [MIT License](LICENSE) 开源，版权归 **星海图** 所有。

### 第三方依赖许可

| 依赖 | 许可证 | 说明 |
|------|--------|------|
| [numpy](https://numpy.org) | BSD-3-Clause（发行包还包含其他第三方许可组件） | 数值计算 |
| [python-can](https://github.com/hardbyte/python-can) | LGPL-3.0 | CAN 总线通信 |
| [pinocchio (pin)](https://github.com/stack-of-tasks/pinocchio) | BSD-2-Clause | 机器人动力学计算 |
| [PyYAML](https://github.com/yaml/pyyaml) | MIT | YAML 配置文件解析 |

本项目的 MIT License 适用于本仓库自身的代码和文档。第三方库及可选 CAN 运行时仍受各自许可证约束；分发产品前请核对实际锁定版本的上游许可证文本和合规要求。
