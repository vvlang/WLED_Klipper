# 使用 WLED 为您的 Klipper/Moonraker 机器添加 LED 控制

阅读本文档时，请注意我是通过反复试验才搞明白的。您的结果可能会有所不同，所有这些信息仅供娱乐目的。我的免费建议对您来说值多少钱，取决于您为此付了多少钱。当然，如果您付了钱，它可能比您付的钱稍微不值钱一点。

无论如何，继续往下看，我花了相当多的时间试图理解如何在我的 RatRig 打印机上添加一些简单的 NeoPixel 灯带。结果实际上相当简单，但当时肯定不是这样的。

我可以告诉您这并不容易。有很多关于 NeoPixel 相关不同事物的信息，但关于如何在我的打印机操作系统上实现的信息不多，尤其是在打印周期的特定时间点。我希望为本文档的读者澄清这一点，并使过程尽可能轻松。我是 RatOS 用户，它是 Klipper 的一个分支，所以它应该适用于大多数 Klipper 机器。我想指出，下面的方法基于 Klipper 的 cfg 文件中包含的宏，并在开始/结束打印宏中以这种方式调用。一旦您构建了控制宏，您也可以轻松地在切片器的开始 G 代码和结束 G 代码中调用它们。我选择这种方法是因为我在不同的切片器和不同的计算机之间切换来切片和发送打印文件。对我来说，在一个地方管理它更容易。

我的 LED 控制系统基于两个控制板：ESP 8266 开发板和 ESP32。两者都可以刷入 WLED，是控制 LED 的绝佳选择。我目前在我的机器上同时使用两者。它们很便宜，所以我只是多买一些。 


## 您需要的东西
我不是在推荐以下任何硬件，只是提供选项。<b><i> 请自行选择。 </b></i>
 - 您选择的 NeoPixel 灯带。确保它们是可单独寻址的。
 - 一些用于将 LED 连接到控制器的电线。我使用 [这个](https://www.amazon.com/StrivedayTM-3-core-Control-Shielded-Headphone/dp/B01LNH9ZYK/ref=sr_1_7?crid=2X5OBMKMEGJNO&keywords=24%2Bawg%2B3%2Bconductor%2Bwire&qid=1661885002&sprefix=24%2BAWG%2Caps%2C109&sr=8-7&th=1) 或 [这个](https://www.amazon.com/C-able-Conductor-WS2811-WS2812b-Extension/dp/B082KNCKB3/ref=sr_1_1_sspa?crid=2X5OBMKMEGJNO&keywords=24+awg+3+conductor+wire&qid=1661885311&sprefix=24+AWG%2Caps%2C109&sr=8-1-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyN1I2TFhXMlhPUkZWJmVuY3J5cHRlZElkPUEwNzEzNTI1M1Y5M1NCMkZNMDEzMiZlbmNyeXB0ZWRBZElkPUEwMDczNDUwMU01NUQ5UEZHUzQzQyZ3aWRnZXROYW1lPXNwX2F0ZiZhY3Rpb249Y2xpY2tSZWRpcmVjdCZkb05vdExvZ0NsaWNrPXRydWU=)，但您应该使用您习惯使用的。我的建议值多少钱取决于您为此付了多少钱。
 - [ESP 8266 开发板](https://www.amazon.com/HiLetgo-Internet-Development-Wireless-Micropython/dp/B081CSJV2V/ref=sr_1_3?crid=NYQ6QPIIUMM7&keywords=8266%2Bdev%2Bboard&qid=1661863407&sprefix=8266%2Bdev%2Bboard%2Caps%2C64&sr=8-3&th=1)
 - [ESP 32](https://www.amazon.com/ESP-WROOM-32-Development-Microcontroller-Integrated-Compatible/dp/B08D5ZD528/ref=sr_1_3?crid=38QCSRFSM52NX&keywords=esp+32&qid=1661863479&sprefix=esp+32%2Caps%2C76&sr=8-3)
 - [刷机软件](https://github.com/marcelstoer/nodemcu-pyflasher)
 - 用于切割和焊接的简单手工工具。如果您决定使用压接端子连接到板子，还需要压接工具。
 - 用于将板子连接到 PC 的线缆
 - 一个用于容纳控制器的[外壳](https://www.thingiverse.com/thing:4816827)，便于安装。请注意，如果您将此用于 ESP32，您应该在 X 轴上以 102% 打印以获得更好的贴合度。这是相同文件的本地链接，感谢 Thingiverse 上的 Rainer Winkel：[外壳](https://github.com/Gliptopolis/WLED_Klipper/tree/main/4816827_NodeMCU__ESP_8266__Case/files)
 - 使用 Mainsail / Klipper 的机器（Mainsail 内置支持 WLED）

# 开始使用
![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/walking-with-iphone-x.jpg)

让我们开始吧！您需要做的第一件事是访问 [Tyson Nichols 的精彩网站](https://tynick.com/blog/11-03-2019/getting-started-with-wled-on-esp8266/) 并按照他的教程在 ESP 8266 上安装 WLED。以防万一，我在文件部分添加了他页面的 [PDF 版本](https://github.com/Gliptopolis/WLED_Klipper/blob/main/Getting%20Started%20With%20WLED%20on%20ESP8266%20-%20tynick.com%20_%20AWS%2C%20Linux%2C%20Raspberry%20Pi%2C%20and%20Home%20Automation.pdf)。

它很好地解释了如何让所有东西协同工作，使 WLED 为您的控制系统启动并运行。

当他的方法如此有效时，我不需要重新发明轮子。但是，我建议如果您使用的是 ESP32 而不是 8266 开发板，请访问这里并安装 WLED。这要简单得多。[Web 安装程序](https://install.wled.me/)

假设现在所有东西都已连接和编程，是时候开始与 Moonraker 和 Klipper 集成了...

现在是打开 WLED 应用程序并创建自定义配置文件的最佳时机，这些配置文件稍后将被 Klipper 调用。当您打开应用程序时，您应该看到：
![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/WLED01.jpg)

第一步是选择调色板或单一颜色。这是从主屏幕，您可以选择最多 3 种可以混合的颜色或使用预定义的调色板。

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/WLED01%20(2).jpg)

选择自定义颜色或预定义集合后，转到下一个标签页，即效果。这是您为 LED 选择动画的地方。尝试一下，找到您喜欢的效果。一旦您对效果满意，就该保存自定义配置文件了。

注意：如果您觉得有必要，可以转到分段标签页并为 LED 灯串的特定部分定义效果。我将所有内容应用到整个灯串，所以我不使用这个。

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/WLED02.jpg))

转到预设标签页，您应该看到这个屏幕：

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/WLED04_B.jpg))

这里要做的第一件事是单击创建预设按钮。

然后您应该看到这个屏幕：

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/WLED04_D.jpg)

此屏幕和过程中最重要的部分是预设的"保存到 ID"编号。这是我们将在 Klipper 宏中调用效果的方式，因此请确保保持编号简单。我使用一个简单的名称来提醒我在新预设部分中什么是什么，并使快速加载标签与保存到 ID 编号相同，因为我不聪明，这很简单。还应该注意的是，预设将根据您在其他标签页上选择的效果和颜色创建。这只是保存它们。很简单。

最终您会得到类似这样的内容：

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/WLED04_P.jpg)

这些是我为打印机底部的 LED 灯串保存的预设。我建议您花时间创建一个"开启"预设和另一个"关闭"预设，您也可以使用它们。我的"开启"只是用于工作的白光，"关闭"显然完全没有光。这在后面的结束打印内容中很有用。

现在是时候开始在 Klipper 中整合这些预设了。首先，您需要让 Mainsail 知道您连接了一些渴望被控制的 LED。

## Moonraker
我不是专家，但以下是对我有用的方法。如果您有更好的方法，那就去做吧。

首先，我们需要让 Moonraker 知道我们有一些由 WLED 控制的 LED。我们需要使用以下信息编辑 Moonraker.conf 文件：

```bash
[wled chamber]
type: http
address: 192.168.0.45
initial_red: 0.5
initial_green: 0.4
initial_blue: 0.3
chain_count: 42
```
对于您的特定安装，您应该将名称从 'chamber' 更改为您想要的任何名称。例如，如果您想将它们称为 'frame'，只需更改为
  
  [wled frame]  

我使用 http 类型，因为我正在利用控制器上的内置 wifi。您可以在 WLED 应用程序上找到地址，并将其输入到地址部分

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/Config_C.jpg)

我不使用红色、绿色部分，因为我设置 WLED 应用程序以保存的配置文件自动启动，但您可以在这里做适合您的事情。这些值基于最大值为 1。

链计数对于您的效果正常工作非常重要。控制器需要知道您的灯串上有多少个 NeoPixel 才能正常工作。只需数一下它们并在此处输入数字。

为您安装的每个控制器重复此过程。我目前在打印机上有 4 个控制器，控制床灯、上部工作灯、后面板上的装饰灯和热端盖上的一个随温度变化颜色的灯。我确信有一种方法可以将一个控制器用于多个灯串，但不知道如何实现。这种方法对我有效。

Moonraker 就是这样。一旦您进行了更新，保存您的新配置文件并准备好转移到 Klipper 进行魔法...

## Klipper
为了控制 WLED，您需要使用宏修改 printer.cfg 文件来控制 LED，并更新您的开始打印和结束打印宏。另外，如果您是 RatOS 用户，请不要修改配置文件夹中的文件。从 macros.cfg 文件中复制您现有的开始和结束打印宏部分，并粘贴到 printer.cfg 文件中，这样您就可以始终轻松地恢复到原始版本。问我怎么知道的...

以下是我的开始和结束打印宏，包括对 WLED 预设特定宏的调用：

```bash
[gcode_macro START_PRINT]
description: 开始打印程序，在您的切片器中使用此功能。
gcode:
  CLEAR_PAUSE
  SAVE_GCODE_STATE NAME=start_print_state
  Breathe_On
  # 公制单位
  G21
  # 绝对定位
  G90 
  # 设置挤出机为绝对模式
  M82
  # 如果需要则回零
  MAYBE_HOME
  M117 Heating bed...
  RESPOND MSG="Heating bed..."
  led_heating
  # 等待热床加热
  M190 S{params.BED_TEMP|default(printer.heater_bed.target, true) }
  # 运行可自定义的 "AFTER_HEATING_BED" 宏
  _START_PRINT_AFTER_HEATING_BED
  # 运行可自定义的 "BED_MESH" 宏
  led_mesh
  _START_PRINT_BED_MESH
  # 开始加热挤出机
  M104 S{params.EXTRUDER_TEMP|default(printer.extruder.target, true) }
  # 运行可自定义的 "PARK" 宏
  led_printing
  _START_PRINT_PARK
  # 等待挤出机加热
  M117 Heating Extruder...
  RESPOND MSG="Heating Extruder..."
  led_heating
  M109 S{params.EXTRUDER_TEMP|default(printer.extruder.target, true) }
  # 运行可自定义的 "AFTER_HEATING_EXTRUDER" 宏
  _START_PRINT_AFTER_HEATING_EXTRUDER
  M117 Printing...
  RESPOND MSG="Printing..."
  RESTORE_GCODE_STATE NAME=start_print_state
  led_printing
  # 根据用户配置设置挤出模式
  {% if printer["gcode_macro RatOS"].relative_extrusion|lower == 'true' %}
    M83
  {% else %}
    M82
  {% endif %}
  G92 E0

[gcode_macro END_PRINT]
description: 结束打印程序，在您的切片器中使用此功能。
gcode:
  SAVE_GCODE_STATE NAME=end_print_state
  _END_PRINT_BEFORE_HEATERS_OFF
  TURN_OFF_HEATERS
  _END_PRINT_AFTER_HEATERS_OFF
  _END_PRINT_PARK
  led_off
  Breathe_Off
  # 清除已加载的倾斜配置文件（如果有）
  {% if printer["gcode_macro RatOS"].skew_profile is defined %}
    SET_SKEW CLEAR=1
  {% endif %}
  # 关闭步进电机
  M84
  # 关闭部件冷却风扇
  M107
  M117 Done :)
  RESPOND MSG="Done :)"
  RESTORE_GCODE_STATE NAME=end_print_state
```

这是突出显示宏的屏幕截图：

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/start_print.jpg)

当然，您不能调用尚未创建的宏，所以现在让我们来处理这些...

首先，您需要在 printer.cfg 文件中添加此内容，以便在创建必要的宏后激活 LED：
```bash
[gcode_macro WLED_ON]
description: 使用可选预设打开 WLED 灯带并重置 LED 颜色
gcode:
  {% set strip = params.STRIP|default("chamber")|string %}
  {% set preset = params.PRESET|default(4)|int %}

  {action_call_remote_method("set_wled_state",
                             strip=strip,
                             state=True,
                             preset=preset)}
```
然后作为调用每个预设所需宏的示例，使用以下格式：
```bash
[gcode_macro led_heating]
gcode:
  WLED_ON STRIP=chamber PRESET=3
```

此宏假设几件事：首先，您的 LED 灯串在 moonraker.conf 文件中命名为 "chamber"，并且您要初始化的预设是 WLED 中"保存到 ID"编号中的 3 号。如果您在添加 WLED 条目时在 Moonraker 中选择了不同的名称，那么这就是在宏中使用的名称。另外，如果您想调用不同的预设，只需使用该预设的相应编号。

需要注意的事项：如果您决定将 ESP 模块直接连接到 5V 而不是使用壁式适配器，只需将您选择的电源的 + 和 - 与到 NeoPixel 的布线并联。像这样：（高度专业的绘图，感谢 Discord 上的 Max.#6343）

![alt text](https://github.com/Gliptopolis/WLED_Klipper/blob/main/images/Max.%236343.jpg)

希望这能帮助您点亮您的项目。如果我能提供帮助，请通过 Discord 联系我，Glipski#1809。我会尝试回答问题。只是记住我不是很聪明。

一个示例：


https://user-images.githubusercontent.com/76233721/187796502-af01a490-b974-4ade-8cb8-97e9b4fa2e5e.mov 

https://user-images.githubusercontent.com/76233721/187796690-7412a831-3b4d-4811-b9f9-880c177fbe00.mov





## 贡献

如果这对您有任何帮助，请随时 [请我喝咖啡或啤酒](https://www.buymeacoffee.com/gliptopoliB)


