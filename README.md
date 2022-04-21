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

## Passing arguments to `tabbed`
It is possible to pass arguments to tabbed, but only with the commands add, gather and new, not when launching an app, as the arguments will be then passed to the app as showed above. To passed arguments to tabbed, just add them after the command, in the $3 position (the instance name have to be given, so). They will be effective, of course, only when a fresh tabbed instance is launched.
  - `tabapp add kitty kitty -o Red` will launch, if no existing 'kitty' tabbed instance is found, a new instance of tabbed kitty, with a red background color.
- To pass arguments 'by default', just add them into the tab_opt variable, in the script, as an array: `tab_opt=("-o" "Red")` is the equivalent of the above command.

## Dependencies
- `wmctrl`
- `xdotool`
- `tabbed` can be installed from the [suckless site](https://tools.suckless.org/tabbed/) or from the AUR repository ; it is recommanded to install tabbed-git then.
- `bspwm`

## Running the script from everywhere
You can run this script 'manually' from the terminal, but it is intended to be used from other scripts. It can be used in ranger, for example into the rifle config, so rifle will always open a given application tabbed. It an be used into a desktop.file, for xdg-open. It can be used also from bspwm, to open a given application via tabapp and so always tabbed. For that, create a script running in the background, for example launched by bspwmrc at boot, using bspc subscribe node_add:


## Install
Just put this script into /usr/local/bin or /usr/bin, and use it as a shell command.