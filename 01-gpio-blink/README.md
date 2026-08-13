# STM32_Embedded_Journey
I am sharing repositories that belongs to STM32 learning projects.

#Repo01 : STM32F407-BARE-METAL-LED-BLINK
## WHAT DID I DO:
I did bare-metal coding via STM32F407-DISC1 in order to LED blink in STM32CubeIDE. This code does blink for LED13. 
## CONCEPTUAL MAP:
1)	Why is clock exist ?
Digital circuits work synchronous manner. Therefore, clock signals are needed to organize the how and when digital circuits elements work together. Firstly, (chip is reset form.) clock signals of all peripherals are closed. The reason for this, clock signals are not sent for peripherals that already does not work. Because of, clock signals should be opened manuelly before working.  
2)	What is RCC and why is it needed ?
RCC (Reset Clock and Control) is a peripheral and includes a lot of registers inside. It’s task is managing and keeping a list that shows which peripheral’s clock is open or not.
3)	What is AHB1ENR ?
AHB1ENR can be imagined like a key that open or close the peripherals which connected to AHB1 bus. For instance, if GPIOD port (etc..) is exist over AHB1 bus, AHB1ENR register should be used to open clock signal of GPIOD port.
4)	What is MODER and ODR ?
MODER is a register that determines the which state of GPIO ports. Each pin occupies 2 bits. Because there are 4 states : 00_input, 01_output, 10_alternate function, 11_analog and 2^2=4. ODR register represents final state of output mode of MODER. It occupies 1 bit : 0 or 1.
## THE MISTAKES I EXPERIENCED 
1)	warning "FPU is not initialized, but the project is compiling for an FPU. Please initialize the FPU before use.". Like GPIO peripherals, FPU comes closed manner when chip is reset defaultly. Therefore, before runtime FPU should be opened manuelly in hardware. 
## KNOW HOW
1)	GPIOD->MODER &= ~(3 << (13*2)); In this code, 3 is written. Because the aim is setting 2 bits and 3 means i binary format 11. 
2)	Busy wait delay that is used code does not represent exact time. The number that belongs inside for loop, is tour for loop like 1000000. These tours signify different amount of time according to speed of clock and Mhz of processor. In addition, if interrupt realizes CPU brokes the loop to process command. Therefore, more time might be needed. Also, this is not portable. If same code is ported to other chip, time changes. Because number of repetition signifies different amount of time in different hardwares. Namely, there is no determinism in this way.


## BONUS RESEARCH
1)	Are bare-metal coding and register-level coding same ? 
These are not same. Bare-metal coding is code that works in hardware without operating system (RTOS, Linux etc.) directly. There are no scheduler, a Kernel or task manager. In register-level coding, it is achieved to peripherals via register addresses directly without HAL (Hardware Abstraction Layer). In HAL, built-in functions are called rather than accesing to registers directly. 
2)	Why register level coding ?
It is needed for maximum speed, clearness and getting control.
3)	Why bare-metal coding ?
It is needed to work deterministic manner directly without timing uncertainty rather than using operating system.

