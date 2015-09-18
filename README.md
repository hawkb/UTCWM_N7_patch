Problem: Ubuntu Touch as of 09/17/2015 doesn't install correctly following the official guide on my Nexus 7.  Nexus 7 is a "reference" device.  What's wrong with mine?  

Poking around the internet (xda, #ubuntu-touch, #ubuntu-kernel) I found that newer Nexus 7 "flo" models, made in/after late 2014 have a different revision to their eMMC controller/hardware/something.  Asus posted a kernel change:
```
mmc: add 5.0 emmc support

bug: 17968808 Kernel change for new eMMC v5.0 parts for FLO/DEB

Change-Id: Ia18152457fe3ff70401b199c267fa37374b9d544
Signed-off-by: hsuan-chih_chen <hsuan-chih_chen@asus.com>
diff --git a/drivers/mmc/core/mmc.c b/drivers/mmc/core/mmc.c
index dc4b125..ea1eca7 100644
--- a/drivers/mmc/core/mmc.c
+++ b/drivers/mmc/core/mmc.c
@@ -293,7 +293,7 @@
 	}
 
 	card->ext_csd.rev = ext_csd[EXT_CSD_REV];
-	if (card->ext_csd.rev > 6) {
+	if (card->ext_csd.rev > 7) {
 		pr_err("%s: unrecognised EXT_CSD revision %d\n",
 			mmc_hostname(card->host), card->ext_csd.rev);
 		err = -EINVAL;
```
So I had a hunch that this was it.  Now the interesting part is that unlike CyanogenMod the kernel is NOT built as part of the Ubuntu Touch build (at least following the guide), it actually comes from a separate package.  

The flo branch of the Ubuntu kernel doesn't have the fix applied.  Hopefully this will be fixed soon.  

I built a patched kernel using ubuntu-wily source on the flo branch and pushed it into otherwise unmodified boot and recovery images.  We aren't touching the bootloader so this should be safe, but I carry no responsibility if the following procedure bricks your device.

Download boot.img and recovery.img from here: 

```
1) Return your device to stock:
	a. Obtain Google Factory image "razor-lmy48m-factory-7c77e178.tgz"
	b. Run "flash-all.sh" with device in the bootloader
	c. Let the tablet boot up, then just power it off.

2) Boot to bootloader.
3) Run 'fastboot flash boot boot.img'
4) Run 'fastboot flash recovery recovery.img'
5) Boot device to recovery
6) Run 'ubuntu-device-flash touch --channel=ubuntu-touch/stable/ubuntu'  # This should run through and say: "Rebooting into recovery to flash" - wait until the program exits.
7) The device should reboot automatically and start spinning the Ubuntu logo.  Wait this out - takes 5 to 10 minutes.  I believe this is Ubuntu installing itself.
8) The device will reboot and get stuck on the Google logo.  Power the device off.
9) Boot the device into the bootloader.  Repeat steps 3 and 4 to reflash the boot and recovery images.
10) Power off and start the device.  Ubuntu should start booting fairly quickly.
```

If you update your Ubuntu image you may have to reflash the boot.img and recovery.img, until this is fixed in the official "flo" branch of the Ubuntu kernel repo.
