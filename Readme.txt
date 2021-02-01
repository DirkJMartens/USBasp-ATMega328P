My implementation of the USBasp project on Windows 10 using an ATMEGA328p. 

USBasp was started by www.fischl.de/usbasp and files are readily available for ATMEGA8, 48 and 88 but not for 328 (which I had lying around). 
This project also uses V-USB files from http://www.obdev.at/vusb and Zadig.exe from https://zadig.akeo.ie 


HARDWARE
========
The hardware and schematic was based on two projects and combined the original USBasp hardware with the input part of the V-USB examples. 
The original two schematics can be found here: 
	1. Go to www.fischl.de/usbasp and download "usbasp.2011-05-28.tar.gz". Use 7-Zip to extract "usbasp.2011-05-28.tar" file from it. 
	   Use 7-Zip to extract all files from "usbasp.2011-05-28.tar" into "usbasp.2011-05-28 folder". 
	   The "circuit" folder contains the info I needed for the hardware part and the "firmware" contains the info we will need for the software part. 
	2. The second circuit can be found at https://www.obdev.at/products/vusb/index.html 

1. For the hardware, I started with the "USBasp_schematic.pdf" as included in the original project on www.fischl.de/usbasp but changed the input section. 
   The original input section uses 2 zener-didoes of 3.6V to clamp the D+ and D- voltages so they cannot exceed 3.6V. 
   The V-USB circuit on the other hand creates 3.6V bydropping 1.2 to 1.4 V across two 1N4148 diodes in series from the 5V input voltage. 
   I used an USB-B input connector: 
	- On pin 1 (5V/red ), use two series 1N4148 diodes to drop input voltage from 5 volts to ~3.6 volts (ideally 3.3v) 
	- On pin 2 (D-/white), use a 1k5 pull-up resistor to 3.6V and a 68 ohm series resistor to PB0 pin (328 - pin 14)
	- On pin 3 (D+/green), use a 1M pull-down resistor to GND and a 68 ohn series resistor to INT0 pin (328 - pin 4) AND to PB1 pin ( 328 - pin 15)
	- On pin 4 (GND/black), use a 4u7 capacitor between 3.6V and GND 
   Other changes I made: using a 6-pin instead of a 10-pin ICSP header and omitting the "Slow Clock" and "Self-programming" jumpers and hard-wiring the "VTarget" jumper. 
   For the main circuitry around the ATMega328P, we have: 
     - 12 MHz crystal (must be 12 MHz) between XTAL1 and XTAL2 pins (pins 9 and 10) with two 22p caps to GND 
     - 10K pull-up on RESET pin (pin 1) to 3.6V 
     - Connections to GND for digital and analog GND (pins 8 and 22) 
     - Connections to 5V (not 3.6 V) for digital and analog Vcc (pins 7 and 20) 
     - By-pass cap between Vcc and GND as close to pins 7 and 8 as possible 
     - On pin 23 RED led with 1K current limiting resistor to 5V to indicate Power On
     - On pin 24 GREEN led with 1K current limiting resistor to 5V to indicate Programming is in progress 
     - Jumper bridge to GND on pin 25 for "Slow SCK" in case program target has clock speed < 1.5 MHz (which I have omitted) 
     - Jumper bridge from RESET (pin 1) to pin 16 for "Self programming" of the 328 (e.g. to update the firmware) (which I have omitted) 
     - Output signals to the target are: 
	- SCK: 328 - pin 19
	- MISO: 328 - pin 18
	- MOSI: 328 - pin 17 
	- SS: 328 - pin 16 
	- GND: from circuit GND 
	- Vcc: via a jumper bridge from 5V USB input. Install jumper bridge to power target from programmer. (which I have made permanent). 

	Files included here are the KiCAD schematic and PCB layout. 

SOFTWARE
========
Once the above circuit has been made (I made it on a piece of perfboard), we can focus on the software side. 
1. From the fischl-download, go to the "firmware" folder: 
	- Edit the makefile by commenting out the 3 existing "TARGET =" lines and by adding the following 3 lines: 
		TARGET=atmega328P
		HFUSE=0xDE
		LFUSE=0xFF
	  (The edited makefile is included in this github) 
	- From a Command Prompt, run the "make" command to show the help file and available options. 
	- Use the "make clean" command to remove the temporary files such as the .o, .lst and .bin files etc. 
	- Finally, use the "make main.hex" command to compile a new hex-file (which with the edit above is now suited for 328p). 
	- This will generate a bunch of files, of which "main.hex" is the one we need for uploading into the 328p. 
	  (This main.hex file is also included in the github) 
2. To upload the "main.hex" file in an ATMEGA328P: 
	- In the File - Preferences, enable "verbose" for Upload. 
	- Use the Arduino IDE to upload the "ArduinoISP" sketch from the "File - Examples" into the Arduino UNO. 
	- This means your Arduino UNO is now configured as a programmer. 
	- Now install the 328p on a breadboard or on a programming shield so we can program it. 
	- Programming happens using the MISO/MOSI/SCK/SS/GND/VCC pins, so connect the 6 pin ICSP header opposite the USB input on the Uno board. 
	- Use 6 jumper wires between your Uno to your ATMega328p on the breadboard. 
	- To test if all is OK, load the Blink sketch in the Arduino IDE and change "Board" to "Atmega328 on a breadboard". 
	- Upload the Blink sketch using the "Upload using programmer" command (do NOT use regular "Upload"). 
	- If you use "Upload" instead of "Upload using programmer", the Blink sketch will uploaded into the Uno (overwriting the ArduinoISP sketch). 
	- Connect an LED to the 328P and verify your Blink sketch has been uploaded successfully. 
  	- If this was successful, you can confirms you are able to load a .hex file into the ATMEGA328p on your breadboard. 
	- Go back to the Arduino IDE, scroll up to the line which invokes the "avrdude" program which uploads the blink.hex file. 
	- Copy-and-paste it to a Notepad file so you can view and inspect it and modify its arguments. 
	- Now edit the Notepad file so it matches your requirements, especially "blink.hex" into "main.hex" 
	- Other arguments (clock speed, port settings, file path names ...) may need to be changed as well. 
	- Copy-and-paste the updated avrdude command from Notepad and use it in a Command Prompt to upload the main.hex file into the 328p using the ArduinoISP as a programmer. 
	- If all went well, we should now have successfully uploaded the USBasp firmware into the ATMEGA328p on the breadboard. 
  	- We can now insert the programmed 328P with the main.hex file into our programmer circuit from step 1. 
3. Plugging the programmer into a USB port of a computer should now report a new USB device 
	- Right-click "My Computer" - "Manage" - "Device Manager" to list the devices. 
	- USBasp should be listed now as a new device. And unless you already installed a driver for it, it will show with a yellow Exclamation logo. 
	- We now need to load the appropriate USB driver for it. 
	- Download "zadig-2.5.exe" and run it. 
	- In the list of USB devices, USBasp should be listed (if not go to "Options - List all devices" menu). 
	- It should show with a VID/PID combination of "16C0 - 05DC". 
	- Select USBasp and from the list of drivers, select "libusbK" and click "Install Driver". 
	- This driver is signed for Windows 10 (by Protostack Pty Ltd). 
	- Once this is completed, the Device Manager should show a "libusbK USB Devices" with the "USBasp" logo without exclamation mark.  

- You should now be able to use avrdude to upload .hex files using the USBasp
	- Connect your target devices using a 6 or 10 pin ICSP ribbon cable 
	- Compile your source-code file into a .hex file 
	- Upload the .hex file with avrdude 
  - Alternatively, you can test it using the Arduino IDE as follows: 
    - Disconnect the USB cable from the Arduino UNO 
    - Connect the USB cable to the USBasp instead 
    - Connect the USBasp output to the 6-pin ICSP port on the Arduino UNO 
    - Load the BLINK sketch in the Arduino IDE
    - Select "Arduino Uno" as Board and "USBasp" as Programmer
    - Use "Upload using Programmer" to upload the BLINK sketch through the USBasp into the Uno using the ICSP port instead of the ATMEGA8U2 USB-to-serial converter 

