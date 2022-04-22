# tabapp
tabapp is a simple script using tabbed into bspwm to launch any application you want in tabs, automatically and without the need of user's keypresses

## Why use tabapp?

Others user's scripts are found on github to implement tabbed into bspwm. [`tabc`](https://github.com/Bachhofer/tabc) is a small but efficient script which allows user to create a tabbed instance, then to add windows to it, _and also remove them_, which is usefull, for _any_ window manager, not only bspwm ; but all this need user's management. [`bsptab`](https://github.com/albertored11/bsptab) is a very complete script base on the former and adding new functionalities, like a deamon which in fact just launch bspc subscribe node_add in the background to add any new node of a given class into tabbed. All of this is beautiful and you should use it if you like a deep control of your tabbed applications, with the possibility to detach tabs from the tabbed container. Personally, I don't use this functionnality. I just want than specific applications _always_ be launched in tabbed. So I wrote this litle script.

## Use

`tabapp <command> <arguments>`

- **If the command is an application name**, it just launch the application into a tabbed instance. You can pass arguments:
  - `tabapp kitty -e bpytop` will launch bpytop into kitty into tabbed
  Or you can specify a specific tabbed instance where to launch the application:
  - `tabapp firefox into kitty` will launch firefox into the 'kitty' tabbed instance. The syntax is: `tabapp <appname> into <instancename>. Then you can't pass arguments to the application (but you could easily change that...).
- **What is a 'tabbed instance'?** Tabapp launch by default a tabbed window with an _instance_ name corresponding to the application name. Actually, is the application reversed, to avoid to 'grep' by accident the tabbed window when grabbing the windows of an application by its name in others scripts. You can specify an instance name, but it is optional. **When you launch any tabapp command, if a tabbed instance of the name of the application/instance name you specify exist, the command will add the windows to the existing tabbed instance**.
  - `tabapp kitty;tabapp kitty` will launch two kitty tabs into a unique tabbed instance with an instance name 'tabbed.yttik'.
- `add <pattern> <instancename>` command will grasp the last node matching the `<pattern>` given to the `<instancename>` tabbed instance, or if not given the `<pattern>` tabbed instance. The pattern can be a node id, or a class name, or even a PID ; anything the wmctrl command can display, as it search into its output (you can change that...).
  - `tabapp add firefox` will grasp the node id of the (last, if multiple matches) match of "1564" pid into the output of `wmctrl -lx`, and send this window to an existing/a new tabbed instance named 'tabbed.xofreif'.
  - `tabbed add 1564 kitty` will send the application of the pid 1564 to the kitty tabbed instance.
- `gather <command> <instancename>` command will graps _any_ node matching the pattern, and send it to the given tabbed instance.
  - `tabapp firefox; tabapp gather kitty firefox` will grasp all the kitty windows in the monitor and send them to the already existing tabbed firefox instance. 
- `new <instancename>` : it is also possible to create an empty tabbed instance of a given name ; then the instance name is obligatory.
- The `-R` flag has to be use just after the command, or before the application name if no command is given. If an existing tabbed instance of application/instance name given exist, it will kill it and restart a fresh one. It 'resets' the instance. 
  - `tabapp -R firefox` or `tabapp add -R nomacs kitty`
- Normally, tabapp does not allow to move the focused node into tabbed. In some situations it is usefull to bypass this behavior and forcing the tabbing by passing the `-f` argument after the command, as -R. -R and -f can't be currently combinated.

## Passing arguments to `tabbed`
It is possible to pass arguments to tabbed, but only with the commands add, gather and new, not when launching an app, as the arguments will be then passed to the app as showed above. To passed arguments to tabbed, just add them after the command, in the $3 position (the instance name have to be given, so). They will be effective, of course, only when a fresh tabbed instance is launched.
  - `tabapp add kitty kitty -o Red` will launch, if no existing 'kitty' tabbed instance is found, a new instance of tabbed kitty, with a red background color.
- To pass arguments 'by default', just add them into the tab_opt variable, in the script, as an array: `tab_opt=("-o" "Red")` is the equivalent of the above command. By default, I set the -c option, closing the tabbed instance when the last tab is closed.

## Dependencies
- `wmctrl`
- `xdotool`
- `tabbed` can be installed from the [suckless site](https://tools.suckless.org/tabbed/) or from the AUR repository ; it is recommanded to install tabbed-git then.
- `bspwm`

## Running the script from everywhere
You can run this script 'manually' from the terminal, but it is intended to be used from other scripts. It can be used in ranger, for example into the rifle config, so rifle will always open a given application tabbed. It an be used into a desktop.file, for xdg-open. It can be used also from bspwm, to open a given application via tabapp and so always tabbed. For that, create a script running in the background, for example launched by bspwmrc at boot, using bspc subscribe node_add. I show here one for libreoffice as its a bit tricky (because libreoffice start with sometimes different class/instance names), even **I not recommand to use tabapp for now for libreoffice, because of a bug**. It comes from the way libreoffice manage files, with .lock files. If a libreoffice instance is killed via `kill` or `xkill`, it crashes (cf. [this issue](https://ask.libreoffice.org/t/close-libreoffice-gracefuly-from-command-line/34120/4). The fix involves a bit of complex coding...

In your script, that you can start at boot, launching it from bspwmrc
```bash

bspc subscribe node_add | while read -a msg; do
  instance=$(cat /tmp/bsp_win_instance)
  class=$(cat /tmp/bsp_win_class)
  
  if [[ "$instance" == "soffice" ]] || [[ "$instance" == "libreoffice" ]]; then
    wid=$(wmctrl -lx | cut -d' ' -f-4 | grep "soffice\|libreoffice\|eciffoerbil" | cut -d' ' -f1)
    n=$(echo $wid  | tr ' ' '\n' | wc -l)
    if [[ $n -gt 1 ]]; then
      sleep .2;tabapp gather -f libreoffice
    fi
  fi
done
```
In bspwm external_rules file:
```
echo $class > /tmp/bsp_win_class
echo $instance > /tmp/bsp_win_instance
```

**In ranger, with rifle** you can just use, in rifle.conf for example : ```mime ^image, X, flag f = tabapp nomacs "$@"```
Or a custom script which apply complex rules, sometimes use tabapp, sometimes not : ```mime ^image, X, flag f = openrule nomacs "$@"```
I will post soon my own 'openrule' script which automate complex opening rules for all my desktop applications.

## Install
Just put this script into /usr/local/bin or /usr/bin, and use it as a shell command.

## Possible issues
For sure, I will not garantee the stability of this script! I still have to test it on the long run and for sure it can create problems with some specific applications. Also, for now, it creates artifacts when moving floating windows to a tabbed instance, with the gather or add command, at the former place of the window sent, if not other window occupy the new empty place. It disappear if we 'recover' them with a window, for example setting any window to fullscreen. No problems with tiled windows. I am gonna try to fix that. Also, maybe than, when launching an app with tabapp, if this app is not actually launched for any reason, the script will still grab the first next node created to a tabbed instance - another small fix I have to do. With some applications, it could be quite risky, like with libreoffice : if we kill the tabbed window without before kill each libreoffice tab, it seems to break libreoffice which doesn't open some files anymore.
