# nginx-rtmp-ffmpeg

An nginx-rtmp container for encoding streams with ffmpeg, and pushing to other streaming servers, e.g. twitch.tv

This is useful if you would like to offload the CPU cost of expensive video compression,
allowing you to stream relatively uncompressed video at high bitrates on a fast, local network,
and have a different PC encode and publish the stream to the remote server.

## required environment variables

- `STREAM_KEY`: the stream key required by your streaming service, e.g. `live_x01234567890123456789x`
- `BITRATE`: the bitrate to output (ensure your internet upstream can handle this value), e.g. `3500k`
- `RESOLUTION` the resolution to output, e.g. `1280x720`
- `PRESET` the x264 preset to encode with, e.g. `veryfast`
- `PROFILE` the x264 profile to use, e.g. `high`
- `LEVEL` the [x264 level](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Levels) to use, e.g. `4.1`
- `FRAMERATE` the framerate to output, e.g. `60`
- `THREADS` the number of CPU threads to use for encoding, e.g. `4`
- `INGEST` the streaming service ingest server, e.g. `rtmp://live-jfk.twitch.tv/app`

You can see a list of Twitch ingest endpoints at [bashtech.net](https://bashtech.net/twitch/ingest.php)

## standalone example

```
docker run --rm -it -p 1935:1935 -e STREAM_KEY=live_x01234567890123456789x \
   -e BITRATE=3500k -e RESOLUTION=1280x720 -e PRESET=veryfast -e FRAMERATE=60 -e THREADS=4 \
   -e PROFILE=high -e LEVEL=4.1 -e INGEST=rtmp://live-jfk.twitch.tv/app shamelesscookie/nginx-rtmp-ffmpeg:latest
```

## docker-compose example

```
version: '2'
services:
  nginx-rtmp-ffmpeg:
    container_name: nginx-rtmp-ffmpeg
    image: shamelesscookie/nginx-rtmp-ffmpeg:latest
    network_mode: host
    ports:
      - 1935
    restart: always
    environment:
      - STREAM_KEY=live_x01234567890123456789x
      - BITRATE=3500k
      - RESOLUTION=1280x720
      - PRESET=veryfast
      - FRAMERATE=60
      - PROFILE=high
      - LEVEL=4.1
      - THREADS=4
      - INGEST=rtmp://live-jfk.twitch.tv/app
```

## source PC settings

Configure your streaming software to output to `rtmp://<docker-server>:1935/livein`
Make sure you include your stream key in your streaming software as well, e.g. for OBS:

- File > Settings > Stream
- Stream Type: `Custom Streaming Server`
- URL: `rtmp://<docker-server>:1935/livein`, e.g. `rtmp://mydockerbox:1935/livein`
- Stream key: `<your-stream-key>`

Then use video settings that are very high in quality and low in overhead, e.g. for OBS and nVidia video cards:

- File > Settings > Output > Streaming
- Encoder: `NVENC H.264`
- Rate Control: `CBR`
- Bitrate: `35000`, or some other very high bitrate for local stream
- Keyframe Interval: `0`
- Preset: `Low-Latency High Quality` or `High Quality`
- Profile: `main` or `high`
