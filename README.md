# 电控组第二次培训作业

- **马晨曦**
- ***20263001274***

## 作业内容

这次作业使用 STM32CubeMX 配置三个 LED 引脚，然后用 CLion 编写流水灯程序。

三个 LED 每 500 ms 轮流点亮，顺序为：

```text
LED0 → LED1 → LED2 → LED0
```

## 引脚配置

在 CubeMX 中把三个引脚设置了 User Label：

| LED | 引脚 |
|---|---|
| LED0 | PA0 |
| LED1 | PA1 |
| LED2 | PA2 |

这三个 LED 都是高电平点亮。

## 实现方法

我没有使用 `HAL_Delay()`，而是使用 `HAL_GetTick()` 来完成非阻塞延时。

程序一开始先定义两个变量：

```c
static uint32_t led_tick = 0U;
static uint8_t led_step = 0U;
```

- `led_tick` 用来记录上次切换 LED 的时间。
- `led_step` 用来记录现在应该亮哪个 LED。

`HAL_GetTick()` 会返回程序已经运行的毫秒数。主循环不断检查当前时间和上次切换时间相差多少。如果相差达到 500 ms，就切换一次 LED。

核心代码如下：

```c
if ((uint32_t)(HAL_GetTick() - led_tick) >= 500U)
{
    led_tick += 500U;

    switch (led_step)
    {
    case 0U:
        HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, LED_ON);
        HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, LED_OFF);
        HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, LED_OFF);
        break;

    case 1U:
        HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, LED_OFF);
        HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, LED_ON);
        HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, LED_OFF);
        break;

    case 2U:
        HAL_GPIO_WritePin(LED0_GPIO_Port, LED0_Pin, LED_OFF);
        HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, LED_OFF);
        HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, LED_ON);
        break;
    }

    led_step++;
    if (led_step >= 3U)
    {
        led_step = 0U;
    }
}
```

这段代码每 500 ms 只点亮一个 LED，并关闭另外两个 LED。执行完最后一个 LED 后，再把状态改回第一个 LED。
