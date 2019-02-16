## macOS-edid-modification

macOS sometimes has problems with using the obtained EDID from the display, especially for TVs. Even so the macOS Tools are able to obtain the proper EDID it is not used for the device. Leading to suboptimal resolution and refresh rate Settings.


## How to obtain the EDID of your Displays

#### ioreg full info

This outputs the Display info. Important are the keys **IODisplayPrefsKey** and **IODisplayEDID**. Former is your identifier, used later to override your EDID, the later is the actual EDID data in hex encoding. Example output is [ioreg-full-info.txt](ioreg-full-info.txt).

```
ioreg -l | grep -5 IODisplayEDID
```

#### ioreg short info

The same as above but only outputs the EDID. Example output is [ioreg-short-info.txt](ioreg-short-info.txt).

```
ioreg -l | grep IODisplayEDID
```

## How to convert the hex encoded EDID to binary Data

Copy the hex encoded EDID from **IODisplayEDID** key into its own file. Example [edid-hex-dell.txt](edid-hex-dell.txt) of my DELL monitor and [edid-hex-lg-tv.txt](edid-hex-lg-tv.txt) of my LG OLED TV.

To convert this hex string now we can do the following.

```
cat edid-hex-dell.txt | xxd -r -p
```

Example output [edid-bin-dell.bin](edid-bin-dell.bin) and [edid-bin-lg-tv.bin](edid-bin-lg-tv.bin).

## Read the EDID binary files

Informations from those binary files can be read with [edid-decode](https://git.linuxtv.org/edid-decode.git/tree/).

```
edid-decode did-bin-dell.bin
```

Example output [edid-decode-dell.txt](edid-decode-dell.txt) and [edid-decode-lg-tv.txt](edid-decode-lg-tv.txt).

#### EDID Editor

You can also use an editor to view the info and modify it, like [AW EDID Editor](https://www.analogway.com/americas/products/software-tools/aw-edid-editor/).

## Modifying the binary files

There are various reasons why to modify the EDID file. For example macOS might force you to use a limited range (YCrCb) signal instead of a full range RBG one. If that happens one needs to remove all the mentions of YCrCb 4:4:4 and 4:2:2 support from the EDID. That way macOS is forced to use an RGB full range signal.

As an example I modified my LG TVs EDID, see [edid-bin-lg-tv-mod.bin](edid-bin-lg-tv-mod.bin). Additionally to the above mentioned changes i also had to remove some vendor specific stuff to make it properly working. Keep in mind to leave the Detailed Timing info in.

## How to override your EDID

Navigate to the **/System⁩/⁨Library⁩/⁨Displays⁩/Contents⁩/⁨Resources⁩/⁨Overrides⁩** directory. Within this folder are all the Display specific properties and settings. The name is a bit cryptic but here our IODisplayPrefsKey key comes into play.

Exemplary the following is the value of that key on my System. The crucial part is are the two last hex digits, **1e6d** (DisplayVendorID) and **1** (DisplayProductID). The first part is part of the folder name we need to look into (**DisplayVendorID-1e6d**) and the second part is part of the file we need to modify (**DisplayProductID-1**).

````
/AppleACPIPlatformExpert/PCI0@0/AppleACPIPCI/PEG0@1/IOPP/PEGP@0/IOPP/pci-bridge@0/IOPP/GFX0@0/ATY,AMD,RadeonFramebuffer@3/AMDFramebufferVega10/display0/AppleDisplay-1e6d-1
````

Just open the file with any Text Editor and you should see something like. See the matching DisplayProductName.

````
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>DisplayProductID</key>
	<integer>1</integer>
	<key>DisplayProductName</key>
	<string>LG TV</string>
	<key>DisplayVendorID</key>
	<integer>7789</integer>
</dict>
</plist>
````

To override our EDID now we need to add another key/value entry into our plist file. Specifically the IODisplayEDID key with a data value encoded in base64. It will look like this.

````
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>DisplayProductID</key>
	<integer>1</integer>
	<key>DisplayProductName</key>
	<string>LG TV</string>
	<key>DisplayVendorID</key>
	<integer>7789</integer>
	<key>IODisplayEDID</key>
	<data>BASE64EncodedEDIDHere</data>
</dict>
</plist>
````

## How to convert the binary EDID into a Base64 encoded string

First we need to convert our binary file back to a hex string. Example output [edid-hex-lg-tv-mod.txt](edid-hex-lg-tv-mod.txt).

````
xxd -ps edid-bin-lg-tv-mod.bin
````

With following command you can convert the hex string file to a base64 encoded string, or use an online Tool like [this](http://tomeko.net/online_tools/hex_to_base64.php?lang=en).

````
cat edid-hex-lg-tv-mod.txt | xxd -r -p | base64
````

The resulting string can be copied into our plist file. You can also rename the display by changing the **DisplayProductName** key.

````
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>DisplayProductID</key>
	<integer>1</integer>
	<key>DisplayProductName</key>
	<string>LG OLED 55C7</string>
	<key>DisplayVendorID</key>
	<integer>7789</integer>
	<key>IODisplayEDID</key>
	<data>AP///////wAebQEAAQEBAQEbAQSzoFp4Au6Ro1RMmSYPUFShCAAxQEVAYUBxQIGAAQEBAQEBCOgAMPJwWoCwWIoAQIRjAAAeAjqAGHE4LUBYLEUAQIRjAAAeAAAA/QA6eR6IPAAAAAAAAAAAAAAA/ABMRyBPTEVEIDU1QzcKAWMCAzPBWmFgEB9mZQQTBRQDAhggISIbAV1eX2JjZD9ALwlXBxUHUFcHAT0GwGcEA+MFgAANAeIPM+sBRtAAJgoJdYBbbGYhULBRABswQHA2AECEYwAAHgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAsw==</data>
</dict>
</plist>
````

All that is left is saving the file and restarting your Computer. In my case my TV was locked at 60Hz and limited range, after i manually added the read EDID i was able to set refresh rates between 24 to 120Hz depending on the resolution. It also made the 4k 50Hz RGB 10bit (or 4:4:4 full chroma) mode available. The 50Hz mode is advantageous, since on 60Hz only 4k 10bit 4:2:2 at max is support with HDMI 2.0a–2.0b and 50Hz is a closer multiple of our most common ~24fps video material.

If macOS is not able to read a proper EDID you have to obtain your EDID data with Linux or Windows.