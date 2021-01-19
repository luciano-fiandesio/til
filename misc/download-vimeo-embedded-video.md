# Download Vimeo embedded videos


[youtube-dl](https://youtube-dl.org/) can't download a Vimeo embedded video directly.
In order to capture the download link, follow these steps:

- open the browser's dev tools and select the network tab
- start the target video and stop it immediately 
- in the dev tools network tab, filter for an url that contains `.json?base64_init=1`. That is the url video stream
- copy it and replace ``.json?base64_init=1`` with `.mpd`
- fire up youtube-dl and download the video:

```
youtube-dl -o embedded.mp4 "https://162vod-adaptive.akamaized.net/my-embedded-video.mp4"
```
