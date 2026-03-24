# PVE-X10SRH-CLN4F-fan-control

Supermicro X10SRH-CLN4F 主板专用 IPMI 风扇控制程序

---

## 主板信息

| 项目 | 详情 |
|------|------|
| 主板型号 | Supermicro X10SRH-CLN4F |
| CPU | Intel Xeon E5-2667 v3 @ 3.20GHz |
| 内存 | 128 GB |
| 风扇接口 | FAN1-FAN6 |
| IPMI BMC | 板载 |

---

## 风扇接口说明

| 接口 | 用途 | 类型 | 控制方式 |
|------|------|------|----------|
| FAN3 | CPU 风扇 | 4pin PWM | 线性调节 ✅ |
| FAN4 | 周边风扇 | 3pin | 固定档位 ❌ |
| FAN5 | 周边风扇 | 3pin | 固定档位 ❌ |

---

## 风扇转速曲线（实测数据）

### FAN3（CPU风扇）- 线性可调

| 百分比 | 十六进制 | 转速 RPM | 说明 |
|--------|----------|----------|------|
| 10% | 0x0A | 2400 | 静音模式 |
| 15% | 0x0F | 2400 | |
| 20% | 0x14 | 2500-2600 | |
| 25% | 0x19 | 2900 | |
| 30% | 0x1E | 3100-3400 | |
| 35% | 0x23 | 3800 | |
| 40% | 0x28 | 4100 | |
| 45% | 0x2D | 4500 | |
| 50% | 0x32 | 4800-4900 | |
| 55% | 0x37 | 5200 | |
| 60% | 0x3C | 5500-5600 | |
| 65% | 0x41 | 5800-5900 | |
| 70% | 0x46 | 6100-6200 | |
| 75% | 0x4B | 6400 | |
| 80% | 0x50 | 6700 | |
| 85% | 0x55 | 6900 | |
| 90% | 0x5A | 7200 | |
| 95% | 0x5F | 7400 | |
| 100% | 0x64 | 7600-7700 | 最大转速 |

**特点**：FAN3 可以线性调节，最小 2400 RPM，最大 7700 RPM

---

### FAN4（周边风扇）- 固定档位

| 百分比 | 十六进制 | 转速 RPM | 说明 |
|--------|----------|----------|------|
| 0-10% | 0x00-0x0A | **1400** | 推荐使用 |
| 11% | 0x0B | 800 | 过渡档位 |
| 12-48% | 0x0C-0x30 | 700 | |
| 49-53% | 0x31-0x35 | 1400 | 异常区间 |
| 54-100% | 0x36-0x64 | 700 | |

**注意**：FAN4 只有 5 个有效档位，不是线性调节

---

### FAN5（周边风扇）- 固定档位

| 百分比 | 十六进制 | 转速 RPM | 说明 |
|--------|----------|----------|------|
| 0-35% | 0x00-0x23 | **1400** | 推荐使用 |
| 40-100% | 0x28-0x64 | 600 | 可选更低转速 |

**注意**：FAN5 只有 2 个档位

---

## 控制策略

### 风扇控制逻辑

| 风扇 | 控制方式 | 说明 |
|------|----------|------|
| **FAN3** | 自动调节 | 根据 CPU 温度线性调节 10-100% |
| **FAN4** | 联动控制 | CPU < 60°C → 0x64 (700 RPM)<br>CPU ≥ 60°C → 0x00 (1400 RPM) |
| **FAN5** | 联动控制 | CPU < 60°C → 0x64 (600 RPM)<br>CPU ≥ 60°C → 0x00 (1400 RPM) |

---

### 温度阈值设置

| 组件 | 升速阈值 | 降速阈值 | 警告温度 |
|------|----------|----------|----------|
| CPU | 60°C | 50°C | 80°C |
| 内存/其他 | 80°C | 45°C | 85°C |
| 磁盘 | 70°C | 55°C | 75°C |

---

### FAN3 温度-转速对照表

| CPU 温度 | FAN3 百分比 | FAN3 转速 |
|----------|-------------|----------|
| < 40°C | 10% | 2400 RPM |
| 40-50°C | 15-20% | 2400-2600 RPM |
| 50-60°C | 25-30% | 2900-3400 RPM |
| 60-70°C | 40-55% | 4100-5200 RPM |
| 70-80°C | 60-75% | 5500-6400 RPM |
| > 80°C | 80-100% | 6700-7700 RPM |

---

## IPMI 命令说明

### Zone 0（FAN3）

```bash
# 切换到手动模式
ipmitool raw 0x30 0x45 0x01 0x01

# 设置转速（低字节）
ipmitool raw 0x30 0x70 0x66 0x01 0x00 <值>
```

### Zone 1（FAN4/FAN5）

```bash
# 设置 FAN4 转速
ipmitool raw 0x30 0x70 0x66 0x01 0x01 <值>

# FAN5 和 FAN4 共用同一个控制寄存器
```

---

## 安装说明

### 前提条件

```bash
# 安装依赖
apt update && apt install -y python3 ipmitool smartmontools beep
```

### 克隆仓库

```bash
git clone https://github.com/z104866/PVE-X10SRH-CLN4F-fan-control.git
cd PVE-X10SRH-CLN4F-fan-control
```

### 运行安装脚本

```bash
./setup.sh
```

### 启动服务

```bash
systemctl enable --now supermicro-fan-control
```

### 查看日志

```bash
journalctl -f -u supermicro-fan-control
```

---

## 配置文件

配置文件位置：`/etc/supermicro-fan-control/settings.yml`

### 关键配置项

```yaml
fan:
  initial_speed: 16      # 初始转速 16%
  max_speed: 50          # 最大转速 50%
  min_speed: 10          # 最小转速 10%
  inc_speed_step: 5      # 增速步进 5%
  dec_speed_step: 2      # 降速步进 2%
  update_interval: 120   # 更新间隔 120秒

cpu:
  max_temp: 80           # CPU 最高温度阈值
  min_temp: 40          # CPU 最低温度阈值
```

---

## 测试记录

### 测试日期
2026-03-23 至 2026-03-24

### 测试方法
1. 切换到 Full 模式
2. 从 0% 到 100%，每次增加 2-5%
3. 等待风扇稳定后记录转速
4. 每个测试重复 3 次取一致结果

### 测试环境
- 系统：Proxmox VE (PVE)
- 内核：Linux 6.17.4-2-pve
- 室温：约 25°C
- 负载：空载至满载

---

## 已知问题

1. **FAN4/FAN5 无法线性调节**
   - 原因：主板 BMC 固件限制
   - 解决：使用固定档位控制

2. **FAN4 在 49-53% 区间行为异常**
   - 该区间会切换到 1400 RPM
   - 建议避免使用该区间

3. **低百分比值可能不稳定**
   - FAN3 在 5% 时可能出现不稳定
   - 建议最低使用 10%

---

## 参考资料

- [Supermicro Fan Speed Control 原始项目](https://b3n.org/supermicro-fan-speed-script/)
- [Supermicro X9/X10/X11 Fan Speed Control](https://forums.servethehome.com/index.php?threads/supermicro-x9-x10-x11-fan-speed-control.10059/)
- [IPMI Fan Control Registers](https://forums.servethehome.com/index.php?resources/supermicro-x9-x10-x11-fan-speed-control.20/)

---

## License

AGPL-3.0

---

## 作者

基于 [Benjamin Bryan](https://b3n.org) 的原始代码改编

针对 Supermicro X10SRH-CLN4F 主板优化
