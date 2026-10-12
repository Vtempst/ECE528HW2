# ECE528HW2

## Section I: Review Questions

1. What is a hardware timer? What kinds of tasks do embedded systems commonly use hardware timers for?

- A hardware timer is a peripheral component inside a microcontroller that counts clock signals to keep track of time. It can trigger an event at specific times. The tasks it can do is generate delays, periodic interrupts, and pulse width modulation signals.

2. What is a clock signal used for in embedded systems? List the primary clock sources available on the MSP432P401R microcontroller and briefly describe each clock source.

- Clock signal is used to synchronize and control the timing of events within a microcontroller such as updating registers and executing isntructions. The primary clock sources available on the MSP432 microcontroller are:
    - LFXT (Low-Frequency Crystal Oscillator) - provides a low-frequency clock 
    - HFXT (High-Frequency Crystal Oscillator) - provides a high-frequency clock
    - DCO (Digitally Controleld Oscillator) - generate adjustable frequencies
    - VLO (Very Low-Power Oscillator) - generates a clock that consumes little power
    - REF0 (Reference Oscillator) - alternative to LFXT, low frequency and lower power
    - MODOSC (Module Oscillator) - provide clocks to peripherals such as ADC
    - SYSOSC (System Oscillator) - provides a clock for system management

3. Consider a 16-bit timer (e.g. Timer_A configured in up mode). If it counts upward, what is the maximum count value before it rolls over? Show your work.

maximum count = 2^n = 2^16 = 65536

However, it starts counting at 0 so the maximum count value before it rolls over is 65536 - 1 = 65535.

4. If a 24-bit timer counts upward from zero at 10 MHz, what is its overflow period? Show your work.

maximum count = 2^24 = 16777216
(Δt) = (T) * (N) = (1/10000000) * (16777216)
(Δt) = 1.6777216 seconds

The timer can count up to 1.6777216 seconds before it rolls over to 0.

5. Describe the SysTick timer on the MSP432P401R microcontroller. Provide two example applications for SysTick in embedded systems. Which clock sources can the SysTick timer use and how can they be selected?

SysTick is the integrated system timer on the MSP432 that provides a 24-bit decrementing counter. It counts down from a specified value to zero using clock signals and once it reaches zero, it generates an interrupt and restarts counting.

The two applications for SysTick in embedded systems is to generate delays and generating periodic interrupts.

The clock sources that the SysTick timer can use are the CPU clock. However, no external clocks are supported. It can select which clock is used by configuring bit 2 in the SysTick Control and Status Register (STCSR).

6. How many registers does the SysTick timer have on the MSP432P401R microcontroller and what is the purpose of each? Name them and provide a short description per register.

    1. SysTick Control and Status Register (STCSR)
        - enables the counter, set the SysTick interrupt, and determine counter status
    2. SysTick Reload Value Register (STRVR)
        - specify the start value to load into the current value register when the counter reaches 0.
    3. SysTick Current Value Register (STCVR)
        - provide the counter's current value
    4. SysTick Calibration Value Register (STCR)
        - allow software to scale to any required speed by dividing and multiplying

7. Describe the Timer_A peripheral on the MSP432P401R microcontroller. List the main operating modes of Timer_A and briefly explain each.

Timer_A is a 16-bit timer with many capture/compare registers. Timer_A has four modes of operation: 
    1. Stop - stops the timer
    2. Up - timer increments from zero to the value of TAxCCR0, timer repeatedly executes this operation
    3. Continuous - timer decrements from zero to 0xFFFF, timer repeatedly executes this operation
    4. Up/Down, counts up from zero to the value of TAxCCR0 and counts down to zero, timer repeatedly executes this operation

8. Briefly state the purpose of the following Timer_A registers.
    - TAxCTL
        - selects Timer_A clock source
        - selects divider for the input clock
        - controls which mode of operation to enable for Timer_A such as stop, up, continuous, and up/down
        - Clearing Timer_A
        - Enables Interrupt and has an interrupt flag
    - TAxCCTLn
        - Determines whether data is captured on the rising edge, falling edge, or both
        - Determines whether to synchronize the capture input signal with the timer clock
        - Determines whether to be in compare or capture mode
        - Enables capture/compare interrupts
    - TAxR 
        - stores the count value of Timer_A
    - TAxCCRn
        - Compare mode: holds the data for comparison to the timer value, TAxR
        - Capture mode: Timer value, TAxR, is copied into TAxCCRn
    - TaxIV
        - interrupt vector value for all other CCIFG flags and TAIFG, identifies where an interrupt occured 
    - TAxEX0 
        - further divide clock source from 2 to 8.

9. What is PWM? Provide at least three example applications that use PWM in embedded systems.

- Pulse width modulation (PWM) is a technique that controls the amount of power delivered by an electrical signal through rapidly switching a digital signal from on and off. The applications that use PWM are controlling motor speed, LED brightness, and audio generation.

10. How many Timer_A modules are available and how many PWM outputs can each module provide?

Timer_A has four available modules: TA0, TA1, TA2, and TA3. Each module can provide 4 PWM outputs.

11. Describe the steps needed to initialize P2.4 as a PWM output driven by Timer_A0 CCR1 (TA0.1).

Use the Timer_A2_PWM.c driver provided in Lab 1 (Motor Control) as a reference.

Include C code such that:
    - The P2.4 pin is configured to use its primary module function (Timer_A output)
    - SMCLK is selected as the clock source for Timer_A
    - The timer clock is divided by 2
    - Up/down mode is selected
    - The output mode is configured as Toggle / Reset by setting the appropriate bits of the OUTMOD field in the CCTL[1] register
    - The PWM signal has a period of 20 ms and a duty cycle of 50%

P2.4 pin is configured to use its primary module function by setting bit 4 in the SEL0 register and clearing bit 4 in the SEL1 register. By setting bit 4 of the DIR register, the pin becomes an output. SMCLK is a 12 MHZ clock source and the timer clock is divided by 2. The timer frequency is 6 MHz. To get the value of CCR0, multiply the timer frequency and the period. The PWM signal has a period of 0.02 s, therefore, 6000000 x 0.02 = 120000 total counts. Since up/down mode is selected, it takes 60000 counts to count up and another 60000 counts to count down. 60000 is the value needed in CCR0. To create a duty cycle of 50%, multiply CCR0 by 0.5 and input the value in CCR1. The output mode is configured as Toggle / Reset by setting the appropriate bits to 010 in the CCTL[1] register. No additional division is required so EX0 is cleared to 0. CTL is set to configure Timer_A0 to use SMCLK, divide by 2, and use up / down mode.

```
P2->SEL0 |= 0x10;
P2->SEL1 &= ~0x10;
P2->DIR |= 0x10;
TIMER_A0->EX0 = 0x0000;
TIMER_A0->CCR[0] = 60000;
TIMER_A0->CCR[1] = 30000;
TIMER_A0->CCTL[1] |= 0x0040;
TIMER_A0->CTL |= 0x0270;
```

12. Describe the steps needed to configure Timer_A0 to generate a periodic interrupt every 2 ms with an interrupt priority level of 2. Note that Timer_A0 has an interrupt number of 8.

Use the Timer_A0_Interrupt.c driver provided in Lab 1 (Motor Control) as a reference.

Include C code such that:
    - SMCLK is selected as the clock source for Timer_A with a prescaler value of 1 (which means the ID field will be 00b)
    - The TAIDEX field in the TAxEX0 register has a value of 000b (divide by 1)
    - Up mode is selected

To select SMCLK as the clock source for Timer_A, bit 9-8 of CTL register needs to be 10. For a prescaler value of 1, bit 7-6 of CTL needs to be 00. To select up mode, it requires setting bit 5-4 to 01. To get the value of CCR0, multiply the time frequency and period. SMLCK is a 12 MHz clock, and it is divided by 1. The timer frequency is 12 MHz. The period is 2 ms or 0.002 seconds. CCR0 = 12000000 x 0.002 = 24000 counts.

```
TIMER_A0->CTL &= ~0x0030;
TIMER_A0->CTL |= 0x0200;
TIMER_A0->CCTL[0] |= 0x0010;
TIMER_A0->CCR[0] = (24000 - 1);
TIMER_A0->EX0 = 0x0000;
NVIC->IP[2] = (NVIC->IP[2] & 0xFFFFF00F) | 0x00000060;
NVIC->ISER[0] |= 0x00000100;
TIMER_A0->CTL |= 0x0014;
```
## References
- MSP432P4xx SimpleLink™ Microcontrollers Technical Reference Manual