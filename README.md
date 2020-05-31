# STM32_HID_Bootloader
Driverless USB HID bootloader for STM32F10X devices (use CMake and Clion IDE)<br>
PID = 0x1209<br>
VID = 0xBEBA<br>

Size bootloder 1980 byte, only the first FLASH page 2k is used, please modiff VECT_TAB_OFFSET your app = 0x00000800U<br>

Enter in "boot mode" two metods<br>
  1. use BOOT 1 PIN <br>
  2. use magic_word 0x424C in BKP RTC registers (BKP->DR4) <br>
 
