Cabusora-Lab-5
==============
C2C Kevin Cabusora, Dr. York, ECE 382

Remote Control

# The Basic Idea

The purpose of this lab was to create a program that would use timer interrupts and pin interrupts to decode a remote control, and thus use the remote control to control some form of functionality on the MSP430.

# Day 1

On Day 1, the objective was to use hook up an IR sensor to my MSP430, connect it to the logic analyzer, and then play around with the remote control to observe what would happen.

## Design

The IR sensor was connected to the GND, VCC, and XIN pins as shown below.

It was then connected to the logic analyzer as shown in the picture below.

I chose the Orion 076E0PV051 Remote Control, designated as #11 in the ECE lab.

## Testing

To test, I ran the logic analyzer, then pressed a random button and observed the logic analyzer output, which would give me the pulse length.  As per instruction, a logic 0 half pulse followed by a logic 1 half pulse of the same size is interpreted as a 0, while a logic 0 half pulse followed by a logic 1 half pulse of greater size is a 1. The screenshot is shown below.

Duration I observed by using the cursors on the logic analyzer to get me a precise value.  For the Timer A counts, besides the start and stop pulses, I used the Variables timer[1] and timer[0] arrays, and averaged them out to give me my Timer A values.

| Pulse                     | Duration (ms) | Timer A counts |
|---------------------------|---------------|----------------|
| Start logic 0 half-pulse  | 9.062         | 8922           |
| Start logic 1 half-pulse  | 4.489         | 4437           |
| Data 1 logic 0 half-pulse | 5.93          | 590            |
| Data 1 logic 1 half-pulse | 1.656         | 1633           |
| Data 0 logic 0 half-pulse | 5.96          | 588            |
| Data 0 logic 1 half-pulse | 5.41          | 432            |
| Stop logic 0 half-pulse   | 5.96          | 590            |
| Stop logic 1 half-pulse   | 43.01         | 3985           |

Using this I could then answer the following questions:

(1) How long did it take the timer to roll over?
 - 65.23 ms
(2) How long does each timer count last?
 - 1 us
(3) Annotate the picture below:

I then proceeded to press the following buttons per the handout, and interpret them into hex.

| Button | Code (Not including start and stop bits) | Code (in hex) |
|--------|------------------------------------------|---------------|
| 0      | 01100001101000001001000001101111         | 61A0D06F      |
| 1      | 01100001101000000000000011111111         | 61A000FF      |
| 2      | 01100001101000001000000001111111         | 61A0807F      |
| 3      | 01100001101000000100000010111111         | 61A040BF      |
| Power  | 01100001101000001111000000001111         | 61A0F00F      |
| VOL +  | 01100001101000000011000011001111         | 61A030CF      |
| VOL -  | 01100001101000001011000001001111         | 61A0B04F      |
| CH+    | 01100001101000000101000010101111         | 61A050AF      |
| CH-    | 01100001101000001101000000101111         | 61A0D02F      |

# Day 2

Dr. York confirmed that my MSP430, IR sensor and C code indeed could take in my remote control's input.

# Basic Functionality

The objective was to create C code which used timer interrupts and a port 2 interrupt to turn an LED on and off with one button on the remote, as well as another LED on and off with a different button.

## Design

In the start5.c file, I put in statements which would toggle the green and red LEDs independently.  Here is the code I added.

````
	while(1)  {
		if (newIrPacket == TRUE) {
			if(irPacket == UP){
							P1OUT ^= BIT6;
						}else if(irPacket == DOWN){
							P1OUT ^= BIT0;
						}
````

As you can see, UP toggles BIT6 while DOWN toggles BIT0.

I then used the diagram supplied by the lab handout for basic functionality (upon much lecturing by an upset Dr. York about my failure to use it) to fill in the rest of my interrupt code.

```
#pragma vector = PORT2_VECTOR			// This is from the MSP430G2553.h file

__interrupt void pinChange (void) {

	int8	pin;
	int16	pulseDuration;			// The timer is 16-bits

	if (IR_PIN)		pin=1;	else pin=0;

	switch (pin) {					// read the current pin level
		case 0:						// !!!!!!!!!NEGATIVE EDGE!!!!!!!!!!
			pulseDuration = TAR;

			if ((pulseDuration > minLogic0Pulse) && (pulseDuration < maxLogic0Pulse)){
				irPacket = (irPacket<<1) | 0;
			}
			if ((pulseDuration > minLogic1Pulse) && (pulseDuration < maxLogic1Pulse)){
				irPacket = (irPacket<<1) | 1;
			}
			packetData[packetIndex++] = pulseDuration;
			TACTL = 0;
			//TACTL = MC_0;
			//TAR = 0x0000;
			LOW_2_HIGH; 				// Setup pin interrupr on positive edge
			break;

		case 1:							// !!!!!!!!POSITIVE EDGE!!!!!!!!!!!
			TAR = 0x0000;						// time measurements are based at time 0

			TA0CCR0 = 0x2710;
			TACTL = ID_3 | TASSEL_2 | MC_1 | TAIE;

			HIGH_2_LOW; 						// Setup pin interrupr on positive edge
			break;
	} // end switch

	P2IFG &= ~BIT6;			// Clear the interrupt flag to prevent immediate ISR re-entry

} // end pinChange ISR
```

In the start5.h file, I then added my constants for the hex code of the different buttons, as well as the pulse values.

```
#define		averageLogic0Pulse	590
#define		averageLogic1Pulse	1633
#define		averageStartPulse	4437
#define		minLogic0Pulse		averageLogic0Pulse - 200
#define		maxLogic0Pulse		averageLogic0Pulse + 200
#define		minLogic1Pulse		averageLogic1Pulse - 200
#define		maxLogic1Pulse		averageLogic1Pulse + 200
#define		minStartPulse		averageStartPulse - 200
#define		maxStartPulse		averageStartPulse + 200

#define		PWR		0x61A0F00F
#define		ONE		0x61A000FF
#define		TWO		0x61A0807F
#define		THR		0x61A0408F

#define		RIGHT	0x61A030CF
#define		LEFT	0x61A0B04F
#define		UP		0x61A050AF
#define		DOWN	0x61A0D02F
```

## Testing/Debugging

Initially my code did not work.  I talked with Dr. York about how perhaps the interrupts were not working correctly.  I consulted with C2C Erik Thompson and found it to be an easy fix:  My UP and DOWN codes were switched with LEFT and RIGHT respectively, so pressing up and down did nothing and I couldnt figure out why.  The source of this error was that the directional buttons correspond to VOL and CH up and down, and I switched the order.

There was also a bit of trouble with the IR sensor not intially receiving the input, so with C2C Thompson's advice I increased the min and max logicpulse values.  Iteratively I increased them from the standard 100 to 200 and found that I had a better result.  

On 10 September 2014, Dr. York observed that my green and red LEDs did indeed toggle correctly.

# Documentation
- C2C Jacob Lawson and I worked on Day 1 material together, utilizing the same #11 remote.
- Dr. York walked me through the function of the interrupts, as well as the particulars of Timer A.
- C2C Kyle Jonas helped me transcribe my logic analyzer output by relaying them to me while I typed them in.
- C2C Erik Thompson helped me in forming my statements for toggling the LED's, as well as pointing out how I had switched the directional buttons as well as increasing my logicvalue range.
