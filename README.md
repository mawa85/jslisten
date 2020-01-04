# jslisten

## Index
 1. [Overview](#overview)
 2. [Usage](#usage)
 3. [Installation](#installation)
 4. [Configuration](#configuration)
 5. [Known limitations](#known-limitations)
 6. [Multiple key combinations](#multiple-key-combinations)

## Overview

This program listen in the background for gamepad inputs. If a special button combination is getting pressed,
the provided command line will be invoked. It runs a command once defined buttons were pressed. It was developed for Raspbian and PS3 wireless controller but should work everywhere with udev and [joystick](https://sourceforge.net/projects/linuxconsole/) support.

## Usage

I have a Raspberry Pi. The installed operation system is raspbian. The Kodi package runs pretty well as a media center. The remote control is an App on a smartphone. So no other controller is needed.

After couple of month I found this great [RetroPie](https://retropie.org.uk) project. There is also another great one which brings it on a software level instead making a boot image. It's called [EmulationStation](http://www.emulationstation.org). The setup with my preferred PS3 controller and the configuration went well and it worked fantastic. The game console works perfectly fine and brings me back to the good old jump'n'run 90s but now with big screen and cool wireless controller!

But one thing was an issue. I wanted to use the same hardware and don't miss my system (raspbian). Dual boot is not cool and takes too long. So basically what i needed is some kind of trigger to end any X11/Kodi processes and start EmulationStation ... and of course vice versa.

In my configuration Kodi runs as default. Media center is still the primary usage. But the idea was to take the PS3 controller press some button combination and the EmulationStation will apear. After I played enough, just press this buttons again and media center will apear and I can put the PS3 controller back.

After searching the internet, I found nothing really interesting. Kodi addon will not work because gamepad support is still missing. EmulationStation addon might work but than ... Kodi solution will be missing. So I had to go one level up to the OS. But even here I found nothing really easy to setup or working. So I decided to write this program. It runs as a daemon in the background and listening in nonblocking mode to /dev/input/js* devices. To make it hotplug capable, it's monitoring the udev signals if no device is present.

<b>Update 12/22/2018:</b>
If you have many different /dev/inputs, you can pass it as an arguement at the startup:

```jslisten --device /dev/input/myinput/js0```

<b>Update 02/12/2019:</b>
Changed the default service user to "pi" in jslisten.service

Use user root only if it's really required.

<b>Update 02/16/2019:</b>
Added a different mode to easier trigger scripts.

When started with `--mode plain` (the default) the program behaves as used. This is fine if you need the button combination once in a while.

However if you want to activate a script a few times in a short time, lets say to adjust your audio volume or if you have a led stripe installed in your arcade and want to adjust the brightness, then
the `--mode hold` comes in handy.

When started with `--mode hold` and you have at least two buttons defined to enter an action, then you hold down the first one, while tipping the second one. If you have three buttons defined you can
hold the second two and then just tip the third. In a four button set you will have to hold the first three.

As soon as you release any but the last button (according to the order in the `jslisten.cfg`) you will have to press the all buttons again.

So the difference to the plain mode is that once you have entered the engaged/elevated state with a three button set for example, you hold down the first two and can quickly trigger the action by
pressing the third. In plain mode you would have to press all three buttons over again to do audio or brightness adjustment.

Also added `--help` for the command line for a brief summary of options.


## Installation

Following example for Raspbian. Should work for many other distributions almost the same way.
 * Use the precompiled binary in bin/ or run "# make" to create the binary
 * Requirements for compilation on raspbian: apt install build-essential libudev-dev
 * Place the binary to "/opt/bin" (if you change the folder, please update your init script)
 * Copy the configuration script to /etc/jslisten.cfg
 * Modify the configuration script to your needs
 * Copy the service script "utils/jslisten.service" to "/etc/systemd/system"
 * Update the systemd daemon: "# systemctl daemon-reload"
 * Start the daemon "# systemctl start jslisten.service"
 * Make it start at boot "# systemctl enable jslisten"

## Configuration

 * I assume your joystick setup is already done and you can see with "jstest /dev/input/js0" nice looking key strokes coming up. There are many tutorials out there and you should not proceed if this doesn't work for you!
 * Run ./jslisten once to create ~/.jslisten config file or copy utils/jslisten.cfg to /etc
 * Think which button combination will work for your gamepad. You can configure up to 4 button combination. In my case i picked "Left Shoulder" + "Right Shoulder" + SELECT. I'm pretty sure i will not need to press this buttons at the same time for any games.
 * Get the number ID's for this buttons with jstest (or any other similar program)
 * Edit the configuration file (either /etc/jslisten.cfg or ~/.jslisten) and maintain the button ID's and the program which you want to run.
 * Make sure this script/program can handle multiple simultanios calls! Even if the daemon will wait till this program ends, you never know... And we like to press key combinations many times, if something doesn't work immediately...

Here is my config example:
```
[Generic]
program="/opt/bin/modeSwitcher.sh"
button1=10
button2=11
button3=0
button4=
```

## Known limitations

 * Kodi and X11 are blocking /dev/input/* events. For X11 you can add an exception in /usr/share/X11/xorg.conf.d/10-quirks.conf but Kodi is ... nasty ... As long as they don't implement a nice
unified unput support, my workaround is to revoke the kodi group rights to the input devices. :(
 * If you experience any issues, feel free to use '--loglevel debug' option and check syslog.

## Multiple key combinations
You can have different key sets to run different programs:
```
[Generic]
program="/opt/bin/modeSwitcher.sh"
button1=10
button2=11
button3=0
button4=

[Fun]
program="/opt/bin/haveFun.sh"
button1=0
button2=3
button3=
button4=
```
