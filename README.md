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
