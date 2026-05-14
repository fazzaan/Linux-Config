## System service script for SysVinit - Devuan, Artix, etc.

gemini: 
> "Tailscale is designed natively for systemd and is not natively packaged for Devuan's default sysvinit init system. To get the tailscaled background service running on Devuan, you must manually create a classic SysV init script, start the daemon, and link it to your system's runlevels."  

### Step 1: Install Tailscale
If you haven't already, install tailscale 
(I forgot how i did this before but this looks fine) 
If you're not on Arch/Artix,  
Fetch the official Tailscale package for Debian (which Devuan is based on) using the standard install script:

```curl -fsSL https://tailscale.com/install.sh | sh```

_Note: The install script might throw an error at the end when attempting to start or enable the tailscaled systemd service, but the core binaries will be installed successfully_  

### Step 2: Create a SysVinit Script
Create a new init.d script for tailscaled using a text editor like nano:  
```sudo nano /etc/init.d/tailscaled```

Paste the following script, which manages the daemon and its environment:  

```
bash#!/bin/sh
### BEGIN INIT INFO
# Provides:          tailscaled
# Required-Start:    $network $remote_fs $syslog
# Required-Stop:     $network $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Tailscale node agent
# Description:       Starts the Tailscale daemon to handle mesh VPN networking
### END INIT INFO

PATH=/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/tailscaled
PIDFILE=/var/run/tailscaled.pid
DESC="Tailscale daemon"
NAME=tailscaled

test -x $DAEMON || exit 0

. /lib/lsb/init-functions

case "$1" in
  start)
    log_daemon_msg "Starting $DESC" "$NAME"
    start-stop-daemon --start --quiet --background --make-pidfile --pidfile $PIDFILE --exec $DAEMON
    log_end_msg $?
    ;;
  stop)
    log_daemon_msg "Stopping $DESC" "$NAME"
    start-stop-daemon --stop --quiet --oknodo --pidfile $PIDFILE
    # Cleanup state
    tailscale down
    log_end_msg $?
    ;;
  restart|force-reload)
    $0 stop
    sleep 1
    $0 start
    ;;
  status)
    status_of_proc -p $PIDFILE $DAEMON $NAME
    ;;
  *)
    echo "Usage: /etc/init.d/$NAME {start|stop|restart|status}"
    exit 1
    ;;
esac

exit 0
```
_N.B. Gemini gave me that, but i believe it was sourced from an attachment on a forum that i don't have direct access to -- [possible source](https://forum.mxlinux.org/download/file.php?id=35304)_  


### Step 3: Set Permissions and Autostart
Once the file is saved, make the script executable and configure it to run automatically on boot:  
```
sudo chmod +x /etc/init.d/tailscaled
sudo update-rc.d tailscaled defaults
```
### Step 4: Start the Service and Log In
Start the newly configured service, and connect the machine to your tailnet:  
```
sudo service tailscaled start
sudo tailscale up
```

Follow the printed URL in your terminal to authenticate and link the Devuan machine to your Tailscale account.  

> [!NOTE]
> On Linux, once you authenticate tailscale up successfully one time, the Tailscale daemon permanently remembers that authenticated state.
> You do not need to execute tailscale up on every boot. Simply ensuring that the tailscaled service starts up automatically via your SysVinit setup will restore your network connectivity instantly.

> [!TIP]
> If you ever need to alter connection parameters automatically (such as enforcing specific exit nodes or routing settings on boot), update your /etc/init.d/tailscaled script to append those parameters directly to the daemon launch, or modify the initial configuration manually.

