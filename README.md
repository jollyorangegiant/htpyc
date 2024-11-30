# htpyc
A simple 10 Foot interface for media libraries requiring little or no configuration

At present does require the following libraries to be installed locally via pip, eventually it will be packaged with standalone wheels for known good versions:

pygame, subprocess, sys, os, platform, threading, time, process_iter, re, json

By default it will present a directory/media file simplified file browser in a 10 foot interface. Files and folders will be detected in the directory where the script is launched from and subdirectories. Files will be opened in VLC with "--play-and-exit" and "--fullscreen" enabled. 

This behavior can be changed in htpyc.json. MPV is supported by the program and can be set in the "default player" value. The "defaultLaunchString" value can be used to set the launch string and arguments for another media player, ie "mplayer -fs".
