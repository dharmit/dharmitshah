+++ 
date = 2022-04-11T17:05:44+05:30
title = "Use OpenJDK with IntelliJ IDEA on Fedora"
+++

This post explains setting up IntelliJ IDEA to use the OpenJDK on Fedora. It is more for my personal future reference than anything else.

Download the things:

1. Download the latest "Community" edition of IntelliJ IDEA from [its website](https://www.jetbrains.com/idea/download/#section=linux). `cd` to the directory where its downloaded, extract the tar ball and start the IDE:
    ```sh
    $ cd Downloads/
    $ tar xvf ideaIC-2021.3.3.tar.gz
    $ cd idea-IC-213.7172.25/bin
    $ ./idea.sh
    ```
2. Optionally, create a launcher for it:
    ```sh
    $ cat /usr/local/share/applications/idea.desktop 
    [Desktop Entry]
    Encoding=UTF-8
    Name=Idea
    Type=Application
    Exec=sh /path/to/idea-IC-213.7172.25/bin/idea.sh
    Icon=/path/to/idea-IC-213.7172.25/bin/idea.png
    Categories=Programming
    ```
3. Install OpenJDK on Fedora:
    ```
    $ sudo dnf install -y java-11-openjdk java-11-openjdk-devel
    ```
4. Open IntelliJ IDEA and click on "New Project".
5. If no SDK was detected by IntelliJ IDEA, click on the drop-down for "Project SDK", enter the path `/usr/lib/jvm/java-openjdk`, and click OK.
