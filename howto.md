# How to use the IdleDetector API

The IdleDetector API is currently available in chrome canaries under a flag:

1) Download chrome canary ([android](https://play.google.com/store/apps/details?id=com.chrome.canary), [desktop](https://www.google.com/chrome/canary/)).
2) Navigate to `chrome://flags` and enable `Experimental Web Platform features`.
3) Navigate to a test page, e.g. https://code.sgo.to/tmp/idle.html
4) Go idle

## On Android

On android devices, you can lock your screen by turning it off. Unlock it (using your pin or fingerprint) and you should get an event back.

## On Linux

On corp linux, locking detection relies on X11. Use the following to reproduce locking events:

sleep 5; xscreensaver-command -activate

If you use KDE/GNOME to lock your screen it doesn’t work, because it notifies the system through dbus instead. 
.
On KDE, DBUS sends messages to this channel:

qdbus org.freedesktop.ScreenSaver /ScreenSaver Lock

On GNOME, this is the channel:

TBD

## On Macs
## On Windows
## On ChromeOS


