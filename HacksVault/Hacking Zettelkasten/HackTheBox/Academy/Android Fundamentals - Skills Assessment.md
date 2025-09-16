# Description

Interacting with the emulator, executing Linux commands, and navigating important directories are essential steps for progressing to the next modules.

The following questions will allow you to practice some of the steps showcased in the previous sections. Given what you've learned, set up an emulated device and answer the questions below.
# Setting up the Enviroment

You can check [Android Fundamentals - Setting Up The Testing Environment](./Android%20Fundamentals%20-%20Setting%20Up%20The%20Testing%20Environment.md) where we launch an emulated phone in Android Studio,
then we proceed to download and unzip the first version of the given app:

```bash
└─$ unzip myapp_os.zip 
Archive:  myapp_os.zip
replace myapp.apk? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
  inflating: myapp.apk  
```
# Installing App via ADB

We can install the app with the following command:

```bash
└─$ adb install myapp.apk     
Performing Streamed Install
Success
```

We restart the server as `root` and spawn a shell:

```bash
└─$ adb root                                                                                                       

└─$ adb shell       
emu64xa:/ # whoami
root
```
## Find Home PATH

Now we can find the app's home path:

```bash
emu64xa:/data/data/com.hackthebox.myapp # ls
cache  code_cache
```
## Find the UID

We `cd` in the `com.android.settings` folder and `ls -la`

```
2|emu64xa:/data/data # cd com.android.settings                                                        
emu64xa:/data/data/com.android.settings # ls -la
total 40
drwx------   4 system system  4096 2025-09-16 17:31 .
drwxrwx--x 226 system system 12288 2025-09-16 17:31 ..
drwxrws--x   2 system system  4096 2025-09-16 17:31 cache
drwxrws--x   2 system system  4096 2025-09-16 17:31 code_cache
```

Now we search the `com.android.settings` user `id`:

```bash
emu64xa:/data/data/com.android.settings # id -u system
1000
```

The answer is actually `system`.
## Sign the App

We `unzip` the new file and clean the device (WIPE):

```bash
└─$ unzip myapp_sign.zip 
Archive:  myapp_sign.zip
  inflating: myapp.apk       
```

Now we sign the application with `apksigner`

```bash
└─$ apksigner sign --ks ../AndroidStudioProjects/keystore.jks --out myappsigned.apk myapp.apk 
Keystore password for signer #1: 
```

Now install the app and run it:

```bash
└─$ adb install myappsigned.apk                                                              
Performing Incremental Install
Serving...
All files should be loaded. Notifying the device.
Success
Install command complete in 601 ms
```
# Native App Building

Following the steps provided in the Native Apps section, develop and deploy an application that will print the string returned from the `Build.MODEL` constant. Use the 'Pixel 3a API 34 Google APIs' (other emulators might work as well). What is the value of this string?

Native apps are software applications written in a specific programming language and tailored to run on a particular platform. In the context of Android, native apps are primarily developed using Java or Kotlin and built with Android Studio, typically leveraging components from the Android Software Development Kit (SDK).

Google, the tech giant who owns and develops Android, is using [Kotlin as the default programming language](https://developer.android.com/kotlin/first) for app development. However, [Javascript](../../../3%20-%20Tags/Programming%20Languages/Javascript.md) is still a popular choice and many users prefer it over [Kotlin](../../../3%20-%20Tags/Programming%20Languages/Kotlin.md). In the following steps, we will show how to create a simple application in Java. Since we have already created a new project, let's examine how the app layout is created and connected with the Java code and other resources. On the left side section under `app/res/layout` we select the `activity_main.xml` file.

Layouts define the structure of the user interface of the application, and Android uses XML to create such layouts. In this file, `activity_main.xml`, we can add text, buttons, images, and other things that will be displayed to the user. The following snippet will create the layout of an application that prints a message on the screen when the button is tapped:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="My Application"
        android:textSize="32sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.097" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/title"
        app:layout_constraintVertical_bias="0.403" />

    <TextView
        android:id="@+id/message"
        android:layout_width="380dp"
        android:layout_height="31dp"
        android:text="@string/message"
        android:textSize="20sp"
        android:textAlignment="center"
		android:textIsSelectable="true"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/button"
        app:layout_constraintVertical_bias="0.25" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

The above snippet consists of three objects: two `TextView` and one `Button`. It is worth noticing the following attributes.

| **Attribute**                  | **Description**                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tools:context=".MainActivity` | Defines the Activity in which the layout will be used. This is primarily used in the layout editor for preview purposes and does not affect the runtime behavior of the app.                                                                                                                                                                                                                                        |
| `android:id`                   | Assigns a unique identifier to the object, allowing it to be referenced in Java code (such as`MainActivity.java`) and other resources. The `@+id` prefix indicates it will be created as a new resource, whereas `@id` would be used if the resource were already defined.                                                                                                                                          |
| `android:text`                 | Sets the text content of the TextView or Button. The value can be a hardcoded string, like `android:text="My Application"`, or a reference to a string resource from the relative to the project file `res/values/strings.xml`. Referencing string resources is the recommended approach for localization and maintainability. The example below shows the contents of the `strings.xml` file used in this project. |

```xml
<resources>
    <string name="app_name">My Application</string>
    <string name="message">Hello World!</string>
</resources>
```

Let's now see the content of the file `MainActivity.java`, which contains the Java code of the app:

```java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
    TextView message;
    Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        message = (TextView)findViewById(R.id.message);
        button = (Button)findViewById(R.id.button);

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                message.setText(Build.MODEL);
            }
        });
    }
}
```

The class `MainActivity` includes the method `OnCreate()`. This method is called when the activity is starting, thus everything inside will run automatically. The line `setContentView(R.layout.activity_main);` indicates that the `activity_main.xml` file will be used to set the layout of this activity. Method `OnCreate()` is also used for initializations since it runs when the activity starts. The variables `message` and `button` are initialized to point to the corresponding objects found in the `activity_main.xml` file.

```java
message = (TextView)findViewById(R.id.message);
button = (Button)findViewById(R.id.button);
```

Using the `R.java` file, the `R.id.message` points to the `android:id="@+id/message"` attribute in the `activity_main.xml` file we saw earlier. Finally, the line `message.setText(Build.MODEL);` will be executed as soon as the button is clicked, like the line `button.setOnClickListener` indicates. Once it's tapped, the text `Hello from Java!` will be set in the TextView.

Once the app is developed, we can export a signed APK file ready for installation. Under Build -> Generate Signed Bundle / APK..., we select APK and click Next.

On the next window, we click `Create new...` to create a new key.

Then, we set the `Key store path`, `Password`, `Alias,` and `First and Last Name`, and click `OK`.

![Pasted image 20250916195847.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250916195847.png)

Finally, we select the option `release` and click on `Finish`.

The signed APK file can be found under the directory `AndroidStudioProjects/MyApplication/app/release/`, with the name `app-release.apk`

We can rename and install the exported APK file directly on the device.
