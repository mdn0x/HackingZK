# Android Studio

Android Studio is an Integrated Development Environment (IDE) based on JetBrains' IntelliJ IDEA and is the official IDE for Android application development. Familiarity with the Android Studio project structure can provide valuable insight during reverse engineering. Android Studio is available for Windows, Linux, and macOS. On Windows and macOS, you can [download](https://developer.android.com/studio) the `.exe` and `.dmg` installers respectively, and follow the Setup Wizard to complete the installation. In this example, we will demonstrate how to install Android Studio on a Debian-based Linux distribution. Replace the version number in the link with the latest one, if necessary:
## Install and Create new project

#### Debian-based Linux

```bash
tar xvzf android-studio-2025.1.3.7-linux.tar.gz
sh android-studio/bin/studio.sh
```

Once the Setup Wizard starts, click `Next` on the first four windows to download the SDK and accept the License Agreement. Then, wait for the components to download and click `Finish`:

![Pasted image 20250916142017.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916142017.png)

On the next window, click on the `Create Project` button to create a new project:

![Pasted image 20250916143028.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916143028.png)

Select `Empty Views Activity` and choose `Groovy DSL (build.gradle)` in the `Build configuration language` option:

![Pasted image 20250916143337.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916143337.png)

Finally, we give the app a name, select the programming language (Java in our case) and click `Finish`.

After clicking 'Finish', we wait for the indexing process to complete before proceeding with the following steps. The above setup configures the project to use Java as the default programming language.
## Project Structure

Once the project is created, we see the project structure.

Understanding the Android Studio project structure is essential for reverse engineering, as it provides insight into how the app is organized during development.

The project contains the following folders:
 
 - **app**

| **Files**  | **Description**                                                                                                                                                          |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `manifest` | Contains essential metadata about the app, including the package name, components (activities, services, etc.), permissions, network configuration, API level, and more. |
| `java`     | Contains the application's Java source code.                                                                                                                             |
| `res`      | Contains app resources such as UI strings, images, layout XML files, and other static assets used by the app.                                                            |

- **Gradle Scripts**

| **Files**            | **Description**                                                                                                                                                  |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `build.gradle`       | Defines build configurations for the project or module, including dependencies, build types, and whether code optimization tools (such as ProGuard) are enabled. |
| `proguard-rules.pro` | Specifies custom rules for ProGuard                                                                                                                              |
## Types of Applications

Depending on the technologies used, Mobile applications can be categorized into three basic types:

1. **Native apps**
Native apps are built specifically for a particular operating system. Android native apps are typically developed using Kotlin or Java. They are generally considered more secure than hybrid apps because they have direct access to platform-level security features and system APIs.

2. **Web apps**
Web apps, while similar in appearance to native apps, are developed to be responsive and accessible from mobile web browsers. They are typically built using HTML, CSS, and JavaScript. Web apps can be vulnerable to web-based attacks due to their reliance on web technologies and browser security.

3. **Hybrid apps**
Hybrid apps combine elements of both native and web apps and are designed to be cross-platform. They use WebViews to display web content within a native app container. While offering the flexibility of web apps, hybrid apps can also be vulnerable to web-based attacks, including cross-site scripting (XSS) attacks, weak SSL implementations, and more.
# Android Emulators

Setting up the testing environment requires our host machine, an emulated Android device, and the tools we will use for the application penetration testing. In the following paragraphs, we will discuss various Android emulators and set up one we are going to be using during the next sections.

Android emulators are a vital tool for application penetration testers. They are programs allowing users to run Android applications and simulate the Android operating system on devices like personal computers. They are widely used for app development, testing, and gaming purposes. Android emulators are a cost-effective solution that allows testing applications on various devices and Android API levels without needing each physical device. Apart from this, emulators also provide most of the capabilities that a real device does, like simulating incoming phone calls and text messages, camera, Bluetooth, sensors, accessing the Google Play Store, and more. Penetration testers can benefit from emulators since they can simulate real-world scenarios and evaluate Android application vulnerabilities.

Most real Android devices use ARM or ARM64 CPU architecture, specifically designed for mobile devices and providing high-energy performance. However, Android emulators use x86 or x86_64 CPU architectures for better performance and compatibility with the host machine's hardware. Different CPU architectures can affect how exploits or vulnerabilities behave in emulated environments compared to real devices. In the following sections, we will use Android Virtual Device (AVD) for testing applications, which is included in the Android Studio IDE.
##  Android Studio AVD

Android Studio's AVD is a fast and feature-rich emulator that allows us to test applications. It will help us run and debug applications developed with Android Studio in order to exploit other apps. Android Studio includes the AVD Manager, which simplifies creating, configuring, and managing Android Virtual Devices to simulate real-world scenarios. The following steps will show how to set up an Android Virtual Device (AVD) using Android Studio.

Since we have already seen how to install Android Studio and create a new project, let's jump directly to creating the AVD. In a new Android project, navigate to `Tools` -> `Device Manager`.
Let's see the process of creating a new device by clicking on the button under `Tools` -> `Device Manager` -> `Create device`. Along with other properties, an AVD contains a hardware profile that defines the characteristics of a device as shipped from the factory. The Device Manager comes pre-equipped with certain hardware profiles (e.g., Pixel devices), allowing users to define or tailor these profiles according to their requirements.

![Pasted image 20250916152020.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916152020.png)

Navigating to the `x86 Images` we can see the different Android API levels. The Google Play logo in the `Play Store` column indicates that the device includes Google Play Store and access to other services.Assuming we select the `Pixel 3a` device, moving on to the next step, we are prompted to choose a system image we choose `API 34 Google APIs` and download it:

![Pasted image 20250916152239.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916152239.png)
![Pasted image 20250916152502.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916152502.png)

System images labeled with `Google APIs` provide access only to Google Play services, while images labeled with `Google Play` also provide a Google Play Store. If the download icon appears near the Release Name, the image is not installed, and we must download it. System images that include the Google Play Store are signed with a release key, and thus elevated privileges (root) are not supported. On the other hand, system images that only include `Google APIs` will allow elevated privileges (root) by using the ADB tool through the terminal. This is a necessary feature that we will utilize while conducting application penetration testing.

Install SDK frameworks:

![Pasted image 20250916154813.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916154813.png)

Once the emulator is launched, it will look like this:

![Pasted image 20250916170857.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916170857.png)

On the device's right side, there is a vertical menu bar. We can stop the emulator by clicking the `x` icon on the top. Finally, among other action buttons in the menu bar, there is a 3-dot icon that contains various device settings we can configure as the device is running.
## Other Emulators

Although Android Studio AVD is the emulator we will use throughout the following sections, several other emulators are available with unique features and capabilities. Some of the popular alternatives include the following emulators.

| **[Corellium](https://www.corellium.com/)**                                               |
| ----------------------------------------------------------------------------------------- |
| Supports a wide range of mobile devices including both Android and iOS operating systems. |
| Offers a web-based interface.                                                             |
| Provides integrated advanced debugging tools.                                             |
| Enables users to take snapshots of the emulator.                                          |
| Emulates both standard and rooted/jailbroken devices.                                     |

|**[Genymotion](https://www.genymotion.com/)**|
|---|
|Designed for app developers and testers|
|Offers a wide range of virtual devices with different Android versions|
|Supports advanced features like camera, GPS, and accelerometer emulation|
|Integrates with development platforms like Android Studio and Xamarin|
|Available for Windows, macOS, and Linux|

| **[BlueStacks](https://www.bluestacks.com/)**                   |
| --------------------------------------------------------------- |
| Widely popular and user-friendly                                |
| Multi-instance feature for running multiple apps simultaneously |
| Integrates with gaming platforms like Twitch and Discord        |

|**[NoxPlayer](https://www.bignox.com/)**|
|---|
|Lightweight and efficient emulator|
|Supports gamepad, keyboard, and script recording|
|Offers advanced features like GPS spoofing and virtual location|

|**[MEmu Play](https://www.memuplay.com/)**|
|---|
|High performance and compatibility|
|Offers multiple Android versions|
|Allows multiple instances and easy app cloning|
|Keyboard mapping and joystick support|

|**[LDPlayer](https://www.ldplayer.net/)**|
|---|
|Optimized for gaming and productivity apps|
|Customizable controls and graphics settings|
|Multi-instance support and synchronization|
# Android Debug Bridge

`Android Debug Bridge` (ADB) is a versatile command-line tool that enables communication between a computer and a device. As a part of the Android SDK (Software Development Kit), it allows developers to perform tasks like installing and debugging applications, transferring files between the host computer and the device, and accessing the device through a shell. `Android Debug Bridge` is a client-server program that can be used on physical and emulated devices, and it includes the following three components.

|**Components**|**Description**|
|---|---|
|`The client`|It runs on the host machine and can be invoked from the terminal by issuing the `adb` command.|
|`The daemon (adbd)`|It runs as a background process on each device, and it is responsible for executing the commands in the device.|
|`The server`|It runs as a background process on the host machine, and it manages the communication between the client and the daemon.|

The following steps show how the ADB connection process works.
 
 - Step 1

The ADB client will start as soon as we issue an `adb` command in the terminal. When this happens, the client will check for an `adb` server process running on the host machine. If there isn't one, it starts the server process, which binds to the local TCP port `5037` and listens for commands sent from the `adb` client.

- Step 2

The server then locates the connected emulators by scanning the odd-numbered ports from `5555` to `5585`. Finally, it sets up the connection with the emulators that run an ADB daemon (adbd).
 
 - Step 3

Each emulator instance has a unique pair of ports to avoid conflicts when running multiple emulators simultaneously. These pairs are an even-numbered port for console connections and an odd-numbered port for ADB connections. In the console connection, we can issue commands to control the emulator using tools like `telnet`. In the ADB connections, we can perform various development and debugging tasks using the `adb` command. The following is an example of two connected emulators.

| **Emulator** | **Service** | **Port** |
| ------------ | ----------- | -------- |
| `Emulator 1` | console     | 5554     |
| `Emulator 1` | adb         | 5555     |
| `Emulator 2` | console     | 5556     |
| `Emulator 2` | adb         | 5557     |

- Step 4

Once the connections are set up, we can use the `adb` command to access the emulators.

## Installing ADB 

In the following paragraphs, we will see how to install and use ADB. The example below shows the installation process on three different operating systems.

#### Debian-based Linux

On Debian-based Linux, through the `APT` package manager:

```bash
$ apt-get install adb
```

#### MacOS

On MacOS, through the `Homebrew` package manager:

```bash
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
$ brew update
$ brew install android-platform-tools
```
#### Windows

On Windows, through the `Scoop` command-line installer.

```powershell
C:\> Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
C:\> iex (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
C:\> scoop bucket add extras
C:\> scoop install adb
```

`ADB` is also made available to us as part of the [Android SDK Platform Tools package](https://developer.android.com/tools/releases/platform-tools), available through Android Studio's [SDK Manager](https://developer.android.com/studio/intro/update#sdk-manager). On Windows, the resulting platform tools including `adb.exe` are located at `C:\Users\<username>\AppData\Local\Android\Sdk\platform-tools`
## Using ADB

To start the ADB server on the host computer, we can type the following command:

```bash
$ adb start-server
```

Once the ADB is installed on our host computer and the emulator is started, we can start issuing ADB commands. As mentioned earlier, starting the ADB server can also be done by directly executing commands using the ADB client. Below are some useful ADB commands.

| **Command**                              | **Description**                        |
| ---------------------------------------- | -------------------------------------- |
| `adb help`                               | List all commands.                     |
| `adb kill-server`                        | Kills the adb server.                  |
| `adb devices`                            | Lists connected devices.               |
| `adb root`                               | Restarts `adbd` with root permissions. |
| `adb install <apk>`                      | Install app.                           |
| `adb push <local> <remote>`              | Copy file/dir to device.               |
| `adb pull <remote> <local>`              | Copy file/dir from device.             |
| `adb logcat [options] [filter] [filter]` | View device log.                       |

Let's list the connected devices using the following command.

```bash
$ adb devices

List of devices attached
emulator-5554   device
```

The above command reveals one attached device using port `5554`. Apart from providing interactive shell access to the emulated device, ADB can also be used non-interactively to execute commands from the host machine directly to the emulated device. The following `adb` command executes the `whoami` command inside the emulated device and prints the result to the terminal.

```bash
$ adb shell whoami
```

Let's issue the following command to get a shell in the emulated device.

```bash
$ adb shell

emu64x:/ $
```

We can now start executing Linux commands inside the emulator:

```bash
emu64xa:/ $ whoami
shell
```
## ADB Root

ADB can also provide the user with elevated privileges. The `adb root` command restarts the ADB daemon (adbd) on the Android emulator with root privileges. It gives elevated permissions to the user that allows for a wider range of command execution, as well as access to system files and directories that are typically restricted to regular users. Root access allows us to perform more in-depth debugging and testing during the application penetration testing. This said, running ADB as root is typically only available on emulators, as most production devices come with locked bootloaders and cannot be rooted without voiding the warranty. Also, as mentioned earlier in the `Android Emulators` section, only system images labeled with `Google APIs` will provide this feature. System images labeled with `Google Play` are signed with a release key, and elevated privileges (root) are not supported.

The following command will restart the `adbd` with root privillages.

```bash
$ adb root
```

Once it's restarted, we can execute the following command to confirm that we can run commands as the user `root`.

```
$ adb shell whoami

root
```

## Installing Apps and Transferring Files

Android Debug Bridge also allows us to install applications. Let's try to install the application we created in the section `Native Apps` using the following command.

```bash
$ adb install myapp.apk 

Performing Streamed Install
Success
```

Once installed, we can locate the app on the device and tap on it to start. Another useful command of ADB is the `push` and `pull` commands. Using these commands, we can transfer files from the host computer to the device and vice versa. To push files from the host to the device, we can execute the following command.

```bash
$ adb push ./myapp.apk /sdcard/Download/

./myapp.apk: 1 file pushed, 0 skipped. 14.9 MB/s (4527105 bytes in 0.291s)
```

We can verify by listing the content of the directory `/sdcard/Download/` in the device.

```bash
$ adb shell ls -l /sdcard/Download/

total 4428
-rw-rw---- 1 u0_a146 media_rw 4527105 2023-05-03 01:44 myapp.apk
```

Similarly, we pull files from the device to the host by executing the following command.

```bash
$ adb pull /sdcard/Download/flag.zip .
```

## Setting Up a Real Device

Setting up ADB with a real Android device requires some extra steps on the device's side. After installing ADB on the host machine, we also have to enable `USB Debugging` on our Android device. To do this, we first have to go to `Settings` -> `About phone` or `About device` or `About emulated device`, find the `Build number`, and tap on it seven times. This will display the message `You are now a developer!`.

The `Developer Options` menu can be found under `Settings` -> `Developer Options` or `Settings` -> `System` -> `Developer Options`, depending on the Android version. After navigating to the path we mentioned earlier, the next step is to enable `USB Debugging` and confirm any prompts that appear.

Once enabled, we can connect the device to the computer using a USB cable. When prompted on the device, choose `File transfer` or `MTP` mode. We can confirm that the device is connected by executing the following command.ù

```bash
$ adb devices

List of devices attached
1234567890ABCDEF	device
```

The device's serial number should be included in the output, indicating that ADB is successfully connected to your device. Finally, we can start executing `adb` commands from our host computer to the device.

# Application Penetration Testing - Methodology and Tools

---

Mobile applications and services are vital for both our personal and professional life, making them prime targets for malicious actors seeking sensitive data. Modern Android applications are widely used in commerce, healthcare, banking, education, and more. Beyond storing sensitive information, these applications often contain security vulnerabilities that could be exploited. Penetration testers and developers play a crucial role in identifying and mitigating these vulnerabilities, reducing risks such as fraud, malware infections, and data breaches. For companies launching new applications, ensuring security and compliance is essential to avoiding legal issues and protecting user data.

Mobile application penetration testing is the process of assessing an app's security by simulating real-world attack scenarios. This methodology helps identify vulnerabilities and ensures that applications remain resilient against threats. The following steps outline a structured, high-level approach that Android penetration testers should follow for a comprehensive security evaluation.

#### 1. Planning and Environment Setup

|**Objectives**|
|---|
|Define the scope and objectives of the penetration test.|
|Obtain necessary permissions from the app owner or organization.|
|Set up the testing environment, including the necessary tools, operating systems, and emulators.|

#### 2. Enumeration and Information Gathering

|**Objectives**|
|---|
|Gather information about the app and its architecture.|
|Understand the functionality of the application.|
|Enumerate any data structure the app uses that is stored in the local storage.|

#### 3. Static Analysis

|**Objectives**|
|---|
|Understand the manifest file of the application. Review the app's components, permissions, and configurations.|
|Decompile and analyze the application's source code to understand the flow and find potential vulnerabilities.|
|Examine native or third-party libraries, frameworks, and dependencies the app uses.|

#### 4. Dynamic Analysis

|**Objectives**|
|---|
|Monitor the application's behavior during runtime. Enumerate the local storage for files and data structures created after performing various functionalities.|
|Perform dynamic instrumentation to read the memory of the application during runtime.|
|Intercept network traffic and test the app for vulnerabilities such as authentication bypass and insecure data transmission.|
|Check for Server-Side attacks by analyzing API calls and looking for vulnerabilities such as injection and XSS attacks.|

#### 5. Documenting and Reporting

|**Objectives**|
|---|
|Document the identified vulnerabilities, exploitation methods, impact, and potential risks.|
|Communicate the findings to the app owner or organization.|
|Provide recommendations for fixing vulnerabilities and mitigating security risks.|

---

## Suggested Tools

The following list presents some of the most widely used tools for mobile app penetration testing , focusing on `reverse engineering`, `dynamic analysis`, `code instrumentation`, `network traffic analysis`, and `digital forensics`. Familiarity with these tools is essential for anyone practicing their Android application penetration testing skills.

#### ADB

A versatile command-line tool that allows communication with Android devices. This tool will let us enumerate the local storage of the device and the installation directory of the application, see the app logs while it is running, and install and run other apps and tools necessary for the steps of the dynamic analysis and instrumentation.

#### JADX

This tool allows us to reverse engineer an application and view its source code through a graphical interface. It provides the user with a Java-like pseudocode that is close enough to the actual code. It also decodes the encoded resources that are packed in the APK file, making the file configurations and the other assets readable to humans. This tool is an essential part of the process since the testers can understand how the app works and what technologies are used.

#### APKTool

This tool also allows reverse engineering Android applications. It will decompile the source code and decode the resources of the APK file. APKTool also enables us to edit the source code and configuration files, recompile the code, and build the APK file again. This is called application patching. The decompiled code provided to the user is a symbolic language called `Smali`. This language is not easy to read, but if combined with a tool that offers Java-like pseudocode like JADX, pentesters can efficiently understand and change the flow of the application to their benefit.

#### Ghidra

An open-source software reverse engineering (SRE) framework developed by the National Security Agency (NSA). It is used for disassembling, decompiling, and analyzing binaries. In Android application penetration testing, Ghidra is often used to analyze the native C++ libraries loaded to the application.

#### Burp Suite

A tool used for web application penetration testing. While we often see it used for testing websites, it can also be used for testing any web-based application communicating with a server, including Android. In the Android application, penetration testing will be mainly used to intercept and analyze HTTP/HTTPS requests.

#### Frida

A dynamic instrumentation toolkit used by developers, reverse engineers, and security researchers. It enables us to inject snippets of JavaScript or native code into the running processes of Android applications, allowing us to analyze and manipulate them during runtime.

#### Autopsy

An open-source digital forensics platform that can be used for various tasks, including Android forensics. Autopsy will help us investigate disk images of Android devices and let us search for files, databases, calls, messages, and logs, using the provided GUI.

---

## Automated Tools

Automated Android penetration testing tools are suitable for scanning common vulnerabilities. They offer a faster and cheaper solution than manual tools and processes, which provide more depth and vulnerability insights by combining human intelligence with automated tools.

#### MobSF

An automated security testing framework for Android, iOS, and Windows platforms that performs static, dynamic, and malware analysis on mobile applications.

#### Drozer

A comprehensive security and attack framework that allows us to assess the security of Android applications. It simulates various attack vectors and provides multiple tools to analyze, exploit, and debug Android applications.

#### Qark

A static code analysis tool that automates the discovery of security vulnerabilities in Android applications. It can also create Proof-of-Concept deployable APKs and ADB commands to exploit the vulnerabilities it finds.

#### Objection

A runtime mobile exploration toolkit that uses Frida to provide an easy environment for assessing Android and iOS application security. It automates several common tasks, such as bypassing SSL pinning, and users won't be needed to create custom scripts whenever they need to exploit common vulnerabilities.

#### Medusa

An extensible and modularized framework that automates processes and techniques practiced during the dynamic analysis of Android Applications. Medusa is based on Frida and can analyze and enumerate an app, attack common entry points, and automate processes like application patching, MITM attack, and more. Medusa can add and remove hooks for Java or Native methods and has more than 90 modules that can be combined.

#### Androbugs

Androbugs is an Android vulnerability analysis framework that automatically scans Android applications and checks for security issues and vulnerabilities.

It is crucial to remember that combining both manual and automated techniques usually results in a better evaluation. Recapping this module, we went through the most critical fundamental concepts necessary for understanding the methodologies that will follow in subsequent modules. We analyzed the structure of the Android OS and the architecture of applications, discussed the different types of apps, and examined examples of applications built with various frameworks. We studied their components and IPCs, and learned how to set up a testing environment and interact with emulators.

Following the Penetration Testing methodology discussed in this section, we are now ready to prepare for the next module, which regards Android Application Static Analysis. In this upcoming module, we will reverse engineer apps and study the source code to identify security issues and exploit vulnerabilities.