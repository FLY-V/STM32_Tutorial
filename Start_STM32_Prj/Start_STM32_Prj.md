#  使用STM32CubeMX创建一个STM32工程	         --	点亮LED灯  

## 1.STM32CubeMX界面  

打开STM32CubeMX，出现创建CubeMX工程界面。  

![Clock_Config](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/New_Project.jpg)  

左边可以快速打开现有工程，中间是创建新工的选项（可根据控制器MCU型号创建、根据ST官方板创建和根据多种方式创建），右边是STM32CubeMX的软件管理（检查、安装或删除器件包，更新软件等）。  

上方菜单栏中，“File”可新建工程或加载工程等；“Windows”配置界面字体大小；“Help”可查看帮助文档、检查更新和管理器件包等。  

## 2.创建工程前的工作  

### · 2.1 在线安装器件包  

由于一般是通过MCU型号创建新工程的，因此在创建工程前需要确保已经安装了相应MCU的器件包。  

![Clock_Config](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/Software_Packages.jpg)  

在菜单栏的“Help”中，选择“Manage embedded software packages”。  

![Install_Packages](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/Install_Packages.jpg)  

在该界面中，先点击下方的“Refresh”刷新，在“STM32Cube MCU Packages”中勾选上需要的器件包后，点击“Install Now”，即可在线安装芯片器件包。  

### · 2.2 另一种安装器件包的方法  

使用离线包进行本地安装。从STM32CubeMX官网中下载对应的[STM32CubeMX器件包](https://www.st.com/zh/embedded-software/stm32-embedded-software.html)，通过“From Local Folder”选择进行安装。  

## 3.创建STM32CubeMX工程  

### · 3.1 根据MCU创建工程（ACCESS TO MCU SEKECTOR）  

在确保器件包安装好后，即可开始创建工程。  

输入需要的芯片型号（如STM32F411CE），在右下角选择对应封装类型的芯片型号，型号对应芯片的产品类型、特征、引脚数、Flash大小、封装类型等。参见芯片命名方法[STM32_Name](#STM32_Name)。  

点击右上角的"Start Project"，打开配置工程界面，根据需要对芯片外设进行配置。  

![From_a_MCU](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/From_a_MCU.jpg)  

### · 3.2 工程配置界面  

在该界面中主要对“Pinout & Configuration”、“Clock Configuration”和“Project Manager”进行设置。  

![Config_Interface](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/Config_Interface.jpg)  

### · 3.3 工程选项设置  

在Project Manager -> Project中，配置工程名称（Project Name）、路径（Project Location）和工具链（Toolchain / IDE），以及设置堆栈大小（Linker Setting）等。  

![Project_Manager](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/Project_Manager.jpg)  

在Project Manager -> Code Generator中，配置软件包选项（是否拷贝所有外设库等）、配置生成文件（每个外设单独生成‘.c/.h’文件等）以及配置HAL库（设置未使用的引脚为模拟输入、使能断言），这部分中使用默认设置即可。  

在Project Manager ->Advanced Settings中，选择配置各个外设的驱动库，可选择HAL库或LL库。一般默认使用HAL库，对代码内存有要求时，使用LL库，可以压缩代码段（没玩过，听说LL库更接近底层，函数封装更少）。  

**在配置芯片外设前需要注意的是配置调试接口和时钟源。**    

### · 3.4 配置时钟源和系统调试接口  

默认使用的是内部高速时钟HSI，但是这里使用高速外部时钟源时，选择[HSE](#HSE)，设置为BYPASS Clock Soure模式，配置为使用外部时钟源。  

![RCC_BYPASS](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/RCC_BYPASS.jpg)  

调试接口根据实际电路可以选择JTAG和串行模式（Serial Wire），一般选择Serial Wire模式，占用IO少。这里设置为Serial Wire模式。  

![SYSDEBUG_Serial](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/SYSDEBUG_Serial.jpg)  

经过上面的配置后，只是使能了外部时钟接口，并没有完成时钟源的配置，还需要对时钟树进行配置。  

###  · 3.5 配置时钟树（Clock Configuration）  

STM32时钟源主要有4个，高速外部时钟（HSE），高速内部时钟（HSI），低速外部时钟（LSE）和低速内部时钟（LSI），其中低速时钟源（LSE和LSI）主要为RTC提供时钟，在此暂时没用到。  

在时钟树配置中，根据实际电路的外部晶振设置Intput frequency，这里设置为25MHz，并将PLL Source Mux选择为HSE，经过[PLL](#PLL)时钟输入分频因子M（1~63）分频后，这里分频因子M设置为/25，成为VCO的时钟输入。  

VCO输入时钟经过 VCO倍频因子 N倍频之后，成为VCO时钟输出，VCO 时钟必须在192~836MHz之间。我们配置 N 为 200，则 VCO 的输出时钟等于200MHz。  

VCO输出时钟之后经过PLLCLK分频因子p配置为/2，输出PLL时钟，此时PLLCLK为100MHz。  

配置系统时钟[SYSCLK](#SYSCLK)选择器System Clock Mux，选择PLLCLK作为时钟源。  

最后根据需要配置外设总线时钟。如下图，时钟树配置。  

![Clock_Config](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/Clock_Config.jpg)  

### · 3.6 配置外设初始化（Pinout & Configuration）  

完成以上配置工程选项、RCC时钟树和调试接口后就可以开始配置外设初始化了。  

这里以最简单的GPIO初始化为例，开始创建点灯工程（_类似学编程的printf("Hello World!")_）。  

---

由于是直接操作GPIO，因此直接在右侧的“Pinout View”选择需要配置的引脚，这里选择配置PA0，设置为GPIO_Output。设置后，可以在选择左侧"System Core -> GPIO"，在中间出现所有外设模块对应GPIO引脚的设置，查看“GPIO”选项。  

在“GPIO”选项中选择PA0后，在可以看到一共有5个选项：

- “GPIO output level”            ：输出极性。设置为“Low”。
- “GPIO mode”                       ：输出模式。设置为“output”。
- “GPIO-Pull-up/Pull-down”  ：上下拉模式，设置引脚内部接上拉电阻或下拉电阻。这里设置为“Pull-down”。
- “Maximum output speed” ：最大控制速度，引脚所能控制输出的最大速率，根据不同的STM32系列，有多种不同的控制速率。这里设置为"Low"。
- “User Label”                         ：用户标签，相当于给引脚设置别名，这里为该引脚设置为“LED”，表明该引脚是LED控制引脚。  

![GPIO_SET](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/GPIO_SET.jpg)  

### ·3.7 生成STM32工程  

完成以上工作后，在"Project Manager -> Project"中选择“Tookchain / IDE”为“makeflie”。  

最后点击右上方的"GENERATE CODE"即可生成基于makeflie工具链的STM32工程。  

**根据不同需要，在此可以选择不同的编译工具，常用的有“MDK-ARM V5”（Keil 5）、“EWARM V8”(IAR v8)。**  

![Code_Generation](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/Code_Generation.jpg)  

## 4.编写应用程序 -- 控制LED亮灭  

### · 4.1 编译测试  

在完成代码生成后，可以先编译一编工程，测试工程是否有问题。  

在目录下执行make，生成.elf、.hex以及.bin文件说明编译通过。  

![make](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/make.jpg)  

### · 4.2 工程文件目录说明  

打开STM32工程，可以查看这个工程的目录文件。  

其中：

- build		: 执行make命令后生成的目标文件夹，所有的过程文件和目标文件都在里面，包括最终需要的.elf、.hex以及.bin文件。
- Drivers    ： 驱动文件库，包括两个文件夹
- CMSIS    ：与内核相关的驱动文件；
- STM32F4xx_HAL_Driver    : 各个外设的HAL库驱动文件。
- Inc            ：包含用户配置应用的头文件。
- Src            ：包含用户应用程序的源码。  

![Project_Code](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/Project_Code.jpg)  

在编写程序时，一般只需要修改Inc和Src目录下的文件，其中以下几个是比较重要的文件：

- stm32f4xx_hal_conf.h                  ：用来配置库的头文件，通过配置宏将驱动库文件包含到工程中。
- stm32f4xx_it.h和stm32f4xx_it.c ：中断相关的函数都在这个文件编写，暂时为空。
- main.h和main.c                             ：main 函数文件，应用函数头文件和源文件。
- startup_STM32f411xe.s                ：包含了对应编译平台的汇编启动文件，实际使用时根据编译平台来选择。
- stm32f4xx_hal_msp.c                   ：该文件提供了MSP（MCU Specific Package）初始化和反初始化代码。
- system_stm32f4xx.c                     ：包含了 STM32 芯片上电后初始化系统时钟、扩展外部存储器用的函数。  

### · 4.3 编写LED程序  

在main.h中定义LED控制的宏函数。添加`LED_ON()`和`LED_OFF()`宏函数，通过调用`HAL_GPIO_WritePin`函数控制LED引脚。  

```c
/* Private defines -----------------------------------------------------------*/
#define LED_Pin 		GPIO_PIN_0
#define LED_GPIO_Port	GPIOA
/* USER CODE BEGIN Private defines */
#define LED_ON()		HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET)
#define LED_OFF()		HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_RESET)
/* USER CODE END Private defines */
```

在main.c中添加`Delay`延时函数，并在`main`函数中添加控制代码。  

```c
/* USER CODE BEGIN PFP */
void Delay(uint32_t x);
/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
void Delay(uint32_t x)
{
	uint32_t i = x;
	while(i--);
}
/* USER CODE END 0 */

...

int main(void)
{
	...
    
	/* USER CODE BEGIN WHILE */
	while (1)
	{
		/* USER CODE END WHILE */

		/* USER CODE BEGIN 3 */
		LED_ON();
		Delay(0xFFFFFF);
		LED_OFF();
		Delay(0xFFFFFF);
	}
	/* USER CODE END 3 */
}
```

在添加自己的程序代码时需要注意，将代码添加到`USER CODE BEGIN XXX`和`USER CODE END XXX`之间。否则再重新使用CubeMX生成工程时会**清除**在这两段以外的用户代码。  

### 4.4 编译LED程序并烧写测试  

添加好代码后，重新执行make命令编译程序，生成.elf、.hex以及.bin文件，并发现生成文件中的text段大小发生变化，说明程序的更改已生效。  

![LED](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/LED.jpg)  

此时可以使用仿真器将.bin文件或使用串口将.hex文件烧写到电路板的STM32芯片中，重新复位即可观察到连接在PA0引脚的LED定时亮灭。  

___emmmm，并没有板子，因此在这里不具体讲如何烧写程序，但是能看到灯确实定时亮灭了。___  

到这里已经实现了通过makeflie编译STM32点灯程序，如果是在生成STM32工程时选择了其他的编译工具，测试代码也是一样的，只是编译的操作过程有区别。  

---
###### [STM8 和 STM32 命名方法]：
![STM32_Name](https://github.com/FLY-V/Imgs/blob/master/STM32_Tutorial/STM32_Name.jpg)  

###### [HSE]：
HSE是高速的外部时钟信号，可以由有源晶振或者无源晶振提供，频率从 1-50MHz 不等。当使用有源晶振时，时钟从 OSC_IN 引脚进入，OSC_OUT引脚悬空为高阻态，当选用无源晶振时，时钟从 OSC_IN 进入，OSC_OUT输出，并且要配谐振电容。  

###### [PLL]：
PLL的主要作用是对时钟进行倍频，然后把时钟输出到各个功能部件。  

###### [SYSCLK]：
系统时钟(SYSCLK)来源可以是：HSI、PLLCLK、HSE，具体的由时钟配置寄存器 RCC_CFGR的 SW 位配置。  