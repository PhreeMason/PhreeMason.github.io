---
layout: post
title:  "Set up React Native on Windows"
date:   2017-09-02 20:51:49 -0400
---


1. The first thing you need to do is install Nodejs which can be found [here](https://nodejs.org/en/).
  * Once the installation is complete you can check it by searching 'Node' in the start window. 
  * This search should reveal Node.js, Node.js command prompt, and a few links to the Node.js documentation and website.
  * The installation should also come along with npm which is a package manager. 
  * You can check this by opening up the Node.js command prompt and running the command `npm -v` which will return your current version.

2. Next, we will update npm by running `npm install -g npm@latest-4`. This command installs the latest version of npm globally on your system.

3. After that we will install React Native with the command `npm install -g react-native-cli`. This will install the react native command line interface which allows you to scaffold react native apps.

4. Install the android sdk. You must also install the Java development kit.
   * The Java development kit can be found [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).
   * Choose the option that best matches your windows operating system.
   * Once that is installed get android studio from [here](https://developer.android.com/studio/install.html).

5. Once both are installed you will need to setup your environment variable.
    * To do this click start and right click on computer. Then click 'properties'
    * This should bring you to the system window.
    * On the left hand side click Advanced system settings then click 'Environment Variable'
    * Click 'new' under the user variables section and enter 'ANDROID_HOME' for variable name.
    * Then enter the location of your sdk folder under variable value. In my case this location is 'C:\Users\Omars-PC\AppData\Local\Android\sdk'
    * Then add the next variable 'JAVA_HOME' with location for variable value. For me it is 'C:\Program Files\Java\jre1.8.0_144'
 
6. Once that is all complete you can scoffold your first react app by running `react-native init APPNAMEHERE`
    * This will take some time to complete. After it finishes, open android studio and click the option to open existing project.
    * From there find the folder of the react native app and click on the android folder.
   
7. Next click the sdk manager icon at the top of the android studio window.
    * From there click Android SDK and click the SDK Platforms tab.
    * From this tab click the show package details checkbox, then check the boxes for each android version that you would like to build apps for and click apply.

8. After thats complete click AVD Manger Icon at the top of the Android Studio window.
    * This will show your virtual devices. You most likely don't have any so just create one and choose the device you would like to test your app with.
    * After creating your vitual device run it 

9. Next in the command prompt run the command 'react-native start' then open a second command line and run 'react-native run-android' 
    * This will compile your app and run it in the virtual device. 
    * You can use an code editor you choose to edit the code.

Thats it! Happy Coding.
