# Exploring UART with a rasberry pi

## Useful Programs:
* Baud and Baud Rate
    * number of communcation pulses(changes in signal) per second. Baud rate is the speed of these communcations.
* minicom
    * user friendly serial communication program.
* screen 
    * Easy cli app to interface with serial connections allows you to set baud rate. This is what we will use. 
## Commands:
`lsblk`
`sudo dmesg | grep tty` - Find serial devices.
`sudo setserial -g` - view tty devices and path to them (makes using screen easier.)
`dd` - disk utility tool. (https://www.geeksforgeeks.org/dd-command-linux/)
`# dd if=hdadisk.img of=/dev/hdb` - Use to write img to drive.