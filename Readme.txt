My implementation of the USBasp programmer on Windows 10 on a perfboard. 
Changes from the original include: 
  1. using an ATMega328P instead of the original ATMega44/88. 
  2. Changing the input circuit  from using zener diodes to regular diodes 
  3. Omitting the jumper bridges for "Self-programming" and "Slow CLK" 
  4. Power to target is always on. 
Software is using the libusbk driver (signed by Protostack Pty Ltd) and installed using Zadig. 

See project description file for full details. 
