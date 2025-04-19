![Abstraction Layers](https://github.com/user-attachments/assets/4a5dc8a7-613e-40e4-b362-18ea9f5a9b1d)

# **Abstraction Layers of Firmware Development**

A microcontroller (MCU) is literally a small computer on a single chip that can control electronic devices. So, just like your desktop computer, it includes some sort of processor, memory, and input/output ports. But to program an MCU, we need to have a different approach. Unlike Windows or Mac, microcontrollers don't usually have operating systems. Most MCUs run a single program that you write, and they just do what you tell them forever, in a loop. This is where your code directly controls the hardware. However, in more complex projects, you might use a Real-Time Operating System (RTOS), but it is out of the scope of this article.

When programming from scratch, also known as bare-metal programming, you work directly with the MCU's hardware by configuring its memory-mapped registers yourself. This gives you full control but also means you need to read datasheets and reference manuals to understand how to activate features like timers or GPIO pins, how to configure ports, how to get communication protocols to work, and so on. It's like building a piece of furniture from raw wood; you do everything by hand, which takes time and knowledge, but results in something highly customized and efficient. In contrast, using abstraction layers where hardware-specific details have already been taken care of (like STM32's HAL or tools like CubeMX) is more like assembling IKEA furniture, much easier and faster, but you trade off some flexibility and performance.

So, when you're working with microcontrollers, you don't always have to start from scratch. You can choose how closely you want to interact with the hardware by picking different abstraction layers. Each layer gives you a tradeoff between control, effort, and convenience. The closer you work to the hardware, the more control you have but it also takes more time and skill. Here's a detailed description of the abstraction layers.

---

### **Bare-metal**  
Bare-metal programming means writing code that runs directly on a microcontroller (MCU) without any operating system (like Windows, Linux, or even a small Real-Time Operating System). Your code talks straight to the hardware, controlling it at a very low level. It is like cutting down a tree, shaping the wood, and building a chair from scratch.

Imagine a microcontroller as a blank brain. It doesn't know anything until you tell it what to do. In bare-metal programming, you are completely in charge! You set up the microcontroller's internal parts like timers, communication ports, and input/output pins by writing to their control registers (special memory locations that control hardware). For this you need to know: How the hardware inside the microcontroller works (reading datasheets and reference manuals is essential). How to work with memory addresses and registers. Basics of embedded C programming (or Assembly if you want to go even lower). Since there's no OS overhead, your program is as fast and efficient as possible. Small memory use. You have full control over everything. But the downside is that it takes longer to setup and debug. Requires a deep understanding of the hardware. Harder to maintain large projects as you manage everything manually.

Example code using Bare-metal (LED Blink):

```c
#include "stm32f4xx.h"

int main(void)
{
    // Enable clock for GPIOA (bit 0 of AHB1ENR)
    RCC->AHB1ENR |= (1 << 0);

    // Set PA5 as output (bits 10:11 in MODER)
    GPIOA->MODER &= ~(3 << (5 * 2));  // Clear mode
    GPIOA->MODER |= (1 << (5 * 2));   // Set as output

    while (1)
    {
        // Set PA5 High
        GPIOA->ODR |= (1 << 5);
        for (volatile int i = 0; i < 1000000; i++); // crude delay

        // Set PA5 Low
        GPIOA->ODR &= ~(1 << 5);
        for (volatile int i = 0; i < 1000000; i++); // crude delay
    }
}

```
---

### **CMSIS**  
Cortex Microcontroller Software Interface Standard is a collection of software tools, header files, and standards that help you program ARM Cortex-M microcontrollers more easily and in a consistent way. It was created by ARM to give developers a common, standardized foundation to access the core features of Cortex-M CPUs, no matter which company's chip they are using (like STM32, NXP, TI, etc.). When you use CMSIS, you're still working very close to the hardware, like in bare-metal programming, but instead of manually calculating memory addresses and setting bits yourself, CMSIS gives you simple, readable functions and definitions. For example, if you want to enable an interrupt, instead of figuring out a memory address and setting the right bits, you can just call `NVIC_EnableIRQ()`. It's as if the trees have already been cut and the wood neatly labeled; you just choose the pieces you need and start from there.

CMSIS provides standardized register names and structure. It doesn't replace libraries like HAL that control peripherals (GPIO, UART, etc.); it focuses only on core processor tasks like interrupts, low-power modes, system timing, and processor configuration.

Example code using CIMSIS (LED Blink):

```c
#include "stm32f4xx.h"

int main(void)
{
    // Enable clock for GPIOA
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;

    // Set PA5 as output
    GPIOA->MODER &= ~(GPIO_MODER_MODE5_Msk);
    GPIOA->MODER |= (1 << GPIO_MODER_MODE5_Pos);

    while (1)
    {
        // Set PA5 High
        GPIOA->BSRR = GPIO_BSRR_BS5;
        for (volatile int i = 0; i < 1000000; i++);

        // Set PA5 Low
        GPIOA->BSRR = GPIO_BSRR_BR5;
        for (volatile int i = 0; i < 1000000; i++);
    }
}

```
---

### **Low-Layer drivers**  
Low-Layer (LL) libraries are a set of lightweight, hardware-focused libraries provided by microcontroller manufacturers to help you control the chip's peripherals (like GPIO, timers, UART, ADC) with very little overhead. LL libraries sit just one step above bare-metal programming; they don't hide much from you. They simply give you ready-made functions and macros that make working with hardware registers easier, faster, and less error-prone. For example, instead of manually writing to a register to turn on a pin, you can call a function like `LL_GPIO_SetOutputPin()`, which directly changes only the necessary bits without the extra checking and abstraction that HAL does. LL libraries are very close to the hardware and are designed to offer high performance, low memory usage, and fine control, almost like writing to registers by hand, but without worrying about making small mistakes.

You can think of LL libraries like using a powerful set of professional tools when building something, like shaping the woods by yourself, but with some very useful tools. You're still building everything yourself and have to know what you're doing, but the tools make it faster, cleaner, and less risky than using just your bare hands. And for that, some extra set of skills are required.

Example code using LL (LED Blink):

```c

#include "stm32f4xx_ll_gpio.h"
#include "stm32f4xx_ll_bus.h"

int main(void)
{
    // Enable clock for GPIOA
    LL_AHB1_GRP1_EnableClock(LL_AHB1_GRP1_PERIPH_GPIOA);

    // Configure PA5 as output
    LL_GPIO_InitTypeDef GPIO_InitStruct = {0};
    GPIO_InitStruct.Pin = LL_GPIO_PIN_5;
    GPIO_InitStruct.Mode = LL_GPIO_MODE_OUTPUT;
    GPIO_InitStruct.Speed = LL_GPIO_SPEED_FREQ_LOW;
    GPIO_InitStruct.OutputType = LL_GPIO_OUTPUT_PUSHPULL;
    GPIO_InitStruct.Pull = LL_GPIO_PULL_NO;
    LL_GPIO_Init(GPIOA, &GPIO_InitStruct);

    while (1)
    {
        LL_GPIO_SetOutputPin(GPIOA, LL_GPIO_PIN_5);
        LL_mDelay(500); // You would need to implement LL_mDelay or use SysTick.

        LL_GPIO_ResetOutputPin(GPIOA, LL_GPIO_PIN_5);
        LL_mDelay(500);
    }
}

```
---

### **Hardware Abstraction Layer**  
HAL (Hardware Abstraction Layer) is a set of libraries provided by microcontroller manufacturers like STMicroelectronics (for STM32) that makes it easier and faster to program microcontrollers without worrying too much about the deep, low-level details of the hardware. HAL sits higher above the hardware compared to LL or bare-metal programming. It gives you ready-made functions that handle all the complex setup steps in the background. For example, if you want to turn on an LED, instead of configuring clock settings, setting pin modes, and writing to output registers manually, you can simply call something like `HAL_GPIO_WritePin()`, and the library takes care of everything internally. HAL is designed to make programming simpler, more readable, and portable.

You can think of HAL like buying a ready-to-assemble IKEA furniture kit: you get a manual, all the parts are ready, and you just have to connect them — no need to cut wood, measure, or drill yourself. It's much faster and easier to get a working result, but you sacrifice some flexibility and performance, because you don't control every small detail yourself. For most simple and medium-complexity projects, HAL is a great choice because it lets you focus on your application rather than getting stuck in low-level setup.

Example code using HAL (LED Blink):

```c
#include "stm32f4xx_hal.h"

GPIO_InitTypeDef GPIO_InitStruct = {0};

int main(void)
{
    HAL_Init();

    __HAL_RCC_GPIOA_CLK_ENABLE();

    // Configure PA5 as output
    GPIO_InitStruct.Pin = GPIO_PIN_5;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

    while (1)
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
        HAL_Delay(500);

        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
        HAL_Delay(500);
    }
}

```

---
### **A side by side overview**

| **Layer**      | **Ease of Use** | **Speed & Size** | **Control** | **Example**                          |
|----------------|-----------------|------------------|-------------|--------------------------------------|
| **Bare-Metal** | Very Hard       | Very Fast        | Full        | Manual register writes               |
| **CMSIS**      | Hard            | Very Fast        | Full (with better names) | `GPIOA->BSRR = GPIO_BSRR_BS5` |
| **LL**         | Medium          | Fast             | High        | `LL_GPIO_SetOutputPin()`             |
| **HAL**        | Easy            | Slower           | Medium      | `HAL_GPIO_WritePin()`                |
---

### **Which Layer Should You Choose?**

- **Bare-Metal / CMSIS** → When you care about every clock cycle and want full control.  
- **LL** → When you want fast and easy low-level hardware access without writing messy register code.  
- **HAL** → When you want to build faster, focus on the application, and don't mind sacrificing some performance.  
