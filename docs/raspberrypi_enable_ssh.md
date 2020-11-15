# Enable SSH on Raspberry Pi

For headless setup, SSH can be enabled by placing a file named `ssh`, without any extension, onto the boot partition of the SD card from another computer. When the Pi boots, it looks for the `ssh` file. If it is found, SSH is enabled and the file is deleted. The content of the file does not matter, it could contain text, or nothing at all.  

If you have loaded Raspberry Pi OS onto a blank SD card, you will have two partitions. The first one, which is the smaller one, is the boot partition. Place the file into this one.