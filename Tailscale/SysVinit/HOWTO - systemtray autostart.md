## Add the Tailscale System Tray to Graphical Login

_Gemini helped me with this_  

> Tailscale includes a built-in cross-platform system tray feature. In a desktop environment managed without systemd (like Devuan), you can launch this visual tray immediately after logging into your desktop session by creating an XDG autostart entry.  

### Step 0. Run the command
The Tailscale binary has an autostarter built in.  
Just run this:  
```tailscale configure systray --enable-startup=freedesktop```

**If that doesn't work, then follow these steps instead:**

### Step 1. Create the Systray Desktop Entry
Create a text file targeted for desktop startup:
```nano ~/.config/autostart/tailscale-systray.desktop```

### Step 2: Populate the Configuration
Paste the following desktop configuration block into the `tailscale-systray.desktop` file and save it:
```
[Desktop Entry]
Type=Application
Version=1.0
Name=Tailscale System Tray
Comment=Tailscale system tray applet for managing Tailscale
Exec=/usr/bin/tailscale systray
TryExec=/usr/bin/tailscale
Terminal=false
NoDisplay=true
StartupNotify=false
Icon=tailscale
Categories=Network;System;
X-GNOME-Autostart-enabled=true
X-Desktop-File-Install-Version=0.28
```
That is the text file that was generated when I ran the first command in **Step 0**.  

> [!NOTE]
> Gemini told me that it's better to have `tailscale configure systray --enable-startup=freedesktop` inside the .desktop file, cos then it automatically handles the desktop environment each time you log in, but idk, this doesn't make sense to me because that command is the command that creates the .desktop file anyway, and it _isn't_ the command that launched the tailscale systray. So idk 🤷  


### Step 3. Get it running
You can log out and log back in to see it work.  
Or if you just want the systray to run now, run:  
```tailscale systray &```
