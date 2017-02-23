# XBee-Darknet
Full software stack for in-town (~3 mile NLOS range) slow-speed communications darknet/meshnet

# Goal
Create software and instructions for assembling the hardware/firmware/software required to get a darknet set up in town to enable legal communications with friends.  Build kit will be within FCC regulated power limitations (1W), and the software will enable encrypted (if speed is reasonable) communications for small files and basic text messaging.

# Plan:
1. Acquire hardware required to perform very long range (65 miles LOS, or at least 3 miles NLOS) communications
-compile this list and refer to it in this repo so anyone can reproduce this project
-hardware stack will not include wifi or ethernet.  Only 900MHz XBee for darknet comms.
2. Write python script that will take in messages and/or files, and send them over 900MHz
-stations will act as repeaters as well, re-broadcasting received messages
--each received message is compared to previously sent and received messages.  If it matches either, then it is discarded
3. Once developed fully, publish the stack here with hardware and build instructions

# References
# Hardware
https://www.amazon.com/XBee-PRO-DigiMesh-Point-Multipoint-America/dp/B01HSBYZ0E/ref=sr_1_3?ie=UTF8&qid=1487832333&sr=8-3&keywords=xbee+sx
https://www.digi.com/products/xbee-rf-solutions/embedded-rf-modules-modems/xbee-sx

# Software functionality:
1. Start system
-startup sequence will initialize Python script, which will carry out below functionality
-inittab or equivalent of that
2. Provision 2 24-bit buffers in memory (SOC and EOC)
-may increase buffer size if entropy isn't sufficient to distinguish from random radio signals
-2^24 is ~16 million, so if my thinking is correct, there is only a one in 16 million chance that random radio signals will match this code.  This assumes that the 900MHz spectrum is being used, and signals will match the amplitude we are looking for
3. Enter into "listening mode", which is a do..until type of loop (while 1=1)
-check 24-bit buffer to see if it matches a sequence that will initiate a "reading" mode
-basically 2 things - read one bit into buffer, then check buffer against a code. repeat until it matches
4. When buffer contents = 000000000010110000000000 *NUL/SYN/NUL", provision 3rd buffer and read bits in until second buffer equals EOC sequence
-once the buffer matches this, it will then listen for an end of communication sequence
-again just do 2 things - read one bit into 2 new buffers(EOC and output), and check EOC buffer against a code
5. When buffer contents = 000000000001011100000000 "NUL/ETB/NUL", stop reading and start writing
-split buffer into bytes and translate to ASCII representation.  Store this in 3rd buffer (output)
-display the output buffer to some kind of video/output device (whatever is on the video port, bluetooth or whatever else you set up)
6. Set system back into "listening mode"
