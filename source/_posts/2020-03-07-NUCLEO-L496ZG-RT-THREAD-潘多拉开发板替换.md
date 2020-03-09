title: '2020-03-07[NUCLEO_L496ZG][RT-THREAD]潘多拉'
author: Thomas Li
abbrlink: 3209598366
tags: []
categories: []
date: 2020-03-07 17:58:00
---
1.本人有一块NUCLOE L496ZG开发板：
2.RT-THREAD 看起来还比较好用，
所以记录一下，每次从潘多拉开发板转到NUCLO L496ZG需要多少步骤:

## 1. 下载 RT-THREAD源码：

github地址：https://github.com/RT-Thread/rt-thread.git  这个权威一些

gitee 下载地址： https://gitee.com/rtthread/rt-thread.git 这个速度快一些

## 2.keil 修改

打开工程：

#### 替换MCU:

![filename already exists, renamed](/images/pasted-2.png)
#### 替换宏定义： STM32L496xx

![filename already exists, renamed](/images/pasted-3.png)

#### 替换board.c中 MCU RCC时钟


```
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};
  RCC_PeriphCLKInitTypeDef PeriphClkInit = {0};

  /** Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_MSI;
  RCC_OscInitStruct.MSIState = RCC_MSI_ON;
  RCC_OscInitStruct.MSICalibrationValue = 0;
  RCC_OscInitStruct.MSIClockRange = RCC_MSIRANGE_6;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_MSI;
  RCC_OscInitStruct.PLL.PLLM = 1;
  RCC_OscInitStruct.PLL.PLLN = 40;
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
  RCC_OscInitStruct.PLL.PLLQ = RCC_PLLQ_DIV2;
  RCC_OscInitStruct.PLL.PLLR = RCC_PLLR_DIV2;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }
  /** Initializes the CPU, AHB and APB busses clocks 
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_4) != HAL_OK)
  {
    Error_Handler();
  }
  PeriphClkInit.PeriphClockSelection = RCC_PERIPHCLK_LPUART1;
  PeriphClkInit.Lpuart1ClockSelection = RCC_LPUART1CLKSOURCE_PCLK1;
  if (HAL_RCCEx_PeriphCLKConfig(&PeriphClkInit) != HAL_OK)
  {
    Error_Handler();
  }
  /** Configure the main internal regulator output voltage 
  */
  if (HAL_PWREx_ControlVoltageScaling(PWR_REGULATOR_VOLTAGE_SCALE1) != HAL_OK)
  {
    Error_Handler();
  }
}
```

#### 修改board.h

```
#define STM32_FLASH_START_ADRESS       ((uint32_t)0x08000000)
#define STM32_FLASH_SIZE               (1024 * 1024)
#define STM32_FLASH_END_ADDRESS        ((uint32_t)(STM32_FLASH_START_ADRESS + STM32_FLASH_SIZE))

#define STM32_SRAM1_SIZE               (256)
#define STM32_SRAM1_START              (0x20000000)
#define STM32_SRAM1_END                (STM32_SRAM1_START + STM32_SRAM1_SIZE * 1024)
```

此时应该可以编译通过



## 修改env配置：
##### 不同IDE需要修改如下几个ld文件

![filename already exists, renamed](/images/pasted-5.png)
##### 修改方式如下
![filename already exists, renamed](/images/pasted-4.png)

L496ZG flash是1M ， RAM是320K但是是由256加上一块128K flash地址(我们暂时还用不到它，先不管他)


0x10000000 + 0x10000

![filename already exists, renamed](/images/pasted-6.png)



```
LR_IROM1 0x08000000 0x00100000  {    ; load region size_region
  ER_IROM1 0x08000000 0x00100000  {  ; load address = execution address
   *.o (RESET, +First)
   *(InRoot$$Sections)
   .ANY (+RO)
  }
  RW_IRAM1 0x20000000 0x00040000  {  ; RW data
   .ANY (+RW +ZI)
  }
}

```

## 修改Kconfig

这个还是需要修改的，每次scons menuconfig配置都需要修改。

SConscript
这个文件中：
修改如下：

![filename already exists, renamed](/images/pasted-8.png)

## 修改template 模板：
这一步其实就是scons需要根据这个模板生成工程


###  重新生成工程

重新生成工程需要使用 env 工具。

#### 重新生成 rtconfig.h 文件

在 env 界面输入命令 menuconfig 对工程进行配置，并生成新的 rtconfig.h 文件。如下图所示：


![filename already exists, renamed](/images/pasted-9.png)


主要要修改console 为lpuart1 

![filename already exists, renamed](/images/pasted-10.png)

###  修改 Kconfig 选项

在本小节中修改 `board/Kconfig` 文件的内容有如下两点：

- 芯片型号和系列
- BSP 上的外设支持选项

芯片型号和系列的修改如下表所示：

| 宏定义             | 意义     | 格式               |
| ------------------ | -------- | ------------------ |
| SOC_STM32F103RB    | 芯片型号 | SOC_STM32xxx       |
| SOC_SERIES_STM32F1 | 芯片系列 | SOC_SERIES_STM32xx |

关于 BSP 上的外设支持选项，一个初次提交的 BSP 仅仅需要支持 GPIO 驱动和串口驱动即可，因此在配置选项中只需保留这两个驱动配置项，如下图所示：

![filename already exists, renamed](/images/pasted-11.png)


### 生成工程

使用 env 工具输入命令 scons --target=mdk5 重新生成工程

