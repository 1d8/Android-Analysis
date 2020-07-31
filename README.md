# Setting Up An Android VM For Analyzing Mobile Applications

## Download Links:

* [apksigner](https://apkpure.com/apk-signer/com.haibison.apksigner/download?from=details) 
* [Google Chrome](https://www.apkmirror.com/apk/google-inc/chrome/chrome-84-0-4147-89-release/google-chrome-fast-secure-84-0-4147-89-14-android-apk-download/)

## Tools Used:

* Genymotion
* BurpSuite
* Apktools

## Getting Started

Genymotion will be used to set up the Android VM. Sign up for an account [here](https://www.genymotion.com/account/create/). For account creation, if you don't want to use your real information, you can use temp-mail.org for email verification. You can also install Genymotion [here](https://www.genymotion.com/download/)

After installing Genymotion & signing in, you will be greeted with a screen like this, except yours will have no devices listed:

![](/images/img1.png)

Click the pink plus button then pick any device you'd like and set whatever specs you want for that device. When you get to the **Virtual Device Installation** screen (see img), leave the **Network Mode** defaulted to NAT. 

**NOTE**: The default amount of resources dedicated to the VMs is quite abundant, I usually set the number of processors to 1 & the ram to 2 GB. I haven't ran into any lag issues or anything of that nature.

![](/images/img2.png)

While that's installing, let's set up Burp.

After opening Burp, go into the **proxy** tab then into the **options** tab. Add a new listener on **all interfaces** on whatever port you'd like, I chose 8080:

![](/images/img3.png)

Now click import/export CA certificate > Export > Certificate in DER format > Choose a path & name it anything with a **.cer** extension > Next

Now let's start up our Android device & set up the proxy & install the certificate.

To install the certificate, run the following command:

`/opt/genymotion/tools/adb push *certificate name* *file location to push to*`

This will use Genymotion's built-in ADB to download the certificate to the Android device. I usually just push the certificate to */sdcard*.

Now go into the device's WIFI settings & click on the network that it's currently connected to. Then click the pencil in the upper right hand corner & click the **Advanced options** drop down menu & set **Proxy** to manual.

![](/images/img4.png)

1. For **hostname**, enter the IP address of the local machine that is running burp suite.

2. For **Proxy port**, enter the port that burp is listening on.

Now back out of the wifi settings, and scroll down to **Security & Location** click it, then click **Encryption & credentials**. Now click **Install from SD card** & find where you saved your certificate from earlier.

Give the certificate a name:

![](/images/img5.png)

Then after you click ok you're greeted with a screen asking you to set up a passcode, comply & follow the following instructions:

![](/images/img6.png)

Now it's time to test if we are able to intercept all traffic from the device. I prefer using Chrome as I've found that the pre-installed webview that comes with Genymotion is quite buggy.

You can install the Chrome apk on your host machine [here](https://www.apkmirror.com/apk/google-inc/chrome/chrome-84-0-4147-89-release/google-chrome-fast-secure-84-0-4147-89-14-android-apk-download/)

After the apk is installed on your host machine, simply drag & drop it into the Genymotion window and you'll get this popup on the VM:

![](/images/img7.png)

Launch chrome on the device & make sure intercept is on in Burp, then go to any website and you should see the request pop up in Burp:

![](/images/img8.png)

But some applications don't like user downloaded certificates, so in order to inspect web traffic for some apps we actually have to decompile the application & add a few things & recompile it

## Recompiling & Decompiling

For this example, I will be using an NYC transit app which I installed from ApkPure.com

First we decompile the app:
`apktool d *file-name*.apk`

Output:

![](/images/img9.png)

Then we go into the **Manifest.xml** file & scroll down to the <\application android> tag & we are going to add the following line if it isn't already there: 

`android:networkSecurityConfig="@xml/network_security_config`

Before adding:

![](/images/img10.png)

After adding:

![](/images/img11.png)

Now go into the **res/xml** folder & create/modify a file named network_security_config.xml with the following contents:

```
<network-security-config>  
      <base-config>  
            <trust-anchors>  
                <!-- Trust preinstalled CAs -->  
                <certificates src="system" />  
                <!-- Additionally trust user added CAs -->  
                <certificates src="user" />  
           </trust-anchors>  
      </base-config>  
 </network-security-config>
```

Then save the file & back out of all the directories & rebuild the apk with the following command:
`apktool b *folder-name/* -o *output-file.apk*`

Output:

![](/images/img12.png)

Now use Genymotion's ADB to push the modified apk to the Android device:

`/opt/genymotion/tools/adb push *file-name*.apk /sdcard`

Now before you can launch this apk, it needs to be signed. This can be done with [Apksigner](https://apkpure.com/apk-signer/com.haibison.apksigner/download?from=details), simply download Apksigner & drag & drop it into the device & launch it:

![](/images/img13.png)

Click **sign a file** & then find the modified apk tool that was pushed onto the device. 

You may have to click the 3 stacked boxes in the upper right corner & click show storage to find the modified apk:

![](/images/img14.png)

Then choose where you want to save your newly signed apk & click save: 

![](/images/img15.png)

Wait for the apk to be signed

Then go into the Amaze file manager & go to the folder where you chose to save the apk & click on it, this will bring this screen:

![](/images/img17.png)

Click install. You may need to allow Amaze file explorer to install applications, just do so.

Now you can intercept the mobile application's traffic & search for bugs:

![](/images/img18.png)
