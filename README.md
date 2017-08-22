# MiniCore
An Arduino core for the ATmega8, ATmega48, ATmega88, ATmega168 and ATmega328, all running a [modified version of Optiboot](#write-to-own-flash). This core requires at least Arduino IDE v1.6.2, where v1.6.5+ is recommended. <br/>
<b>This core gives you two extra IO pins if you're using the internal oscillator!</b> PB6 and PB7 is mapped to [Arduino pin 20 and 21](#pinout).<br/>
If you're into "pure" AVR programming, I'm happy to tell you that all relevant keywords are being highlighted by the IDE through a separate keywords file. Make sure to test the [example files](https://github.com/MCUdude/MiniCore/tree/master/avr/libraries/AVR_examples/examples) (File > Examples > AVR C code examples). Try writing a register name, <i>DDRB</i> for instance, and see for yourself!


# Table of contents
* [Supported microcontrollers](#supported-microcontrollers)
* [Supported clock frequencies](#supported-clock-frequencies)
* [BOD option](#bod-option)
* [Link time optimization / LTO](#link-time-optimization--lto)
* [Programmers](#programmers)
* [Why add Arduino support for these microcontrollers?](#why-add-arduino-support-for-these-microcontrollers)
* [Write to own flash](#write-to-own-flash)
* **[How to install](#how-to-install)**
	- [Boards Manager Installation](#boards-manager-installation)
	- [Manual Installation](#manual-installation)
* **[Getting started with MiniCore](#getting-started-with-minicore)**
* **[Pinout](#pinout)**
* **[Minimal setup](#minimal-setup)**


## Supported microcontrollers:
* ATmega8<b>*</b>
* ATmega48<b>*</b>
* ATmega88<b>*</b>
* ATmega168<b>*</b>
* ATmega328<b>*</b>

<b>*</b> All variants (A, P, PA) except PB

Can't decide what microcontroller to choose? Have a look at the specification table below:

|              | ATmega328 | ATmega168 | ATmega88 | ATmega48 | ATmega8 |
|--------------|-----------|-----------|----------|----------|---------|
| **Flash**    | 32kB      | 16kB      | 8kB      | 4kB      | 8kB     |
| **RAM**      | 2kB       | 1kB       | 1kB      | 512B     | 1kB     |
| **EEPROM**   | 1kB       | 512B      | 512B     | 256B     | 512B    |
| **PWM pins** | 6         | 6         | 6        | 6        | 3       |

## Why add Arduino support for these microcontrollers?
* They are all Arduino UNO compatible (drop-in replacement)
* They're extremely popular and used in almost every Arduino project out there
* They're cheap (some can be bought for less than a dollar at AliExpress and Ebay)
* They come in both DIP and TQFP packages
* You can now choose the suited microcontroller for your project. No need to go for overkill!


## Supported clock frequencies
* 16 MHz external oscillator (default)
* 20 MHz external oscillator
* 18.432 Mhz external oscillator <b>*</b>
* 12 MHz external oscillator
* 8 MHz external oscillator
* 8 MHz internal oscillator <b>**</b>
* 1 MHz internal oscillator

Select your microcontroller in the boards menu, then select the clock frequency. You'll have to hit "Burn bootloader" in order to set the correct fuses and upload the correct bootloader. <br/>
Make sure you connect an ISP programmer, and select the correct one in the "Programmers" menu. For time critical operations an external oscillator is recommended. 
</br></br>

<b>*</b> When using the 18.432 MHz option (or any frequency by which 64 cannot be divided evenly), micros() is 4-5 times slower (~110 clocks). It reports the time at the point when it was called, not the end.
This clock frequency is not recommended if your application relies on accurate timing, but is [superb for UART communication](http://wormfood.net/avrbaudcalc.php?bitrate=300%2C600%2C1200%2C2400%2C4800%2C9600%2C14.4k%2C19.2k%2C28.8k%2C38.4k%2C57.6k%2C76.8k%2C115.2k%2C230.4k%2C250k%2C.5m%2C1m&clock=18.432&databits=8). 
Millis() is not affected, only micros() and delay(). Micros() executes equally fast at all clock speeds, but returns wrong values with anything that 64 doesn't divide evenly by.
<br/>

<b>**</b> There might be some issues related to the internal oscillator. It's factory calibrated, but may be a little "off" depending on the calibration, ambient temperature and operating voltage. If uploading failes while using the 8 MHz internal oscillator you have three options:
* Edit the baudrate line in the [boards.txt](https://github.com/MCUdude/MiniCore/blob/3ba977a7c6f948beff5a928d7f11a627282779e2/avr/boards.txt#L83) file, and choose either 115200, 57600, 38400 or 19200 baud.
* Upload the code using a programmer (USBasp, USBtinyISP etc.) or skip the bootloader by holding down the shift key while clicking the "Upload" button
* Use the 1 MHz option instead 


## BOD option
Brown out detection, or BOD for short lets the microcontroller sense the input voltage and shut down if the voltage goes below the brown out setting. To change the BOD settings you'll have to connect an ISP programmer and hit "Burn bootloader". Below is a table that shows the available BOD options:
<br/>

| ATmega328 | Atmega168 | ATmega88 | ATmega48 | ATmega8  |
|-----------|-----------|----------|----------|----------|
| 4.3v      | 4.3v      | 4.3v     | 4.3v     | 4.0v     |
| 2.7v      | 2.7v      | 2.7v     | 2.7v     | 2.7v     |
| 1.8v      | 1.8v      | 1.8v     | 1.8v     | -        |
| Disabled  | Disabled  | Disabled | Disabled | Disabled |


## Link time optimization / LTO
After Arduino IDE 1.6.11 where released, There have been support for link time optimization or LTO for short. The LTO optimizes the code at link time, making the code (often) significantly smaller without making it "slower". In Arduino IDE 1.6.11 and newer LTO is enabled by default. I've chosen to disable this by default to make sure the core keep its backwards compatibility. Enabling LTO in IDE 1.6.10 and older will return an error. 
I encourage you to try the new LTO option and see how much smaller your code gets! Note that you don't need to hit "Burn Bootloader" in order to enable LTO. Simply enable it in the "Tools" menu, and your code is ready for compilation. If you want to read more about LTO and GCC flags in general, head over to the [GNU GCC website](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)!


## Programmers
Mini does not adds its own copies of all the standard programmers to the "Programmer" menu. Just select one of the stock programmers in the "Programmers" menu, and you're ready to "Burn Bootloader" or "Upload Using Programmer".

Select your microcontroller in the boards menu, then select the clock frequency. You'll have to hit "Burn bootloader" in order to set the correct fuses and upload the correct bootloader. <br/>
Make sure you connect an ISP programmer, and select the correct one in the "Programmers" menu. For time critical operations an external oscillator is recommended.
 
 
## Write to own flash
A while ago [@majekw](https://github.com/majekw) announced that he'd [successfully modified the Optiboot bootloader](http://forum.arduino.cc/index.php?topic=332191.0) to let the running program permanently store content in the flash memory.
The flash memory is much faster than the EEPROM, and can handle about 10 000 write cycles before it's worn out. <br/>
To enable this feature your original bootloader needs to be replaced by the new one. Simply hit "Burn Bootloader", and it's done! <br/>
Please check out the [Optiboot flasher example](https://github.com/MCUdude/MiniCore/tree/master/avr/libraries/Optiboot_flasher/examples/SerialReadWrite) for more info about how this feature works, and how you can try it on your MiniCore compatible microcontroller.


## How to install
#### Boards Manager Installation
This installation method requires Arduino IDE version 1.6.4 or greater.
* Open the Arduino IDE.
* Open the **File > Preferences** menu item.
* Enter the following URL in **Additional Boards Manager URLs**:

    ```
    https://mcudude.github.io/MiniCore/package_MCUdude_MiniCore_index.json
    ``` 

* Open the **Tools > Board > Boards Manager...** menu item.
* Wait for the platform indexes to finish downloading.
* Scroll down until you see the **MiniCore** entry and click on it.
  * **Note**: If you are using Arduino IDE 1.6.6 then you may need to close **Boards Manager** and then reopen it before the **MiniCore** entry will appear.
* Click **Install**.
* After installation is complete close the **Boards Manager** window.


#### Manual Installation
Click on the "Download ZIP" button in the upper right corner. Exctract the ZIP file, and move the extracted folder to the location "**~/Documents/Arduino/hardware**". Create the "hardware" folder if it doesn't exist.
Open Arduino IDE, and a new category in the boards menu called "MiniCore" will show up.


## Getting started with MiniCore
Ok, so you're downloaded and installed MiniCore, but do I get the wheels spinning? Here's a quick start guide:
* Hook up your microcontroller as shown in the [pinout diagram](#pinout), or simply just plut it into an Arduino UNO board.
	- (If you're not planning to use the bootloader (uploading code using a USB to serial adapter), the FTDI header and the 100 nF capacitor on the reset pin can be omitted.) 
* Open the **Tools > Board** menu item, and select a MiniCore compatible microcontroller.
* If the *BOD option* is presented, you can select at what voltage the microcontroller will shut down at. Read more about BOD [here](#bod-option).
* Select your prefered clock frequency. **16 MHz** is standard on most Arduino boards, including the Arduino UNO.
* Select what kind of programmer you're using under the **Programmers** menu.
* If the *Variants* option is presented, you'll have to specify what version of the microcontroller you're using. E.g the ATmega328 and the ATmega328P got different device signatures, so selecting the wrong one will result in an error.
* Hit **Burn Bootloader**. If an LED is connected to pin PB5 (Arduino pin 13), it should flash twice every second.
* Now that the correct fuse settings is sat and the bootloader burnt, you can upload your code in two ways:
	- Disconnect your programmer tool, and connect a USB to serial adapter to the microcontroller, like shown in the [minimal setup circuit](#minimal-setup). Then select the correct serial port under the **Tools** menu, and click the **Upload** button. If you're getting some kind of timeout error, it means your RX and TX pins are swapped, or your auto reset circuity isn't working properly (the 100 nF capacitor on the reset line).
	- Keep your programmer connected, and hold down the `shift` button while clicking **Upload**. This will erase the bootloader and upload your code using the programmer tool.

Your code should now be running on your microcontroller! If you experience any issues related to bootloader burning or serial uploading, please use *[this forum post](https://forum.arduino.cc/index.php?topic=412070.0)* or create an issue on Github.


## Pinout
This core uses the standard Arduino UNO pinout and will not break compatibility of any existing code or libraries. What's different about this pinout compared to the original one is that this got three aditinal IO pins available. You can use digital pin 20 and 21 (PB6 and PB7) as regular IO pins if you're ussing the internal oscillator instead of an external crystal. If you're willing to disable the reset pin (can be enabled using [high voltage parallel programming](http://www.atmel.com/webdoc/stk500/stk500.highVoltageProgramming.html)) it can be used as a regular IO pin, and is assigned to digital pin 22 (PC6). 
<b>Click to enlarge:</b> 
</br> </br>
<img src="http://i.imgur.com/bOJPN5h.jpg" width="850">
</br> </br>
<img src="http://i.imgur.com/sdCDEbY.jpg" width="700">


  
## Minimal setup
Here is a simple schematic showing a minimal setup using an external crystal. Skip the crystal and the two 22pF capacitors if you're using the internal oscillator. If you don't want to mess with breadboards, components and wiring; simply use your Arduino UNO!<br/>
<img src="http://i.imgur.com/d7Xhtht.png" width="750">
