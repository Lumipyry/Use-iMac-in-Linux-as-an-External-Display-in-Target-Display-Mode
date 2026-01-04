[<img width="510" height="340" alt="target-display-mode3" src="https://github.com/user-attachments/assets/eef72173-3e8b-44b0-b0c7-486198486a29" />](https://cdsassets.apple.com/live/7WUAS350/images/imac/imac-target-display-mode-hero.jpg)

######


This is a step-by-step instruction for [smc_util](https://github.com/floe/smc_util) including [Powerbutton support](https://github.com/floe/smc_util/pulls) by [FreekMank](https://github.com/FreekMank).

Target Display Mode is set On and Off by pressing Powerbutton.

Applicable with iMac models [2009 - 2014](https://support.apple.com/en-us/105126). Needs to have Apple High Sierra or earlier (Dual boot with Linux) in the machine which is used as external display.

Proven to work with MiniDisplay Port cable. No personal experiences with Thunderbolt cable (although Apple Support page says this needs Thunderbolt cable - keep in mind that iMacs 2009-2010 do not support Thunderbolt).

The Off may not always work (but long pressing of power button naturally shuts down the iMac).

This text is also in https://superuser.com/questions/1932625/use-imac-in-linux-as-an-external-display-in-target-display-mode/1932626#1932626

***

**NOTE**: You may want to leave `rc.local` out of the installation (`Step 16`) - if and when you want also use the Linux OS of the display machine. (`rc.local` makes the machine boot directly to Target Display Mode)

Without `rc.local` the machine boots first to Linux OS and pressing Powerbutton sets TDM on.

***

1.Download smc_util to home directory (`in the intended display iMac`)
```
git clone https://github.com/floe/smc_util.git
```
2.Move to smc_util
```
cd smc_util
```
3.Compile SmcDumpkey with GCC
```
gcc -O2 -o SmcDumpKey SmcDumpKey.c -Wall
```
4.Create files tdm_toggle.sh, powerbutton, powerbutton.sh and rc.local
```
touch tdm_toggle.sh powerbutton powerbutton.sh rc.local
```
5.Change content of file tdm_off.sh
```
#!/bin/bash
rm -f tdm_started
./SmcDumpKey MVHR 0
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --auto
```
6.Change content of file tdm_on.sh
```
#!/bin/bash
./SmcDumpKey MVHR 1
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --off
touch tdm_started
```
7.Create content to file tdm_toggle.sh
```
#!/bin/bash
pushd $(dirname "${BASH_SOURCE[0]}")

if [[ -f "tdm_started" ]]; then
  echo "Switching off TDM"
  source tdm_off.sh
else
  echo "Switch on TDM"
  source tdm_on.sh
fi

popd
```
8.Create content to file powerbutton
```
event=button/power PBTN
action=/etc/acpi/powerbutton.sh
```
9.Create content to file powerbutton.sh. `REMEMBER TO CHANGE XXXXXXXXX to your username (in TWO LINES in the script)`
```
#!/bin/bash

pushd $(dirname "${BASH_SOURCE[0]}")

FILE=powerbutton_pressed
NOW=$(date +%s)

if [[ -f "$FILE" ]]; then
  # Read timestamp of previous powerbutton press from file
  echo "File exists"
  typeset -i PREV=$(cat $FILE)
  echo Compare $NOW and $PREV

  # if two powerbutton presses were <1 seconds apart -> shutdown
  if [[ $(($NOW-$PREV)) -lt 2 ]]; then
    # Shutdown
    echo "Powerbutton pressed twice in a row: Shutting down"
    shutdown now
  else
    echo "Toggle TDM"
    /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
  fi
else
  echo "Toggle TDM"
  /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
fi

echo $NOW > $FILE

popd
```
10.Create content to file rc.local. `REMEMBER to change XXXXXXXXX to your username`
```
#!/bin/bash

# Start Target Display Mode such that the pc is used as external monitor right away
# Note: The Power button toggles TDM (see /etc/acpi/events)
pushd /home/XXXXXXXXX/smc_util
./tdm_on.sh
popd
```
11.Remove the SMC kernel driver to avoid conflicts
```
sudo rmmod applesmc
```
12.Make tdm_toggle.sh, rc.local and powerbutton.sh executable (`change XXXXXXXX`)
```
sudo chmod +x /home/XXXXXXXXX/smc_util/tdm_toggle.sh /home/XXXXXXXXX/smc_util/rc.local /home/XXXXXXXXX/smc_util/powerbutton.sh
```
13.Install build-essential and acpid
```
sudo apt install build-essential acpid acpid-suppport
```
14. Copy file powerbutton to /etc/acpi/events (`change XXXXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton /etc/acpi/events
```
15.Copy file powerbutton.sh to /etc/acpi (`change XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton.sh /etc/acpi
```
16.Copy file rc.local to /etc (`change XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/rc.local /etc/rc.local
```
17.Restart acpid
```
sudo systemctl restart acpid
```
18.Change the Powerbutton behaviour in Linux OS to
```
Do Nothing
```
