# jelly-star-shenanigans

This is a brief write up of my shenanigans with the Unihertz Jelly Star. 

This is incomplete as it's an ongoing project and is not meant to be used as a documentation tool or tutorial. Informal, yet Informative.

If you're working on something similar it may be helpful however as I'd run into a couple of weird quirks. 

## Goal.

I'm looking to use a dumbphone 

We start here.

https://www.reddit.com/r/unihertz/comments/16sviga/unihertz_jelly_star_running_great_with_lineageos/

"The only thing I didn't test was the weird gimmicky lights on the back. You probably need the Unihertz app specifically for controlling them (can probably be extracted from the ROM), but I don't care enough to figure that out."

well... I care.

List APKs 
```
> .\adb.exe shell pm list packages | select-string -Pattern "led" -simplematch

package:com.agui.providers.ledbelt
package:com.agui.ledsettings
package:com.agui.ledbeltwidget
package:com.agui.ledbeltdebug


```

gives me the following paths.

```
> .\adb.exe shell pm path <APK name>
```

```
- package:com.agui.providers.ledbelt
	- /system/system_ext/priv-app/AguiLedBeltContactsProvider/AguiLedBeltContactsProvider.apk

- package:com.agui.ledsettings
	- /system/system_ext/app/AguiLEDSettings/AguiLEDSettings.apk

- package:com.agui.ledbeltwidget
	- /system/app/AguiLedBeltWidget/AguiLedBeltWidget.apk

- package:com.agui.ledbeltdebug
	- /system/system_ext/priv-app/AguiLedBeltDebug/AguiLedBeltDebug.apk
```

becomes

```
> .\adb.exe pull /system/system_ext/priv-app/AguiLedBeltContactsProvider/AguiLedBeltContactsProvider.apk .\LEDapps\
> etc ...
```

well now fastboot doesn't fucking work. 

https://xdaforums.com/t/solved-cant-pass-fastboot-commands-to-bootloader.4141539/

```
**SOLUTION:** It was a driver issue after all. Windows 10 would not detect any drivers, nor let me install any drivers manually from files, however, it did let me select built-in drivers from my computer. YMMV, but If you are running into this issue try this:  
1) Put your phone into bootloader mode.  
2) Open device manager.  
3) Right-click the device called "Android" with the yellow question mark, and select "update driver."  
4) Click the "Browse my computer" option.  
5) Click "Let me pick."  
Note: There are several categories here, I suggest you try them all: portable devices, android devices, and in my case, Samsung devices. In one of the categories, you will find Google listed as a manufacturer.  
6) In one of the categories, you will find Google listed as a manufacturer (in my case it was under the "Samsung Devices" category).  
7) Select Google and then select the newest fastboot driver listed (in my case it was dated 2016), and install that driver.  
8) Go back to PowerShell and run "fastboot devices." With a little luck, it will work!
```

so we flash with 

```
.\adb.exe reboot bootloader
.\fastboot.exe devices
.\fastboot flashing unlock
.\fastboot reboot fastboot
.\fastboot.exe delete-logical-partition product
.\fastboot.exe erase system_a
.\fastboot.exe flash system_a ..\..\lineage-20.0-20241118-UNOFFICIAL-arm64_bvN.img
.\fastboot.exe --set-active=a
.\fastboot.exe erase userdata
```

ok fuck... here we go.

`.\fastboot reboot`

...

WORKS 
need to test calling 

android auto doesnt work?
https://xdaforums.com/t/gsi-fix-communication-error-22-on-android-auto.4456645/

lets root this boy.
https://mcwain.net/how-to/jelly_root/
https://topjohnwu.github.io/Magisk/install.html

get the firmware from here (god bless you unihertz just having it right here.)
https://drive.google.com/drive/folders/1feYyrSFOPcpJUOp4BdQBHjPFZVXQ9Ubx

```
> .\adb.exe push ..\..\2024043017_g58v89c2k_dfl_tee\2024043017_g58v89c2k_dfl_tee\boot.img /storage/self/primary/Download/
```

magisk happens

```
> .\adb.exe pull /storage/self/primary/Download/magisk_patched-28100_a2kzj.img .
```

do i need to do vbmeta? 
both of those docs say i do...
https://xdaforums.com/t/closed-what-is-vbmeta-img.3931588/

![ugh]()
UUUUUUUUUUUUUUUUUUUUUUUUUGH NEVER say this.
THIS THREAD could very well be what is put on google, and now you've been immortalized as an asshole....

lets just do it. sounds like the vbmeta.img pairs with the boot.img to provide the trusted keys. So i probably want the one that came with this version of the boot. lets try it. 

```
> .\fastboot.exe flash vbmeta ..\..\2024043017_g58v89c2k_dfl_tee\2024043017_g58v89c2k_dfl_tee\vbmeta.img
Warning: skip copying vbmeta image avb footer (vbmeta partition size: 0, vbmeta image size: 8192).
Sending 'vbmeta' (8 KB)                            OKAY [  0.003s]
Writing 'vbmeta'                                   FAILED (remote: 'No such file or directory')
fastboot: error: Command failed
```

....

ok bet... Lets just try it without. 

```
> .\fastboot.exe flash boot ..\..\2024043017_g58v89c2k_dfl_tee\patched\magisk_patched-28100_a2kzj.img
```

ok it worked... lets see if i get stuck in a boot loop... 

```
> .\fastboot.exe reboot
```

WOO it boots. i didnt bork all my progress.. looks like my data is still there. too. lets check magisk. 

alright, lets install the systemizer module- 

lol nope. thats greyed out. is it a version thing??

https://www.reddit.com/r/androidapps/comments/uirlla/comment/i7emuuj/
```
As you probably know, you need root for this. If you have root, the most simple method is to simply copy the .apk under `/system/priv-app/NameOfApp/NameOfApp.apk`.

Please note that often the system partition has no leftover space for additional apps and you may not be able to do this. For this and other reasons the more elegant solution is to install Magisk in your boot partition. Magisk is pretty easy to install if you have root, just [download the Magisk Manager app](https://github.com/topjohnwu/Magisk) and it will do the rest for you.

With Magisk you put the .apk in a Magisk module (a zip [with a certain structure](https://github.com/prabi/magisk-module-template)). You just need the META-INF folder and module.prop for something like this, and add your .apk with the above path in there too. Remember to edit module.prop.

Magisk will pretend-copy the .apk to /system and it will look like it was really copied there and work as a system app, but it will actually leave /system untouched.
```

So it looks like I have 3 options 
1. Resize the system partition
2. Figure out what updates are necessary to the systemizer module to make it compatible, with current magisk
	- [like so](https://xdaforums.com/t/module-template-custom-app-systemizer.4499551/)
3. write my own magisk module to install android auto as a system app.

so someone did already 

oh...

so someone already did 3 for me.
https://xdaforums.com/t/module-template-custom-app-systemizer.4499551/ 
bet. 

So i install it... it works? 
I go to my car and test it. it asks me to update android auto from the stub app--- it .... doesnt work. 

ok maybe update from the aurora store? 
signature mismatch??

is there a way around this? no? 

can i install from APKmirror? https://www.apkmirror.com/apk/google-inc/android-auto/android-auto-13-8-6508-release/android-auto-13-8-650804-release-android-apk-download/download/?key=93100061bcde547ff66ed118da9a2154e8798403&forcebaseapk=true

doesnt work on the phone... can i install it with ADB? 

```
.\adb.exe install com.google.android.projection.gearhead_13.8.650804-release-138650804_minAPI26(arm64-v8a)(nodpi)_apkmirror.com.apk
```

success??

ok back to the car. 

ok yes clear approved cars from the app,

ok delete phone from head unit

pair again....

it connects...

IT CONNECTS???

THE SCREEN IN THE CAR IS JUST BLACK???

OH COME ON

# BITE ME

.....

so the packaged android auto app comes with  `Android Auto  XLauncher Unlocked (AAXLU)`...

It has a troubleshooting tool....
we try this tomorrow... 

....
good god.... 

is it signature spoofing???

I proceed to mess around with these things some:
- https://xdaforums.com/t/index-how-to-get-signature-spoofing-support.3557047/
- https://github.com/LSPosed/LSPosed/releases
- https://api.xposed.info/reference/packages.html
- https://github.com/thermatk/FakeGApps

I'm in too deep. no reasonable person could get this working. only us freaks....

do i need to redefine success again? 
I'm 10+ anonymous maintainers into getting rid of google. I don't know if I can even trust all these people- I'm losing track of all the disparate repositories- 
at what point is this just as bad if not worse than my solution of google. 

# Flash to Stock :c 

get stock image from here https://drive.google.com/drive/folders/1iJZV76NIYV_P_T4cxt8fH13kG5zvvrDt

https://www.getdroidtips.com/stock-rom-unihertz-jelly-2-firmware/


![warning]()

oh thanks there, ill look out for that.

The application is either in mojibake or mandarin and does not seem to 

maybe this isn't the move. 

# LineageOS W/ GApps? 

So its put the sim card in and this again...
```
.\fastboot.exe erase system_a
.\fastboot.exe flash system_a ..\..\lineage-20.0-20250221-UNOFFICIAL-arm64_bgN-signed.img
.\fastboot.exe --set-active=a
.\fastboot.exe erase userdata

.\fastboot reboot
```

I set up the bare minimum. Do NOT give it a google account.

I go out to the car and try again. Android Auto IS installed so we can skip that. 

I get in my car and clear the old connection from the head unit, rename the phone, and try again.

The menu progresses farther than it had the first time, it doesn't seem to be crashing...

the screen shows....

It's making me install google maps, google, text to speech services. ugh ok we can tweak that later,,,

I install these through Aurora store to avoid entering a google account...

We connect...
*it works....*

I cannot believe it. It fucking works... 

Waze shows instead of google maps like I configured.

I enter settings and start disabling what i can.

I disable the google app and deny it network connectivity,
I gut the permissions on google play services (disabling crashes android auto) and deny it network connectivity.
I gut the permissions on google maps, and again- deny it network connectivity. 

It seems to work...

I don't feel victory, it's a compromise. 

I don't feel success, but relief. 

Inside I report my success to my friends. They're amused and proud, and have funny things to say. 

I export my contacts from my old phone and import them to the new one, `./adb.exe pull/push` . 4 straight days of work coming to a close. I note to my friend foxyy that now I have to organize my documentation in narrative- fill out the CAT phone's documentation, and fill out the narrative portion of my story. Organize, Format, Publish. I'm not done yet, but she says she's proud of me none the less. It's comfort. 

---
# Finishing up

**Map data
- https://takeout.google.com

**Lights
```
> .\adb.exe install ..\..\LEDapps\AguiLedBeltContactsProvider.apk
Performing Streamed Install
adb: failed to install ..\..\LEDapps\AguiLedBeltContactsProvider.apk: Failure [INSTALL_FAILED_SHARED_USER_INCOMPATIBLE: Reconciliation failed...: Reconcile failed: Package com.agui.providers.ledbelt has no signatures that match those in shared user android.uid.system; ignoring!]
```

so there is a signature issue here. That makes sense. These came from the vendor stock image, 

This is KIND of my issue 
https://xdaforums.com/t/solved-install-failed-shared-user-incompatible.1219029/

what was tried: 
- Used LSPosed's CorePatch to bypass signature validation to get the apks installed
- setenforce 0 then to put SELinux in permissive mode to get past the tombstones it was throwing on launch
- review logcat for debug. 

![lunatic]()

I emailed peter about it and received no response. 

well what about the originals. I can use core to install then.
ugh. ok. they fail. but why. 

```
03-06 21:58:56.935 11082 11082 D AndroidRuntime: Shutting down VM
03-06 21:58:56.938 11082 11082 E AndroidRuntime: FATAL EXCEPTION: main
03-06 21:58:56.938 11082 11082 E AndroidRuntime: Process: com.agui.ledbeltwidget, PID: 11082
03-06 21:58:56.938 11082 11082 E AndroidRuntime: java.lang.NoClassDefFoundError: Failed resolution of: Lcom/agui/server/ledbelt/AguiLedBeltManager;
```

and taking a look at the APK's in JadX does include a line 

```
import com.agui.server.ledbelt.AguiLedBeltManager;
```

i missed something.

ok so. 

I need the system image again. https://www.reddit.com/r/unihertz/comments/1j3r99a/jelly_star_stock_systemimg_not_in_google_drive/ i made a post

I obtain the system image and upacked it. I'm writing this paragraph well after i did so I'm missing some things and will verify all this later- BUT 

these are the tools I used to do so IIRC.

- https://xdaforums.com/t/extract-system-img-vendor-img-and-product-img-from-super-img.4184901/
- https://github.com/VeloshGSIs/Super_Guide
- https://newandroidbook.com/tools/imjtool.html

However unpacking the binary does not provide any information on a package called `AugiLedBeltManager`

My options going forward are find a way to sign these with the keys from the lineage GSI I used, OR if I really hate myself, maintain my own image. I think actually that I'll be doing the later. stay tuned dorks. 