[i3](http://i3wm.org/) the X window manager, organizes the screen into multiple workspaces. I allocate each workspace to specific apps with similar functionality. For eg., my workspace 2 (named as 2: web) is used for browsing online web. I either use chromium or firefox (iceweasel) to browse, so both apps are configured to show up in workspace "2: web" via the assign command in ~/.i3/config
```
assign [class="(?i)chromium"] 2: web
assign [class="(?i)iceweasel"] 2: web

for_window [class="(?i)chromium"] workspace number 2
for_window [class="(?i)iceweasel"] workspace number 2
```

For convenience, once the browser launches, it will switch to workspace 2. This is configured via the for_window command. 

----

i3-ws-monitor script make use of i3status, which generates a status line for i3bar.  The interval directive (in i3status config) specifies the time in seconds for which i3status will sleep before printing the next status line. Running i3-ws-monitor as a replacement for i3status, the script checks which is the current visible workspace and then increment the current workspace usage time by interval seconds. (It makes the assumption that for the past interval seconds, the screen is in the current workspace) 

The script outputs i3status status line together with the current workspace usage time (in > h:mm:ss format) inserted in front. The usage time is displayed in color (variable  warnStatusCol) if the workspace usage time is within some time (variable  warnStatusInSecs) of the workspace configured maximum time in mins (variable warnifWSMins) or the total time spent is within warnStatusInSecs of the configured maximum time in mins (variable warnifMins). In the case when the total time spent is closer to its maximum, the status line will have a prefix $ (eg $> 2:59:00 ). 

i3-ws-monitor will give a warning message when either the total time spent has reached or exceeded the configured maximum time in mins (variable warnifMins), or the time spent in current workspace has reached or exceeded the configured maximum time for the workspace in mins (variable warnifWSMins). Beside showing the warning message, it will switch to a configured unused workspace. (variable ignoreWS) This workspace should not be used by the user, for the script does not compute usage time in this workspace. And, it will display the total time spent in the status. 

Usage time is saved periodically (variable saveMins) to a configured existing directory (variable savetoDir) in json format. 

i3-ws-monitor.config file is a file that defines the various configured variables, is simply sourced by the script i3-ws-monitor.   
The "declare -A warnifWSMins" declares a bash associative array.   warnifWSMins define the maximum time for the workspaces, where each key is the workspace name, and the value is maximum time for the workspace in mins. 

[i3-ws-monitor.config](bin/i3-ws-monitor.config)
```bash
#  i3status output interval = 5 s (as in ~/.config/i3status/config)
interval=5
#  gives warning if total time spent >= ? mins
warnifMins=180
#  saves time spent every ? mins
saveMins=30
#  save to dir
savetoDir="$HOME/personal/times"
#  ignores workspace 10 (unused workspace, also for showing warning)
ignoreWS=10
#  gives warning if time spent in workspace >= ? mins
declare -A warnifWSMins
warnifWSMins=([2: web]=60 [9: ebook]=45)
#  warning status color
warnStatusCol="#FF0000"
#  warning status if within ? secs of max time (either workspace or total)
warnStatusInSecs=60
```

To replace i3status with i3-ws-monitor (assume script & config are within PATH), the below is modified in ~/.i3/config 
```
bar { 
#        status_command i3status 
        status_command exec ~/bin/i3-ws-monitor
} 
```

---- 

i3-ws-monitor script also make use of:  
 * [jq](https://stedolan.github.io/jq/), a command-line JSON processor, to read back the saved usage data; and to filter and read the workspaces data from i3.
 * [notify-send](https://wiki.archlinux.org/index.php/Desktop_notifications) - a command-line program to send desktop notifications. 

This is based on my earlier blog post: [Monitoring usage time in i3](http://www.timelesssky.com/blog/monitoring-usage-time-in-i3).
