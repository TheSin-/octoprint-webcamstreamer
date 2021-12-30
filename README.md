# OctoPrint-WebcamStreamer

## Overview 

Plugin that adds a tab to OctoPrint for viewing, starting, and stopping a live stream. 

<img src="https://github.com/adilinden-oss/octoprint-webcamstreamer/raw/master/assets/img/screenshot_tab.png">

**Credits:**

Inspired by and based on the work by 

- Alex Ellis found [here](https://blog.alexellis.io/live-stream-with-docker/)
- jneilliii [OctoPrint-YouTube Live](https://plugins.octoprint.org/plugins/YouTubeLive/) plugin found [here](https://github.com/jneilliii/OctoPrint-YouTubeLive)

## Requirements for Streaming

Although this plugin should work with a wide variety of webcam, it has only been tested with a Raspberry Pi cam running on [OctoPi](https://octoprint.org/download/) and made available via OctoPrint mjpeg-streamer. This plugin relies on a [Docker](https://www.docker.com/) container running [FFmpeg](https://www.ffmpeg.org/) to convert the mjpeg-streamer video stream and pipe it to a live streaming service. The YouTube Live and Twitch.Tv streaming services have been tested.

## Installation

Using `ssh` access the OctoPi and install `docker`:

    curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
    sudo usermod -aG docker pi
    sudo systemctl enable docker
    sudo reboot now
    
2-3 minutes after the reboot, ssh back in and test that docker is working:

    docker run hello-world

You should see something like this.

<img src="https://cdn-images-1.medium.com/max/800/1*QtTDQb8cl_Mv-ZIT8fa9Lw.png">

If not, [try this page](https://medium.freecodecamp.org/the-easy-way-to-set-up-docker-on-a-raspberry-pi-7d24ced073ef) for troubleshooting.

Pull the `kolisko/rpi-ffmpeg` image:

    docker pull kolisko/rpi-ffmpeg:latest

Install OctoPrint-WebcamStreamer via one of these 3 methods, also in-depth explained on the official OctoPrint [Installing a plugin](https://plugins.octoprint.org/help/installation/) page.

1. Open the plugin repository in the Plugin Manager's settings dialog, search for "OctoPrint-WebcamStreamer" and install with the "Install" button.

2. Open the plugin repository in the Plugin Manager's settings dialog, click on "Get more..." and enter the `https://github.com/adilinden-oss/octoprint-webcamstreamer/archive/master.zip` URL in the "... from URL" box. Click the Install button to complete the installation.

3. Access the OctoPi command line and run the `~/oprint/bin/pip install https://github.com/adilinden-oss/octoprint-webcamstreamer/archive/master.zip` command.

## Setup

Pull up **Webcam Streamer** in the OctoPi settings panel

- Enter your streaming providers embed URL for your live stream video into the "Viewer Embed URL" field.
    * YouTube: Your "YouTube Channel ID" is needed to construct the embed URL. It can be found on your [Advanced settings](https://www.youtube.com/account_advanced) page. With the "YouTube Channel ID" in hand, constrcut the URL like this:

        https://www.youtube.com/embed/live_stream?channel=CHANNEL_ID

    * Twitch: Your Twitch username is needed to construct the embed URL. With your username in hand, construct the URL like this:

        https://player.twitch.tv/?channel=USERNAME&parent=FDQN&autoplay=false&muted=true

        NOTE: twitch has made changes to embedding, for it to work you need to use and fdqn that is externally accessible.

- Enter your streaming providers ingest or stream server URL into the "Stream Server URL" field.
    * YoutTube: Your complete URL is created by appending your "Stream name/key" to your "Server URL". Both values can be found on your [Live dashboard](https://www.youtube.com/live_dashboard) page. With both values in hand, the complete "Stream Server URL" should look like this:

        rtmp://a.rtmp.youtube.com/live2/xxxx-xxxx-xxxx-xxxx

    * Twitch: Your complete URL is created by appending your "Primary Stream key" to the `rtmp://live.twitch.tv/app/` URL. The "Primary Stream key" can be found on your [Dashboard Settings](https://www.twitch.tv/dashboard/settings) page. The complete "Stream Server URL" should look like this:

        rtmp://live.twitch.tv/app/live_xxxxxxxxx_xxxxxxxxxx

- Enter your OctoPi webcam URL into the "OctoPi Webcam URL" field. A fully qualified URL is needed containing either the resolvable hostname or the IP address of the OctoPi. The "OctoPi Webcam URL" typically looks something like this:

        http://192.168.10.79:8080/?action=stream

Terse setup information is also availabe via the expandable "Additional Information" section on the "Webcam Streamer" settings page.

<img src="https://github.com/adilinden-oss/octoprint-webcamstreamer/raw/master/assets/img/screenshot_settings.png">

## Advanced Setup

Advanced option allow for modification of the [FFmpeg](https://www.ffmpeg.org/) command line for use with other streaming services or cameras. It also allows for a different [Docker](https://www.docker.com/) container to be specified if desired or needed.

- "Webcam Frame Rate" default is 10 frames per second which is appropriate for the Raspberry Pi webcam.

- "Docker Image" default is `kolisko/rpi-ffmpeg:latest`. This value needs to match the docker image installed in the setup steps.

- "Docker Container" default is `WebStreamer`. This value is rather arbitrary but the default makes sense (to me).

- "FFmpeg Command" allows for customization of the `ffmpeg` command line. Variable substitution is performed to insert setup values into the `ffmpeg` command line.

    Default command line for `ffmpeg` is:

        ffmpeg -re -f mjpeg -framerate 5 -i {webcam_url} {overlay_cmd} -ar 44100 -ac 2 -acodec pcm_s16le -f s16le -ac 2 -i /dev/zero -acodec aac -ab 128k -vcodec h264 -pix_fmt yuv420p -framerate {frame_rate} -g {gop_size} -strict experimental -filter:v {filter} -f flv {stream_url}

    The following variable substitutions are available:

    | FFmpeg Cmd Var   | Settings value
    | -----------------| -------------------------------------------|
    | `{stream_url}`     | Stream Server URL
    | `{webcam_url}`     | OctoPi Webcam URL
    | `{overlay_cmd}`    | FFMPEG switch To Add A Stream Overlay
    | `{frame_rate}`     | OctoPi Webcam Frame Rate
    | `{gop_size}`       | Internal Calculated Value (frame rate * 2)
    | `{filter}`         | Internal Calculated Value   

<img src="https://github.com/adilinden-oss/octoprint-webcamstreamer/raw/master/assets/img/screenshot_settings_advanced.png">

## Todo

- Reset buttons to restore each advanced value with the click of a button
