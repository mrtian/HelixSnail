# HelixSnail 3D打印换色方案

## 1. 系统原理
采用多电机驱动单挤出机架构实现多色打印功能，核心原理包括：
- 每个颜色通道配备独立步进电机负责材料输送
- 挤出机配备压力传感器和光电开关实现材料位置检测
- 使用舵机控制挤出机压轴开合
- CAN总线控制多个电机协同工作
- 通过电流监测防止材料打滑

## 2. 材料元件列表
| 部件 | 规格 | 数量 | 备注 |
|------|------|------|------|
| 步进电机 | NEMA17 | 4-6个 | 根据颜色数量决定|
| 光电开关 | 漫反射型 | 每个颜色通道1个 | 检测材料位置 |
| 限位开关 | 机械式 | 2个 | 挤出机位置检测 |
| 舵机 | SG90 | 1个 | 控制挤出机压轴 |
| CAN总线板 | 带电机驱动 | 1个 |  |
| 电流传感器 | ACS712 | 1个 | 挤出机电流检测 |
| 导线 | AWG20 | 若干 | 连接各部件 |
| 3D打印支架 | 自定义 | 若干 | 固定电机和传感器 |

## 3. 安装步骤
### 3.1 机械安装
1. 在Y轴末端15cm处安装光电开关支架
2. 将各颜色通道步进电机安装在材料架上
3. 在挤出机入口处安装光电限位开关
4. 安装舵机连接挤出机压轴机构
5. 布置所有线缆并固定

### 3.2 电气连接
1. 将各电机连接到CAN板的电机驱动接口
2. 连接所有光电开关到CAN板的限位接口
3. 连接舵机到PWM输出接口
4. 连接电流传感器到ADC接口
5. 确保所有接地良好

## 4. 系统配置
### 4.1 CAN总线配置
```ini
[board]
name: MKS_TinyBee
can_id: 1

[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32f103xe_35FFDA054243303343742157-if00
restart_method: command

[tmc2209 extruder]
uart_pin: PA10
interpolate: True
run_current: 0.8
hold_current: 0.5
stealthchop_threshold: 0

[tmc2209 filament1]
uart_pin: PA9
interpolate: True
run_current: 0.6
hold_current: 0.3
stealthchop_threshold: 0

[tmc2209 filament2]
uart_pin: PA8
interpolate: True
run_current: 0.6
hold_current: 0.3
stealthchop_threshold: 0
```

## 5. G代码实现
### 5.1 换色宏定义
```gcode
[gcode_macro CHANGE_FILAMENT]
description: Change filament to specified color
gcode:
    {% set TARGET = params.TARGET|default(0)|int %}
    {% set RETRACT = params.RETRACT|default(50)|float %}
    {% set PURGE = params.PURGE|default(50)|float %}
    
    ; Step 1: Retract current filament
    M117 Retracting filament
    SET_FILAMENT_SENSOR SENSOR=fs_extruder ENABLE=0
    G92 E0
    G1 E-{RETRACT} F1200
    M400
    
    ; Step 2: Check for filament slip via current
    {% set current = printer.extruder.driver_current %}
    {% if current < 0.5 %}
        M118 WARNING: Possible filament slip detected
        G1 E-5 F600
    {% endif %}
```

## 6. 维护方案

| 周期  | 维护项目       | 操作说明                         |
|-------|----------------|----------------------------------|
| 50h   | 齿轮清洁       | 使用酒精棉片擦拭齿轮表面         |
| 200h  | 轴承润滑       | 添加二硫化钼润滑脂               |
| 500h  | 传感器校准     | 使用标准测试片检测光电开关灵敏度 |

## 7. 故障排除指南

| 故障现象     | 可能原因           | 解决方案                     |
|--------------|--------------------|------------------------------|
| 换色失败     | 材料回抽不足       | 增加RETRACT参数值            |
| 挤出异常     | 喷嘴堵塞           | 执行冷拔或更换喷嘴           |
| 传感器误报   | 灵敏度设置不当     | 重新校准光电开关             |
| 电机过热     | 电流设置过高       | 降低run_current参数值        |
