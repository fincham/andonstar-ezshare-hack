# andonstar-ezshare-hack

I wanted to use an LZeal brand "ez Share" WiFi SD card (https://www.aliexpress.com/item/32731178253.html) with my Andonstar ADSM301 microscope (http://www.andonstar.com/e_products/Digital-microscope-ADSM301-5.html) to quickly transfer microscope photos to my PC without needing to eject the SD card, mount it, copy the photos and so on.

The ez Share card can only read photos located in a directory called `DCIM` in the root of the card's FAT32 partition. Unfortunately, the Andonstar microscope can only write photos in to a directory called `CARDV`.

To solve this problem I created a special FAT32 filesystem image to place on the SD card of the device which includes both `DCIM` and `CARDV` directories, however both of these entries in the FAT directory table point to the same start cluster. This means that when the microscope writes files in to the `CARDV` directory, the WiFi SD card firmware can read them immediately from the `DCIM` directory. Only a single copy of the file is stored.

Once this image is written to the SD card it allows the microscope to operate normally and for the stored images to be readable over WiFi. If the "format" option on the microscope is used however, the special filesystem image is lost and will need to be re-written.

Tools like `chkdsk` and `fsck` will consider this filesystem to be damaged and try and "fix" it, which will also break the link between the two directories. If the filesystem is inadvertently "repaired" with one of these tools it'll need to be manually fixed with a hex editor, or simply re-imaged.

## Writing the filesystem to the SD card from Linux

Once you have connected the SD card to your Linux computer and identified which block device the kernel has chosen for it (most likely this will be something like `/dev/sdx` if you're using a normal USB card reader) you can easily write the filesystem image to the card from a root shell:

    # xzcat sd-card.img.xz > /dev/sdx

Make sure you write it to the whole device and not a partition, as the image contains a partition table. The extracted image will fit on a 16GB SD card though you may be able to resize it if you write it to a larger card.

Writing this image will erase any existing files on the SD card, though should preserve the WiFi settings.

## Resetting the LZeal ez Share card to default settings

If you need to reset the card to its default settings (SSID of `ez Share` and password of `88888888`) then simply delete the empty `ezshare.cfg` file from the card and power cycle it.

The card hands out DHCP leases in the `192.168.4.0/24` network, and configures itself as the default route and DNS server at `192.168.4.1`. It will then respond to HTTP requests to `http://ezshare.card/` which resolves to `192.168.4.1`.

The default administrator password is `admin`.

## Connecting to the ez Share card with wpa_supplicant

If the card is in its default configuration then you can use `wpa_supplicant` to connect to it using this `wpa_supplicant.conf` file:

    network={
      ssid="ez Share"
      #psk="88888888"
      psk=80cc6006700a6b78ed31ee85b21130c4eba8decd5bc822f0ed30a02bdc3b1977
    }

I used a USB `Ralink Technology, Corp. RT5370 Wireless Adapter` successfully. This adapter was previously sold as the "WiPi" adapter for use with the Raspberry Pi single board computer.
