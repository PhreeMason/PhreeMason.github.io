---
layout: post
title:  "Set up React Native on Windows"
date:   2017-09-03 00:51:48 +0000
---


1. The first thing you need to do is intall Nodejs which can be found [here](https://nodejs.org/en/).
  * Once the installation is complete you can check it by searching 'Node' in the start window. 
  * This search should reveal Node.js, Node.js command prompt, and a few links to the Node.js documentation and website.
  * The installation should also come along with npm which is a packaga manager. 
  * You can check this by opening up the Node.js command prompt and running the command `npm -v` which will return your current version.

2. Next, we will update npm by running `npm install -g npm`. This command installs the latest version of npm globally on your system.

3. After that we will install React Native with the command `npm install -g react-native-cli`. This will install the react native command line which allows you to scaffold react native apps.

4. Install the android sdk for creating adroid apps from. In order to do this you must first install the Java development kit first.
   * The Java development kit can be found [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).
   * Choose the option that best matches your windows operating system.
   * Once that is installed you can install android studio.

5. Once both are installed you will need to setup your environment variable.
    * To do this this start and right click on computer and click 'properties'
    * This should bring you to the system window.
    * On the left hand side click Advanced system settings then click 'Environment Variable'
    * click 'new' under the user variables section and enter 'ANDROID_HOME' for variable name.
    * Then enter the location of your sdk file under variable value. In my case this location is 'C:\Users\Omars-PC\AppData\Local\Android\sdk'
    * Then add the next variable 'JAVA_HOME' with variable value of lacation. For me it is 'C:\Program Files\Java\jre1.8.0_144'
 
