# Nomachine Setup


## Setup Steps

1. On CentOS8 machine:
   1. `wget https://download.nomachine.com/download/6.12/Linux/nomachine_6.12.3_7_x86_64.rpm`
   2. `sudo dnf install nomachine_6.12.3_7_x86_64.rpm`
2. On Windows machine:
   1. Download nomachine exe installation file from official site: https://www.nomachine.com/download.
   2. Install it.
3. Enable 3440x1440 resolution on CentOS8:
   1. `gtf 3440 1440 60`

        Sample output: 
        ```console
          # 3440x1440 @ 60.00 Hz (GTF) hsync: 89.40 kHz; pclk: 419.11 MHz
        Modeline "3440x1440_60.00"  419.11  3440 3688 4064 4688  1440 1441 1444 1490  -HSync +Vsync
        ```
    2. Copy above output, then run `xrandr --newmode "3440x1440_60.00"  419.11  3440 3688 4064 4688  1440 1441 1444 1490  -HSync +Vsync`
    3. List the display:

        ```console
        $ xrandr --listmonitors
        Monitors: 1
         0: +*VNC-0 1920/508x1200/317+0+0  VNC-0
        ```
    4. `xrandr --addmode VNC-0 "3440x1440_60.00"`
4. You can select 3440x1440 resolution in display setting now.
