# Commands:

* Install flashrom:
 * `sudo apt install flashrom`
* Find serial device:
    * `sudo dmesg`
* Flashrom read:
    * `sudo flashrom -p ch341a_spi -r firmwareout.bin`


### Reversing:

* Extract the file system with binwalk.
    * `binwalk -e -M firmwaretest.bin`   
* CD into the new dir:
    * `cd _firmwaretest.bin.extracted`
* # Install standard extraction utilities
$ sudo apt-get install mtd-utils gzip bzip2 tar arj lhasa p7zip p7zip-full cabextract cramfsprogs cramfsswap squashfs-tools sleuthkit default-jdk lzop srecord
