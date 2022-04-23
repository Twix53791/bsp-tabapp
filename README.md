# tabapp
Tabapp is a simple script using tabbed into bspwm to launch any application you want in tabs, automatically and without the need of user's keypresses, especially the ones which _cannot_ natively launch into tabbed.

## Why use tabapp?

Others user's scripts are found on github to implement tabbed into bspwm. [`tabc`](https://github.com/Bachhofer/tabc) is a small but efficient script which allows the user to create a tabbed instance, then to add windows to it, one by one, _and also remove them_, which is usefull, for _any_ window manager, not only bspwm ; but all this need user's management. [`bsptab`](https://github.com/albertored11/bsptab) is a very complete script base on the former and adding new functionalities, like a deamon which in fact just launch bspc subscribe node_add in the background to add any new node of a given class into tabbed. All of this is beautiful and you should use it if you like a deep control of your tabbed applications, with the possibility to detach tabs from the tabbed container. Personally, I don't use this functionnality. I just want than specific applications _always_ be launched in tabbed, with a unique tabbed container for each application, like it was a build-in feature. So I wrote this litle script.

**When not use it?** Some applications can be natively launched as child of another window ; for example, [zathura](https://man.archlinux.org/man/zathura.1.en) accepts the -e argument to reparent its window to another. Of course, in this case, it is better to use the build-in feature of the application than tabapp. See tabbed man page for examples for other applications.

## Use

- **If the command is an application name**, it just launch the application into a tabbed instance. You can :
  - pass arguments: `tabapp kitty -e bpytop` will launch bpytop into kitty into tabbed.
  - specify an instance name with `into` ; you can stil pass arguments to the app, after the instance name : `tabapp kitty into toto -e bpytop` will name the tabbed instance 'toto', and launch into it `kitty -e bpytop`.
  - run into an existing tabbed instance if the name specified after `into` is the name of an existing tabbed instance.
  - use the -R flag (cf. below) : tabapp kitty -R -e bpytop

- **What is a 'tabbed instance'?** Tabapp launch by default a tabbed window with an _instance_ name corresponding to the application name. Actually, _is the application reversed_, to avoid to 'grep' by accident the tabbed window when grabbing the windows of an application by its name in others scripts. You can specify an instance name, but it is optional. **When you launch any tabapp command, if a tabbed instance of the name of the application/instance name you specify exist, the command will add the windows to the existing tabbed instance**.
  - `tabapp kitty;tabapp kitty` will launch two kitty tabs into a unique tabbed instance with an instance name 'tabbed.yttik'.

- **Commands, flags**
  `tabapp <command> <flag> <pattern> <instancename> <arguments>` : all after the command is optionnal, except the <pattern>
  - `add <pattern> <instancename>` command will grasp the last node matching the `<pattern>` given to the `<instancename>` tabbed instance, or if not given the `<pattern>` tabbed instance. The pattern can be a node id, or a class name, or even a PID ; anything the `wmctrl -lxp` command can display, as it search into its output (you can change that...).
    - `tabapp add 1564 firefox` will grasp the node id of the (last, if multiple matches) match of "1564" pid, and send its window to an existing/a new tabbed instance named 'tabbed.xofreif'.
  - `gather` command will graps _any_ node matching the pattern, and send it to the given tabbed instance. 
    - `tabapp firefox; tabapp gather kitty firefox` will grasp all the kitty windows in the monitor and send them to the already existing tabbed firefox instance. 
  - `-R` : the -R flag reset any existing instance, so first kill it with all its inside tabs, them create a new one. 
  - `tabapp -R firefox` or `tabapp add -R nomacs kitty`
  - `-f` : by default, tabapp add any tab matching the pattern to the instance, even the focused window. In some situations it is usefull to avoid this behavior. The -f flag ignores the focused window.

## Passing arguments to `tabbed`
It is possible to pass arguments to tabbed, but only with the commands add and gather, not when launching an app, as the arguments will be then passed to the app as showed above (it could be enable easily, if needed). To passed arguments to tabbed, just add them after the command, in the $3 position (the instance name have to be given, so). They will be effective, of course, only when a fresh tabbed instance is launched.
  - `tabapp add kitty -o Red` will add the kitty window matched, if no existing 'kitty' tabbed instance is found, to a new instance of tabbed 'kitty', with a red background color. If an existing 'kitty' instance exist, the arguments will have no effect on tabbed, of course.
  - To pass arguments 'by default', just add them into the tab_opt variable, in the script, as an array: `tab_opt=("-o" "Red")` is the equivalent of the above command. By default, I set the -c option, closing the tabbed instance when the last tab is closed.

## Dependencies
- `wmctrl`
- `xdotool`
- `bspwm`
- `tabbed` can be installed from the [suckless site](https://tools.suckless.org/tabbed/) or from the AUR repository ; it is recommanded to install tabbed-git then. But if you want a patched tabbed, or your arrive to patch it succesfully (which is not my case), or you find a fortunate git hub repo implementing your wanted patches. There is plenty. Personnaly, I use only the hidetabs patch, from [this repo](https://github.com/hXtreme/suckless-tabbed). Just git clone, then, as root is something if wrong, `make clean install` as the read me advise.

## Running the script from everywhere
You can run this script 'manually' from the terminal, but it is intended to be used from other scripts. It can be used in ranger, for example into the rifle config, so rifle will always open a given application tabbed. It an be used into a desktop.file, for xdg-open. 

In ranger, with rifle you can just use, in rifle.conf for example : `mime ^image, X, flag f = tabapp nomacs "$@"` Or a custom script which apply complex rules, sometimes use tabapp, sometimes not : `mime ^image, X, flag f = openrule nomacs "$@"`. I will post soon my own 'openrule' script which automates complex opening rules for all my desktop applications.

It can be used also from bspwm, to open a given application via tabapp and so always tabbed. For that, create a script running in the background, for example launched by bspwmrc at boot, using bspc subscribe node_add. I show here one for libreoffice as its a bit tricky (because libreoffice start with sometimes different class/instance names), moreover, **there is a bug (see below for a fix)** produced when the tabbed instance containing libre office files is killed from outside (bspc node -c, or `kill`, or `xkill`...). It comes from the way libreoffice manage files, with .lock files. If a libreoffice instance is killed from outside it crashes (cf. [this issue](https://ask.libreoffice.org/t/close-libreoffice-gracefuly-from-command-line/34120/4). The best will be to hack the code of tabbed, or making this python script above working, but I am not currently enough skilled to do that.

In your script, that you can start at boot, launching it from bspwmrc
```bash

bspc subscribe node_add | while read -a msg; do
  instance=$(cat /tmp/bsp_win_instance)
  class=$(cat /tmp/bsp_win_class)
  
  if [[ "$instance" == "soffice" ]] || [[ "$instance" == "libreoffice" ]]; then
    tabapp add -f ${msg[4]} libreoffice
  fi
done
```
In bspwm external_rules file:
```
echo $class > /tmp/bsp_win_class
echo $instance > /tmp/bsp_win_instance
```
  - **A more advanced script for libreoffice**. 
  This script will tab libreoffice automatically, but only the windows which are not "dialog windows" (like popup windows, oening files, etc.), and only when a second libreoffice instance is started. So, when you open only one file in libreoffice, libreoffice will not be tabbed. Good for reliability : you still have access to a 'normal' libreoffice, in case of bug. Moreover, with only one window, tabbed is not needed. 
  In a first script:
  ```bspc subscribe node_add | while read -a msg; do /path-to/extended_rules.sh ${msg[4]}; done```
  In a second one, mine called extended_rules:
  ```bash
  node=$1
  class=$(cat /tmp/bsp_win_class)   # From the modified bspwm external_rules file, see above
  
  libreoffice(){
   dialog=$(xprop -id $1 | grep "WINDOW_TYPE_DIALOG")   # If the window is a dialog window, don't tab it
   if [[ -z $dialog ]]; then
      togrep="Soffice\|libreoffice\|eciffoerbil"        # It will tab libreoffice only when a second instance is opened
      isrunning=$(wmctrl -lx | cut -d' ' -f-4 | grep "$togrep" | wc -l)
      [[ $isrunning -gt 1 ]] && tabapp add $1 libreoffice
   fi
  }

  tabbed_libreoffice(){
    tabapp gather libreoffice libreoffice
  }
  
  instance=$(cat /tmp/bsp_win_instance)
     if [[ $instance == libreoffice ]]; then
      libreoffice $1
   elif [[ $class == Soffice ]]; then
      libreoffice $1
   elif [[ $instance == tabbed.eciffoerbil ]]; then
      tabbed_libreoffice $1
   fi
   ```
   
  - **Libreoffice-tabbed bug fix**. It is possible to bypass in a bit questionable manner this bug sending a 'ctrl+q' key press to tabbed instead of kill/close the all window. Link this script to a key press and you will close each tab one by one when closing tabbed. If libreoffice (but we could extend to other applications, if needed) open a popup window to ask for saving/discarding changes in the file, it waits until the user answers and so forth. It is not completely reliable, that's why it is good to have to possibility to run libre office outside tabbed, so if it crash, you can access it again normally (and it will recover the files corrupted).

```bash
#!/bin/bash

id=$(wmctrl -lx | cut -d' ' -f-4 | grep -i $(bspc query -N -n focused))
tabbed=$(echo $id | grep "tabbed")

getchildren(){
   xwininfo -id ${id%%' '*} -children | \
   sed -n '/[0-9]\+ \(child\|children\):/,$s/ \+\(0x[0-9a-z]\+\).*/\1/p'
}

if [[ ! -z $tabbed ]]; then
   tabs=$(getchildren)
   for child in $tabs; do
      saved=$(wmctrl -lx | cut -d' ' -f-4 | grep "soffice.Soffice")
      until [[ -z $saved ]];do
         saved=$(bspc query -N -n ${saved%%' '*})
         sleep .2
      done
      xdotool key --window ${id%%' '*}  ctrl+q
      sleep .15
   done
 fi
 ```

## Install
Just put this script into /usr/local/bin or /usr/bin, and use it as a shell command.

## Possible issues
For sure, I will not garantee the stability of this script! I still have to test it on the long run and for sure it can create problems with some specific applications. Also, for now, it creates artifacts when moving floating windows to a tabbed instance, with the gather or add command, at the former place of the window sent, if not other window occupy the new empty place. It disappear if we 'recover' them with a window, for example setting any window to fullscreen. No problems with tiled windows. I am gonna try to fix that. Also, maybe than, when launching an app with tabapp, if this app is not actually launched for any reason, the script will still grab the first next node created to a tabbed instance - another small fix I have to do. With some applications, it could be quite risky, like with libreoffice : if we kill the tabbed window without before kill each libreoffice tab, it seems to break libreoffice which doesn't open some files anymore.
