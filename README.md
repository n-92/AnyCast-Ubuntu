# AnyCast-Ubuntu
This repository contains all the scripts required to stream audio/video to a Anycast dongle.

![alt text](https://m.media-amazon.com/images/I/61wz5O38F0L._AC_SL1500_.jpg)

# Getting started
1. You need to be on X11, not Wayland. You can check by executing `echo $XDG_SESSION_TYPE`
2. You might need to add the required permission to xhost to allow access to the built in display. `xhost +SI:localuser:<Your User ID>`
3. You would need ffmpeg installed. My version is `ffmpeg version 4.4.2-0ubuntu0.22.04.1`
4. To test if ffmpeg can capture your screen, use the following command. `ffmpeg -f x11grab -video_size 640x360 -framerate 25 -i :0.0 -t 10 -c:v libx264 -pix_fmt yuv420p output.mp4`

# Using the scripts. 
In one terminal screen, you can execute the following commands depending on what you want to stream. 

## Stream video and audio from mic
`ffmpeg -thread_queue_size 512 -f x11grab -video_size 1280x720 -framerate 30 -i :0.0 \
-thread_queue_size 512 -f pulse -i default \
-c:v libx264 -preset ultrafast -tune zerolatency -pix_fmt yuv420p -b:v 800k \
-c:a aac -b:a 128k \
-f mpegts -listen 1 http://0.0.0.0:8090/streamvideo`


## To mute the audio on your workstation
`pactl set-sink-mute @DEFAULT_SINK@ 1`

## To unmute the audio 
`pactl set-sink-mute @DEFAULT_SINK@ 0`

## Stream both video and audio from computer
`ffmpeg -fflags +genpts -use_wallclock_as_timestamps 1 \
-thread_queue_size 512 -f x11grab -video_size 1280x720 -framerate 25 -i :0.0 \
-thread_queue_size 512 -f pulse -i alsa_output.pci-0000_00_1b.0.analog-stereo.monitor \
-c:v libx264 -preset ultrafast -tune zerolatency -pix_fmt yuv420p -b:v 1000k \
-c:a aac -b:a 128k \
-f mpegts -listen 1 http://0.0.0.0:8090/streamvideo`


## Stream only audio  with best possible settings. 
`ffmpeg \
-f pulse -i alsa_output.pci-0000_00_1b.0.analog-stereo.monitor \
-c:a aac -b:a 320k -ar 48000 -ac 2 -profile:a aac_low \
-f mpegts -listen 1 http://0.0.0.0:8090/streamvideo`

## Stream computer audio and mic
`ffmpeg \
-f pulse -i alsa_output.pci-0000_00_1b.0.analog-stereo.monitor \
-f pulse -i alsa_input.pci-0000_00_1b.0.analog-stereo \
-filter_complex "[0:a][1:a]amix=inputs=2:duration=longest:dropout_transition=2[aout]" \
-map "[aout]" \
-c:a aac -b:a 320k -ar 48000 -ac 2 -profile:a aac_low \
-f mpegts -listen 1 http://0.0.0.0:8090/streamvideo`

## Once you have any of the streaming commands running via ffmpeg, you can run the python scripts on another terminal. There is a delay of about 4-7 seconds for the video streaming to the dongle. Also, you might need to add the port to your firewall rules using a tool like ufw. For my own testing, I simply disabled it using `ufw disable`
