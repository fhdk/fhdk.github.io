---
published: true
date: '2019-10-01 00:00'
publish_date: '2019-10-01 00:00'
title: 'Run a Java program on Manjaro'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## What is a jar file
A jar is java package - a zip-compressed archive - including the code for a java application.

## How can I start the application
The jar requires at least JRE (Java Runtime Environment) - some requires the full JDK (Java Development Kit). A JDK contains the corresponding JRE.

If in doubt of the actual requirements of the application - install the JDK.

To install the latest JRE

    sudo pacman -Syu jre-openjdk

Or the complete environment

    sudo pacman -Syu jdk-openjdk

## Commandline

To run the application contained in the jar

    java -jar java-app.jar

## Create a script and a launcher

If you want to launch the program from your menu or the terminal - create a script and a launcher as follows.

1. **Create the bin folder**

       mkdir ~/.local/bin

2. **Move the jar to the bin folder**
Assuming it is in Downloads

       mv ~/Downloads/java-app.jar ~/.local/bin

3. **Create the script**
Open your favorite text editor with a new file and save it as *~/.local/bin/java-app* with below content - the **"$@"** ensures that any arguments to the script is passed on to the java-app.jar file

   ```
   #!/bin/sh
   java -jar ~/.local/bin/java-app.jar "$@"
   ```

4. **Make the script executable**

       chmod +x ~/.local/bin/java-app

5. **Create the application folder**

       mkdir -p ~/.local/share/applications

6. **Create the desktop launcher**
Use the text editor to create a new file and save it as  *~/.local/share/applications/java-app.desktop* with below content

   ```
   [Desktop Entry]
   Encoding=UTF-8
   Name=Some Java App
   Comment=Run the java-app.jar program
   GenericName=Some Java App
   Exec=~/.local/bin/java-app
   Terminal=false
   Type=Application
   Icon=java-app-icon
   Categories=Application;
   StartupWMClass=java-app
   StartupNotify=true
   ```

## Conclusion
You are now be able to run **Some Java App** by providing the script name and any arguments for the java-app.jar and you can find it in the system's menu tree.
