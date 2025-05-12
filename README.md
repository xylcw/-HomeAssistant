#-家庭助理
在云服务器上通过Docker安装家庭助理，实现通过外网管理米家设备，并且可以接入其他品牌设备
#🌀 斐讯M1空气检测仪接入HA(云服务器版)

>📌**核心目标**：通过本教程，将你的==斐讯M1空气检测仪==无缝接入**米家**生态系统，实现**跨平台联动**和**语音控制**！

---

##📋  前置准备清单
-✅ 斐讯M1空气检测仪设备(刷入了固件版本的，即烧录过，[烧录教程](https://github.com/a2633063/zM1/wiki/%E5%9B%BA%E4%BB%B6%E7%83%A7%E5%BD%95)）
-✅ 安卓手机(iOS端及鸿蒙端暂无相关适配软件）
-✅ 云服务器(使用电脑或者带Linux系统的设备都行，==当前指南为云服务器版教程==，其他版本教程请自行寻找）
-✅  安卓应用[ZControl](https://www.pgyer.com/rGpHakMY)(点击下载)
-✅**重要工具**：[家庭助理](https://www.home-assistant.io/)、mqtt服务器（均在云服务器上进行安装，此处不用管）

---
%% 注：检测仪设备需用数据线持续供电， 请在操作和使用过程中保持通电 %%
##🛠️  接入步骤详解

###1️Ю设备初始化连接WiFi
1.长按设备按钮15秒左右恢复出厂模式，开机后，用手机/电脑连接设备的热点：==ZM1_XXXX==(XXXX为唯一标识)，连接后用浏览器打开：http：//192.168.0.1
2. 在打开的页面中填写你的``WiFi名称及密码``，点下一步，会显示配置成功，设备自动重启后连接wifi
3.在==ZControl==的增加设备页面中点击获取局域网设备按钮.并确认设备列表中增加了设备.注意：此步骤可能会通信不稳定，请多试几次(如图1-1)
![[IMG_20250507_085555.jpg]]
#图1-1
**完成以上步骤后，当zM1设备在局域网下，手机已经可以直接控制zM1设备了.****

---
###2️为云服务器安装宝塔面板
  [宝塔官网](https://www.bt.cn/new/download.html)
1.打开宝塔官网，复制通用安装脚本命令，通过终端连接服务器，并粘贴脚本，等待安装完成 (如图2-1)
![[IMG_20250507_111232.jpg]]
#图2-1
2.打开显示的外网面板链接，并输入账号密码，登录后绑定宝塔账号，没有宝塔账号可以先注册一个
3.安装面板推荐配置，等待安装完成
4. **关键步骤**：
   >❗ 安装Docker
   > 在面板左侧菜单栏点击**Docker**，点击前往安装，等待安装完成

---
###3️在Docker应用商店安装home Assistant以及EMQX
1.在面板中点击Docker→应用搜索🔍，输入``家庭助理``并安装 (如图3-1
![[mmexport 1746585639186.png]]
#图3-1
 
 2. 输入``emqx``并安装 (如图3-2)
![[mmexport 1746585641345.png]]
#图3-2

---
###4️➈开放服务器端口，确保安装的Docker应用能够访问
1.给服务器的==安全组/防火墙==添加新==规则/策略==，配置如下:
```
规则方向：入

协议：自定义tcp

端口范围/端口：18083、1883、8883、8083、8084、8123 (请分批次开放端口，一个端口一个端口来)

授权ip：0.0.0.0/0
```
(如图4-1，``注:不同服务器供应商页面不同，步骤差不多``)
![[mmexport1746586808245.jpg]]
              #图4-1

---
### 5️⃣配置mqtt服务器
1. 在宝塔面板中点击Docker→应用商店 →已安装→找到==emqx软件卡片==
2. 点击详情→记住默认账号密码
   (如图5-1)
![[IMG_20250508_000945.png]]
              #图5-1
3. 点击卡片端口:``18083``，前往登录页面，输入账号密码登录 (如图5-2)
![[IMG_20250508_001755.jpg]]
              #图5-2
4. 登录后会叫你修改密码，修改完成后就可以进行下一步了
---
### 6️⃣配置软件端，上传数据到mqtt服务器
1. 打开==ZControl==APP，在侧边栏中点击设置按钮
2. 输入MQTT地址(``云服务器IP+1883`` )如:45.15.4.5:1883、MQTT用户名`admin`、MQTT登录密码(你修改过后的)，后返回设备界面，确保底部有提示服务器已经连接。
3. 点击右上角云图标，app会通过udp将MQTT配置发给zM1.如果正常,zM1会直接连上MQTT服务器,现在就可以通过MQTT控制设备了.
如图6-1
![[a2ebd6166cbb4cf42a8756bb66569e4955e011406011ed8ec76131bf43a5b4f7.0.png]]
              #图6-1

---
### 7️⃣配置Home Assistant
1. 先为HA安装小米官方插件:
	1. 打开宝塔面板，点击Docker，点击容器，找到有HA名称的容器
	2. 点击容器右边的终端 → 确认 
	  (如图7-1)
![[IMG_20250512_190443.png]]
              #图7-1
       3. 分三次输入以下指令(隔一行为一次)
```
git clone https://github.com/XiaoMi/ha_xiaomi_home.git

cd ha_xiaomi_home

./install.sh /config
```
2. 完成后重启容器 (如图7-2)
![[IMG_20250512_190347.png]]
              #图7-2
3. 等待重启完后，按照5️⃣-1的方法找到HomeAssistant卡片，点击``端口:8123`` 前往配置(如图7-3)
![[clipboard_2025-05-12_18-33.png]]
              #图7-3
4. 点击创建智能家居后设置账号密码。 (如图7-4)
![[IMG_20250508_004242.jpg]]
              #图7-4
5. 然后一直点击下一步，直到显示HA主页 (如图7-5)
![[clipboard_2025-05-12_18-38.png]]
              #图7-5
6. 后点击左下角设置 → 设备与服务 →
右下角添加集成 → 搜索 ``xiaomi`` 选择第二个==Xiaomi Home==(如图7-6)
![[clipboard_2025-05-12_19-08.png]]
              #图7-6
7. 点击后一直点击下一步，当弹出登录页面，点击登录/注册小米账号
(如图7-7)
![[clipboard_2025-05-12_19-11.png]]
              #图7-7

---
### 8️⃣配置HA文件(最后一步!!!)
1. 按照7️⃣-6的方法添加集成，搜索mqtt
→ 选择第一个  (如图8-1)
![[IMG_20250512_203212.jpg]]
              #图8-1
2. 输入==服务器IP==和==mqtt服务器==账号和密码，完成后提交  (如图8-2)
![[IMG_20250512_203147.png]]
              #图8-2
3. 返回到宝塔面板，在Docker卡片页面点击HA的安装目录 → 进入data目录  (如图8-3)
![[IMG_20250512_204250.jpg]]
              #图8-3
4. 找到==configuration.yaml==文件，点击打开，在最后面添加以下内容
```
homeassistant:
    packages: !include_dir_named packages
```
5. 添加完成后保存并关闭窗口，在同一目录下添加==packages==文件夹，并在文件夹内添加名为==m1.yaml==的文件        (如图8-4)
![[IMG_20250512_205123.jpg]]
              #图8-4
6. 💡文件内要填入的内容见下方配置文档，在内容中，需要把==b0f89323bd29==替换成自己的检测仪设备的MAC地址
7. MAC地址在==ZControl==软件内右上角点击笔图标即可看到
8. 完成后保存，并返回到卡片页面，重启HA应用。(如图8-5)
![[clipboard_2025-05-12_20-52.png]]
              #图8-5
重启完成后即可在[手机端HA](https://m.downxing.com/az/218015.htm)或通过访问网址(服务器IP:8123)来查询和设置检测仪数据。==至此接入工作也就圆满完成啦==！

---

## 🌐 配置文档(记得替换mac地址)
```
mqtt:

  sensor:

    - name: 'zm1_b0f89323bd29_temperature'

      unique_id: zm1_b0f89323bd29_temperature

#     friendly_name: 温度

      state_topic: 'device/zm1/b0f89323bd29/sensor'

      unit_of_measurement: '°C'

      icon: 'mdi:thermometer'

      value_template: '{{ value_json.temperature }}'

#      availability_topic: "device/zm1/b0f89323bd29/availability"

#      payload_available: 1

#      payload_not_available: 0

    - name: 'zm1_b0f89323bd29_humidity'

      unique_id: zm1_b0f89323bd29_humidity

#     friendly_name: 湿度

      state_topic: 'device/zm1/b0f89323bd29/sensor'

      unit_of_measurement: '%'

      icon: mdi:water-percent

      value_template: '{{ value_json.humidity }}'

#      availability_topic: "device/zm1/b0f89323bd29/availability"

#      payload_available: 1

#      payload_not_available: 0

    - name: 'zm1_b0f89323bd29_pm25'

      unique_id: zm1_b0f89323bd29_pm25

#      availability_topic: "device/zm1/b0f89323bd29/availability"

#      payload_available: 1

#      payload_not_available: 0

#      friendly_name: PM25

      state_topic: 'device/zm1/b0f89323bd29/sensor'

      unit_of_measurement: 'μg/m³'

      icon: mdi:blur

      value_template: '{{ value_json.PM25 }}'

    - name: 'zm1_b0f89323bd29_hcho'

      unique_id: zm1_b0f89323bd29_hcho

#     friendly_name: 甲醛

      state_topic: 'device/zm1/b0f89323bd29/sensor'

      unit_of_measurement: 'mg/m³'

      icon: mdi:chemical-weapon

      value_template: '{{ value_json.formaldehyde }}'

#      availability_topic: "device/zm1/b0f89323bd29/availability"

#      payload_available: 1

#      payload_not_available: 0

  light:

      name: zm1_b0f89323bd29_brightness

      unique_id: zm1_b0f89323bd29_brightness

      schema: template

      command_topic: "device/zm1/b0f89323bd29/set"

      state_topic: "device/zm1/b0f89323bd29/state"

      command_on_template: >

        {"mac": "b0f89323bd29"

        {%- if brightness is defined -%}

        , "brightness": {{ ((brightness-1) / 64 )|int +1 }}

        {%- else -%}

        , "brightness": 4

        {%- endif -%}

        }

      command_off_template: '{"mac": "b0f89323bd29", "brightness": 0}'

      state_template: >

        {%- if value_json.brightness == 0 -%}

          off

        {%- else -%}

          on

        {%- endif -%}

      brightness_template: >

        {%- if value_json.brightness is defined -%}

          {{ ( value_json.brightness *64 )|int }}

        {%- endif -%}





homeassistant:

  customize:

    light.zm1_b0f89323bd29_brightness:

      friendly_name: zM1亮度

    sensor.zm1_b0f89323bd29_temperature:

      friendly_name: zM1温度

    sensor.zm1_b0f89323bd29_humidity:

      friendly_name: zM1湿度

    sensor.zm1_b0f89323bd29_pm25:

      friendly_name: zM1 PM2.5

    sensor.zm1_b0f89323bd29_hcho:

      friendly_name: zM1甲醛


```

---

## ❓ 常见问题解答（FAQ）

### 🔍 检测仪无法被软件发现？
- 网络不是很稳定多刷新几次，或者退出软件重进
### 📡 数据同步延迟？
- 由于mqtt服务器要先获取设备数据，然后再上传到服务器，最后HA从mqtt服务器订阅，导致延迟会有点大，作者暂时不知道怎么解决，可以自行研究

---

#### 🎯 版本说明
- 文档最后更新：2025年5月12日20时
- 教程文档版本：1.0.0

```

📝 **作者注**：本教程实测基于大佬部分教程以及小米官方文档，遇到问题欢迎在3138848070@qq.com邮箱留言讨论！
```

# 最后附上来自deepseek和作者的结晶: ==网页版m1空气检测仪数据面板==

![[检测仪 1.html]]
