# Ubuntu Remove the Shortcuts of Ctrl+Alt+Left and Ctrl+Alt+Right


In Ubuntu, PyCharm conflicts with the gnome desktop shortcuts, which makes key-bindings `Ctrl+Alt+Left` and `Ctrl+Alt+Right` not work.

Use below commands to remove the shortcuts from gnome desktop.

```bash
# List the current shortcuts.
liangr@ubuntu-dev:~$ gsettings list-recursively org.gnome.desktop.wm.keybindings | grep Left
org.gnome.desktop.wm.keybindings move-to-monitor-left ['<Super><Shift>Left']
org.gnome.desktop.wm.keybindings move-to-workspace-left ['<Control><Shift><Alt>Left']
org.gnome.desktop.wm.keybindings switch-to-workspace-left ['<Control><Alt>Left']

liangr@ubuntu-dev:~$ gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-left "[]"
liangr@ubuntu-dev:~$ gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-right "[]"
```
