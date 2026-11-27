<a id="english"></a>

[简体中文](README.md) | **English**

# A1Z — 6-DOF Robotic Arm Python SDK (Optional G1Z Gripper)

<p align="center">
  <img src="assets/A1Z_G1Z.png" alt="A1Z robotic arm with G1Z gripper" width="500"/>
</p>

An open-source Python SDK for A1Z six-axis robotic-arm research, algorithm validation, and application integration. It provides cross-platform CAN communication, motor control, Pinocchio-based dynamics and kinematics, gravity compensation, zero-force teaching, position hold, and optional G1Z gripper control.

Repository: [github.com/userguide-galaxea/GALAXEA-A1Z](https://github.com/userguide-galaxea/GALAXEA-A1Z)

## Quick Navigation

- [URDF and CAD](#urdf-and-cad)
- [Features](#features)
- [Platform and Hardware Support](#platform-and-hardware-support)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [CAN Communication Troubleshooting](docs/can-troubleshooting.md#can-communication-troubleshooting)
- [SDK Guide](docs/sdk-guide.md#en)
- [Changelog](CHANGELOG.md#english)
- [Safety](#safety)
- [Development and Contributing](#development-and-contributing)

## URDF and CAD

- Bare-flange, G1Z gripper, and G1Z wrist-camera variants with model previews. [A1Z URDF](https://github.com/userguide-galaxea/robot_descriptions/tree/galaxea/main/a1z_description)

  <a href="https://raw.githubusercontent.com/userguide-galaxea/robot_descriptions/galaxea/main/a1z_description/a1z_g1z_description/previews/a1z_g1z.png"><img src="https://raw.githubusercontent.com/userguide-galaxea/robot_descriptions/galaxea/main/a1z_description/a1z_g1z_description/previews/a1z_g1z.png" alt="A1Z URDF exterior with G1Z gripper" width="420"></a>

- STEP/STP models for the base, wrist camera mount, and J6 output flange, plus PDF drawings for the base and flange. [A1Z CAD](https://github.com/userguide-galaxea/CAD_files/tree/main/A1Z)
- Standard and rib-reinforced G1Z gripper finger STEP models with previews. [G1Z Gripper Fingers for 3D Printing](https://github.com/userguide-galaxea/CAD_files/tree/main/G1Z)

| Standard | Rib-reinforced |
| :---: | :---: |
| <a href="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print.png"><img src="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print.png" alt="Standard G1Z gripper finger" width="220"></a> | <a href="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print_reinforced.png"><img src="https://raw.githubusercontent.com/userguide-galaxea/CAD_files/main/G1Z/previews/g1z_gripper_finger_3d_print_reinforced.png" alt="Rib-reinforced G1Z gripper finger" width="220"></a> |

## Features

- **Cross-platform CAN backends**: SocketCAN on Linux; `gs_usb` userspace support for the Galaxea USB2CANFD adapter on macOS/Windows; optional PEAK PCAN-USB support.
- **Six-axis arm control**: mixed MotorA/MotorB chains, MIT position-velocity-torque control, and a default 250 Hz control loop.
- **Dynamics and kinematics**: Pinocchio-based RNEA gravity compensation, forward kinematics, and inverse kinematics.
- **Runnable examples**: gravity compensation, position hold, zero-force teach-and-playback, and gripper testing.
- **Diagnostics and safe cleanup**: CAN checks, passive listening, motor scans, error-code decoding, and repeated disable on exit.
- **Gripper-aware configuration**: Set `with_gripper: true` in `config.yaml` to enable the G1Z gripper and select the matching URDF mass model.

## Platform and Hardware Support

| Platform | Default backend | Adapter/runtime | Notes |
|----------|-----------------|-----------------|-------|
| Linux | SocketCAN | SocketCAN-compatible CAN interface | Configure the interface at 1 Mbps; some kernels may require a `gs_usb` patch |
| macOS | `gs_usb` | Galaxea USB2CANFD adapter | Userspace USB backend; the current version has passed 250 Hz hardware control validation |
| Windows | `gs_usb` | Galaxea USB2CANFD adapter + libusb-compatible driver | Try the existing driver first; use Zadig/WinUSB only if the device cannot be opened |
| macOS / Windows | `pcan` | PEAK PCAN-USB + vendor runtime | Select explicitly with `--bustype pcan` |
| macOS / Windows / Linux | `cansub` | CSS Electronics CANsub (USB or Ethernet) | Select explicitly with `--bustype cansub` |

### Validation Levels

To avoid treating “the adapter opens” as “the arm is safe to control,” validation is divided into three levels:

1. **Adapter level**: the USB-CAN device can be discovered, opened, and given a send request. On Windows, use `tools/verify_can_win.py`.
2. **Motor-communication level**: the arm is powered, wiring and termination are correct, and every joint returns valid feedback. Use `tools/motor_diag.py --scan`.
3. **Control level**: verify loop frequency, loss/latency, stability, joint limits, and gravity-compensation direction under the real payload. This level must be completed on the local hardware; passing the first two levels does not imply that it has passed.

Current hardware-validation status (2026-09):

| Platform/configuration | Result |
|------------------------|--------|
| macOS + Galaxea USB2CANFD + asynchronous userspace backend | 250 Hz hardware control validated both without a gripper and with the G1Z gripper enabled |
| Windows + Galaxea USB2CANFD | 250 Hz hardware gravity-compensation control validated both without a gripper and with the G1Z gripper enabled |

## Project Structure

```
a1z/
├── pyproject.toml                 # Build config (flit)
├── setup.py                       # setuptools fallback
├── README.md
├── LICENSE                        # MIT open-source license
├── .python-version                # Shared Python 3.11 environment hint
├── a1z/                       # SDK main package
│   ├── dynamics/
│   │   └── gravity_model.py       # Pinocchio RNEA gravity compensation
│   ├── motor_drivers/
│   │   ├── can_backend.py         # Cross-platform CAN backend selection (Linux/macOS/Windows auto-adaptation)
│   │   ├── can_interface.py       # CAN bus wrapper
│   │   ├── motor_a_driver.py      # MotorA driver (MIT mixed control)
│   │   ├── motor_b_driver.py      # MotorB driver + MixedMotorChain
│   │   └── utils.py               # Data structures, float↔uint conversion
│   ├── robots/
│   │   ├── robot.py               # Robot Protocol (abstract interface)
│   │   ├── arm_robot.py           # ArmRobot implementation (control loop + gravity comp + safety state machine)
│   │   ├── get_robot.py           # Factory function get_a1z_robot()
│   │   ├── gripper.py             # Gripper control (MotorB CAN ID 0x07)
│   │   ├── server.py              # Cross-platform local server (Unix socket / Windows TCP)
│   │   └── kinematics.py          # FK/IK (Pinocchio)
│   ├── robot_models/
│   │   └── a1z/               # URDF model files (defaults to A1Z_Flange.urdf; use A1Z_G1Z.urdf when the gripper is attached)
│   └── utils/
│       └── utils.py               # RateRecorder, logging utilities
├── examples/
│   ├── gravity_comp.py            # Gravity compensation example
│   ├── position_hold.py           # Position hold example
│   ├── teach_and_play.py          # Zero-force teach recording and playback
│   └── gripper_hybrid_test.py     # Gripper test: free travel + torque saturation
└── tools/
    ├── a1zctl                     # Arm control CLI (serve/move/gripper/dance/stop)
    ├── gripper_set_zero.py        # Gripper zero calibration (factory-done, rarely needed)
    ├── motor_diag.py              # Motor communication diagnostics
    ├── set_zero.py                # Motor zero calibration
    ├── setup_can.sh               # Linux SocketCAN one-step setup
    ├── verify_can_mac.sh          # macOS CAN communication verification
    └── verify_can_win.py          # Windows CAN communication verification
```

## Installation

### Prerequisites

- Python >= 3.11 (the common supported baseline on all platforms)
- **Linux**: SocketCAN (hardware CAN interface required)
- **macOS / Windows**: Connects via the Galaxea USB2CANFD adapter; the SDK automatically uses the gs_usb userspace backend
- URDF model files are bundled with the SDK and synchronized from [robot_descriptions](https://github.com/userguide-galaxea/robot_descriptions/tree/galaxea/main/a1z_description). The default is `A1Z_Flange.urdf`; `with_gripper=True` selects `A1Z_G1Z.urdf`.

> See the [Cross-Platform CAN Configuration](#cross-platform-can-configuration) section for details.

### Install the SDK

```bash
# Clone the SDK
git clone --branch main https://github.com/userguide-galaxea/galaxea_a1z.git GALAXEA-A1Z

cd GALAXEA-A1Z

# Create the environment with Python 3.11 on Linux/macOS; follow the Conda workflow below on Windows
python3.11 -m venv .venv
source .venv/bin/activate

# Development mode (recommended)
python -m pip install -e .

# Or standard install
python -m pip install .
```

Dependencies installed automatically include `numpy`, `python-can>=4.0`, and `pyyaml`. On Linux/macOS, pip also installs `pin` (Pinocchio). On Windows, pip skips `pin`; install the Conda package `pinocchio` as described below.

On macOS / Windows, install with the gs_usb extras instead (includes the gs_usb userspace backend and libusb runtime):

```bash
python -m pip install -e ".[gs_usb]"
```

> Note: zsh (the default macOS shell) treats `[...]` as a glob and fails with `no matches found` — the quotes are required. Windows also needs a driver that permits libusb access; do not replace a working existing driver unnecessarily.

### Configuration File

The examples, diagnostic and calibration tools, and `a1zctl serve` automatically read `config.yaml` from the repository root. Edit this file before running them.

- `with_gripper: false` (default) selects the arm-only `A1Z_Flange.urdf`.
- `with_gripper: true` enables the G1Z gripper and selects `A1Z_G1Z.urdf`.

```yaml
can_channel: can0
control_freq_hz: 250
gravity_comp_factor: 1.0
with_gripper: false
```

| Setting | Description |
|---|---|
| `gripper_max_torque` | Maximum G1Z gripping torque in N·m. |
| `urdf_path` | Uncomment and set a path to override the URDF selected by the gripper setting. |
| `can_channel`, `bustype` | CAN interface and backend. Options include `socketcan`, `gs_usb`, `pcan`, `slcan`, and `cansub`. Auto selection uses `socketcan` on Linux and `gs_usb` on macOS and Windows. |
| `motor_a_use_new_enable_protocol` | Set to `true` only if the MotorA firmware supports the `0x7FF` enable protocol. |
| `control_freq_hz`, `min_freq_hz` | Control rate and low-frequency stop threshold in Hz. A sustained drop below the threshold triggers a protective stop. |
| `gravity_comp_factor` | Gravity compensation scale. Use `0` for off and `1` for full compensation. |
| `zero_gravity_mode` | `true` floats and `false` holds position. Omit to use the command's default mode. |
| `default_kp`, `default_kd` | Position and velocity gains in J1–J6 order. Uncomment the corresponding line and edit its values to override the [SDK defaults](#default-control-parameters). |

Example usage:

> [!CAUTION]
> Before your first run, secure the arm, clear the workspace, and keep the physical emergency stop within reach.

```bash
python examples/position_hold.py
python tools/a1zctl serve
python tools/motor_diag.py --scan
python tools/set_zero.py --all
```

### Configure the CAN Bus (SocketCAN)

> Note: verify that the CAN termination resistor is installed correctly!

Using the Galaxea USB2CANFD adapter:

```bash
# 1. Load the driver
sudo modprobe gs_usb

# 2. Confirm the interface appears (usually can0 for a single adapter)
ip link show type can

# 3. Configure and bring up (1 Mbps)
sudo ip link set can0 type can bitrate 1000000
sudo ip link set can0 up
```

If no CAN interface appears, see [Linux driver troubleshooting](docs/can-troubleshooting.md#3-linux-check-the-kernel-version-and-fix).

#### macOS (gs_usb userspace mode)

macOS has no SocketCAN. The SDK automatically detects the platform and uses the gs_usb userspace backend to connect to the Galaxea USB2CANFD adapter — **no manual CAN interface configuration is required**.

```bash
# 1. Install SDK + gs_usb extras (includes bundled libusb; usually no brew needed)
python -m pip install -e ".[gs_usb]"

# 2. Verify CAN communication (optional)
bash tools/verify_can_mac.sh
```

#### Windows (gs_usb userspace mode)

Windows also uses the gs_usb userspace backend. The `.[gs_usb]` extras install `libusb-package` and `libusb1`. Zadig is unnecessary when the adapter's existing driver already permits libusb access; bind the device to WinUSB only when it cannot be opened and the installed driver is confirmed incompatible.

```powershell
# 0. If Chinese output is garbled, switch the console to UTF-8 (Windows Terminal is recommended)
chcp 65001

# 1. Create and activate an isolated Conda environment
# The project uses Python 3.11 consistently across platforms
conda create -n a1z python=3.11 -y
conda activate a1z

# 2. Install Pinocchio with Conda (the Windows package name is pinocchio)
conda install -c conda-forge pinocchio -y

# 3. Install the SDK + gs_usb extras in the same environment
python -m pip install -e ".[gs_usb]"

# 4. Confirm python/pip point to the active Conda environment and check imports
where.exe python
python --version
python -m pip --version
python -c "import pinocchio, can, usb, usb1, gs_usb; print('Environment OK')"

# 5. Verify that the USB-CAN adapter opens and can submit a test frame
python tools\verify_can_win.py
```

#### Cross-Platform CAN Configuration

The SDK automatically selects the CAN backend by platform:

| Platform | Adapter | Install | Usage |
|----------|---------|---------|-------|
| Linux | SocketCAN kernel driver | `python -m pip install -e .` | `--can can0` |
| **macOS** | Galaxea USB2CANFD adapter | `python -m pip install -e ".[gs_usb]"` | (default) `gs_usb` |
| **macOS** | **PEAK PCAN-USB** | `python -m pip install -e ".[pcan]"` + MacCAN PCBUSB runtime | `--bustype pcan --can PCAN_USBBUS1` |
| **Windows** | Galaxea USB2CANFD adapter | `python -m pip install -e ".[gs_usb]"`; use Zadig/WinUSB only for an incompatible driver | (default) `gs_usb` |
| **Windows** | **PEAK PCAN-USB** | `python -m pip install -e ".[pcan]"` + PEAK official driver | `--bustype pcan --can PCAN_USBBUS1` |
| macOS / Windows / Linux | CSS Electronics CANsub | `python -m pip install -e ".[cansub]"`, no driver needed | `--bustype cansub --can aabbccdd-usb.local@1` |

When `--bustype` is omitted, the OS default is used: Linux defaults to `socketcan`/`can0`; macOS/Windows default to `gs_usb`/device index `0`.

## Quick Start

### Example Scripts

> With a G1Z gripper attached, set `with_gripper: true` in `config.yaml` so gravity compensation includes the gripper mass.

```bash
# Start with a low gravity compensation factor on the first run
python examples/gravity_comp.py --gravity_factor 0.3

# Increase to full compensation once direction is confirmed correct
python examples/gravity_comp.py --gravity_factor 1.0

# Position hold mode
python examples/gravity_comp.py --mode hold

# Position hold + move slowly to target (with_gripper: false)
python examples/position_hold.py --q_target_deg 0,30,-20,-15,0,0 --speed 0.5
```

> ⚠️ **If the commands above error out or the arm does not respond**, jump to [CAN Communication Troubleshooting](docs/can-troubleshooting.md#can-communication-troubleshooting).

More usage examples  [Zero-Force Teaching and Playback](docs/sdk-guide.md#zero-force-teaching-and-playback) · [Gripper](docs/sdk-guide.md#gripper) · [a1zctl](docs/sdk-guide.md#using-the-a1zctl-server) · [API Reference](docs/sdk-guide.md#api-reference)

## Tools

### Motor Communication Diagnostics

```bash
# Check CAN interface
python tools/motor_diag.py --check-can

# Scan the arm motors (includes the gripper when with_gripper is true)
python tools/motor_diag.py --scan

# Detailed probe of a specific joint (full TX/RX flow + feedback parsing)
python tools/motor_diag.py --probe 3

# Continuous monitoring of all motor states (position/velocity/temperature/error codes)
python tools/motor_diag.py --monitor

# Passive CAN bus listener (no commands sent, for diagnosing bus conflicts)
python tools/motor_diag.py --listen --duration 10

# Clear MotorB error codes
python tools/motor_diag.py --clear-error
python tools/motor_diag.py --clear-error --joints 3 4
```

> ⚠️ `--scan`, `--monitor`, and `--probe` are not passive diagnostics: they enable motors and send control frames, so the arm may move briefly. The script requires typing `YES` before continuing. Clear the workspace, support the arm securely, keep the emergency stop within reach, and do not touch the arm while the command runs. Use `--yes` for automation only after equivalent safety checks have been completed.

The diagnostic script automatically detects and suggests remedies for common issues:
- Motor not responding (unpowered / reversed CAN wiring / wrong ID / firmware mode)
- MotorB error codes (overvoltage / undervoltage / overcurrent / overtemperature / comm loss / overload)
- CAN bus faults (bus-off / error-passive / restart count)
- Temperature warnings

### Motor Zero Calibration

```bash
# Calibrate all motors (set current position as zero)
sudo python tools/set_zero.py --all

# Calibrate specific joints
sudo python tools/set_zero.py --joints 0 3
```

## Control Architecture

### MIT Position-Velocity-Torque Mixed Control

Each control cycle, the SDK sends a five-tuple (`pos`, `vel`, `kp`, `kd`, `torque`) to each motor via `send_mit_command`. The motor firmware executes PD + feedforward internally:

```python
# motor_a_driver.py / motor_b_driver.py — send_mit_command
def send_mit_command(self, pos, vel, kp, kd, torque):
    pos_u16    = float_to_uint(pos,    r.pos_min,    r.pos_max,    16)
    vel_u12    = float_to_uint(vel,    r.vel_min,    r.vel_max,    12)
    kp_u12     = float_to_uint(kp,     r.kp_min,     r.kp_max,     12)
    kd_u9      = float_to_uint(kd,     r.kd_min,     r.kd_max,      9)
    torque_u12 = float_to_uint(torque, r.torque_min, r.torque_max, 12)
    # pack into 8-byte CAN frame and send
```

The SDK's `_update()` runs at each control cycle (default 250 Hz); the send path lives in `_send_command_and_cache_hold_locked()`:

1. Read motor feedback from the CAN bus and run per-cycle safety checks (feedback freshness, motor error codes, temperatures, velocities)
2. On stale/incomplete feedback, first resend the cached gravity-only MIT hold frame, then raise a recoverable fault (see [Fault Policy](#fault-policy)); confirmed hard faults (over-temperature / over-speed / motor errors) disable the motors immediately
3. Compute full inverse dynamics `τ_id(q, vel, acc)` (gravity + Coriolis + inertia) via Pinocchio RNEA
4. Compose final torque: `τ = torque_ff + τ_id × scale × factor` — PD + gravity compensation
5. Clip to the per-joint safe range and send via the motor chain, caching a gravity-only safe-hold frame for the feedback-loss resend path

### Zero-Force Float Mode

```python
# get_a1z_robot(zero_gravity_mode=True)
self._command.kp = np.zeros(self._num_joints)        # no position stiffness
self._command.kd = self._default_kd.copy() * 0.5    # low damping
# only τ_g counteracts gravity; the arm can be freely backdriven
```

### Position Hold Mode

```python
# get_a1z_robot(zero_gravity_mode=False)
self._command.kp = self._default_kp.copy()  # [146.8988, 62.9454, 89.2416, 120.0, 40.0, 100.0]
self._command.kd = self._default_kd.copy()  # [5.0, 5.0, 5.0, 2.0776, 1.5059, 1.2553]
# PD control + gravity compensation, locks to current position
```

## Safety

- Mount the arm securely on a stable base, clear its entire reachable workspace, and keep a physical emergency stop within immediate reach.
- When a G1Z gripper is attached, set `with_gripper: true` in `config.yaml` so the dynamics model includes the gripper mass.
- `--scan`, `--monitor`, `--probe`, the example scripts, and `a1zctl` may move motors. Keep people outside the arm's workspace while they run.
- Gravity-torque thresholds, software joint limits, and smooth shutdown are software safeguards; they do not replace physical limits, guarding, an emergency-stop circuit, or a risk assessment.
- If communication stops, the process fails, or motion is unexpected, use the physical emergency stop immediately and then confirm that all motors are disabled.
- Do not use unvalidated scripts for unattended operation, production machinery, or direct human collaboration.
- Estop and fault behavior is managed by an explicit safety state machine (see "Safety State Machine" and "Fault Policy" below)
- On feedback loss / transient bus errors the SDK resends the cached safe-hold frame and keeps trying to recover; if the fault persists for 0.2 s it latches a position hold; over-temperature / over-speed / motor errors disable the motors immediately
- All target joint angles are clipped to the URDF joint limits (see "Joint Limit Handling" below)

### Safety State Machine

The control loop runs on an explicit state machine: `STOPPED` / `RUNNING` / `SOFT_ESTOP` / `COMMAND_HOLD` / `FAULT_HOLD` / `HARD_DISABLED` / `HARD_DISABLE_UNCONFIRMED`.

- `start()` is only allowed from `STOPPED` (no live control thread, no sticky lifecycle fault); otherwise it raises `RuntimeError`. After `HARD_DISABLED` / `HARD_DISABLE_UNCONFIRMED` the object cannot be restarted — recreate the `ArmRobot` instance.
- `estop()` is a **soft** emergency stop: position gain zeroed, damping halved, command position pinned to the current measured pose; **motors stay enabled** and gravity compensation keeps running, so the arm does not collapse under load. Calling it from `STOPPED` / `FAULT_HOLD` / `HARD_*` returns `False`.
- `estop()` / `release()` both return `bool`. `release(expected_fault_code=None)` returns to `RUNNING` only when the control loop is alive, feedback is fresh and complete, and the state matches (`SOFT_ESTOP` / `COMMAND_HOLD` / `FAULT_HOLD`); otherwise it returns `False` and holds. `expected_fault_code` protects against releasing an unrelated fault source.
- While estopped / holding, `command_joint_pos` / `command_joint_state` / `command_gripper` / `move_joints` are rejected (return `False` or return early, with a log entry).
- `stop()` stops the control loop and disables all motors; it **no longer shuts down the CAN bus** — the bus is closed when the object is destroyed (or by the owning caller), so the same object can be started again after `stop()`.
- `get_fault_status()` returns a thread-safe snapshot (`state` / `code` / `reason` / fault age / per-joint feedback health / `restart_allowed`, etc.).

### Fault Policy

| Class | Triggers | Behavior |
|-------|----------|----------|
| Recoverable | Stale feedback, transient CAN TX errors (EAGAIN/EWOULDBLOCK/EINTR/ENOBUFS/TX buffer full), inverse-dynamics computation errors | Hold the last command and resend the cached gravity-only MIT hold frame (motors **not** disabled); if no successful update for 0.2 s → latch `FAULT_HOLD` (full-gain position hold); `release()` once feedback recovers |
| Hard | Over-temperature, over-speed, motor error codes, non-finite feedback | Immediately disable all motors (`HARD_DISABLED`); the fault is latched and the object cannot be restarted |

### Joint Limit Handling

- Streaming commands (`command_joint_pos` / `command_joint_state`, teleop / trajectory replay): out-of-limits targets are **always clipped** to the physical limits instead of rejecting the frame or engaging estop, so non-violating joints keep tracking. Before clipping, each target is unwrapped on the S¹ circle topology to the equivalent angle nearest the current command, preventing a ±π wrap from producing a full-stroke jump. Large violations beyond the per-joint tolerance (0.05 rad by default) are logged at ERROR; small overshoots within tolerance at WARNING (rate-limited).
- `move_joints` one-shot moves: a target beyond the limit tolerance still raises `ValueError`; the new `max_jump_rad` argument refuses targets too far from the measured pose (e.g. elbow-flipped IK solutions).
- A measured position drifting past the soft limits at runtime now only logs (1 Hz rate-limited) instead of triggering an estop.

## Joint Limits

| Joint | Name | Mechanical (°) | Mechanical (rad) | Software (°) | Software (rad) |
|-------|------|----------------|-----------------|--------------|----------------|
| 0 | arm_joint1 | [-130°, 130°] | [-2.269, 2.269] | [-120°, 120°] | [-2.094, 2.094] |
| 1 | arm_joint2 | [-1.94°, 192.78°] | [-0.034, 3.365] | [0°, 180°] | [0.000, 3.142] |
| 2 | arm_joint3 | [-200.38°, 0°] | [-3.497, 0.000] | [-180°, 0°] | [-3.142, 0] |
| 3 | arm_joint4 | [-91.88°, 110.38°] | [-1.604, 1.926] | [-85°, 85°] | [-1.484, 1.484] |
| 4 | arm_joint5 | [-90°, 90°] | [-1.571, 1.571] | [-85°, 85°] | [-1.484, 1.484] |
| 5 | arm_joint6 | [-120°, 120°] | [-2.094, 2.094] | [-115°, 115°] | [-2.007, 2.007] |

## Default Control Parameters

| Parameter | Value |
|-----------|-------|
| Default KP | `[146.8988, 62.9454, 89.2416, 120.0, 40.0, 100.0]` |
| Default KD | `[5.0, 5.0, 5.0, 2.0776, 1.5059, 1.2553]` |
| Command gain caps | kp ≤ 200, kd ≤ 5 (custom gains in `command_joint_state` / `move_joints` are rejected beyond these) |
| Joint sign | `[1, 1, -1, 1, -1, 1]` (joints 3 and 5 are inverted relative to URDF) |
| Gravity torque scale | `[1, 1, 1, 1, 1, 1]` |
| Max gravity torque | `[50, 50, 50, 24, 10, 10]` Nm |
| Torque clip | `[70, 70, 70, 27, 10, 10]` Nm |
| MotorA KT | 2.8 (current-to-torque conversion factor) |
| Control frequency | 250 Hz |

## Development and Contributing

Issues and pull requests (PRs) are welcome. Prefer small, reviewable changes, and document platform differences, hardware assumptions, and safety implications explicitly.

Before opening a pull request, confirm that:

- The change addresses one focused problem and does not include unrelated formatting or generated files.
- For hardware paths that cannot be automated, document the complete manual verification procedure and result.
- Changes to public APIs, CLI behavior, defaults, configuration, or platform support update both the Chinese and English README sections.
- A successful test-frame send is not presented as proof of motor communication or control-loop stability.

## Getting Help and Reporting Issues

Before reporting a problem, review [CAN Communication Troubleshooting](docs/can-troubleshooting.md#can-communication-troubleshooting) and search existing [GitHub Issues](https://github.com/userguide-galaxea/GALAXEA-A1Z/issues). A useful new issue includes:

- Operating system and version, Python version, and repository branch/commit.
- CAN adapter model and VID:PID, backend, channel, and bitrate.
- Whether a G1Z gripper is attached, the configuration file used, and the complete command.
- The validation level that failed, complete error output, and minimal reproducible steps.
- Results of checking arm power, CANH/CANL wiring, and termination resistance.

Remove usernames, local paths, device serial numbers, and other sensitive information before posting logs publicly.

## License

This project is open-sourced under the [MIT License](LICENSE), copyright © **Galaxea**.

### Third-Party Dependency Licenses

| Dependency | License | Description |
|------------|---------|-------------|
| [numpy](https://numpy.org) | BSD-3-Clause (distributions also include separately licensed components) | Numerical computation |
| [python-can](https://github.com/hardbyte/python-can) | LGPL-3.0 | CAN bus communication |
| [pinocchio (pin)](https://github.com/stack-of-tasks/pinocchio) | BSD-2-Clause | Robot dynamics computation |
| [PyYAML](https://github.com/yaml/pyyaml) | MIT | YAML configuration parsing |

The repository's MIT License applies to this repository's own code and documentation. Third-party libraries and optional CAN runtimes remain subject to their own licenses. Before redistribution, review the upstream license text and compliance requirements for the exact versions you ship.
