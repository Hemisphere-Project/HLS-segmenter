# HLS-segmenter
HLS Segmenter is a configurable python script to achieve Http Live Streaming segmentation, playlist and variants playlist from a given movie file.

HLS Segmenter use ffmpeg, and works great on Linux.

## Segmenter

DEPENDENCIES: FFmpeg

* Tested with FFmpeg 2.5.8 on Ubuntu 15.04 
* Tested with FFmpeg 2.6.4 on Debian 8 (http://www.deb-multimedia.org/)


CONFIGURE:

You can create "Profiles" in presets.py (see existing ones to get an idea).

USE:

./segmenter -i *video_path* [-u *url_prefix*] [-p]

* -i *video_path* indicate the path to the input video file
  * it can be either relative (to the script execution path) or absolute.
* -u *url_prefix* allow to prepend an url to the playlists path, i.e. : **http://yourserver/videofolder/**
  * This base url should reach the input video folder location.
* -p only regenerate the variant playlist but do not re-encode the chunks (dryrun)

Let say your input media is /var/www/video/**mymovie.mp4**:
And that you have a webserver with http://localhost/ linked to /var/www/

The command will be

./segmenter -i /var/www/video/mymovie.mp4 -u http://localhost/video/

It will create a new directory /var/www/video/**mymovie**/ next to the input video location, with
* a re-encoded version of the intput movie, using the first profile (can be use as a fallback for players without HLS support) also named /var/www/video/**mymovie**/**mymovie.mp4**
* the main variant playlist file .m3u8, which link to the profile's playlists named /var/www/video/**mymovie**/**mymovie.m3u8**
* subdirectories for each quality profile, hosting the chunks and the associated playlist.

To play the HLS stream you only need to provide the movie.m3u8 url, like this http://localhost/video/**mymovie**/**mymovie.m3u8**

And if your client doesn't support HLS you can fallback to
http://localhost/video/**mymovie**/**mymovie.mp4** 
which is a re-encode of your original movie with the first preset in the list.

Your original movie should still be available from there:
http://localhost/video/**mymovie.mp4**

## TODO

### Segmenter
* Add more options to the segmenter command line:
  * Output path
  * Segments size (it can be defined in presets.py at SEGMENT_SIZE)
  * FFmpeg preset for h264 encoding (it can be defined in presets.py at FFMPEG_PRESET)
  * Ability to choose specific profiles
* Reorganize the Presets / Profiles file
* Auto disable profiles with higher bitrate/resolution than the original media
* validate HLS flux compatibility (using Apple provided tools)
  * https://developer.apple.com/library/ios/technotes/tn2235/_index.html
  * https://developer.apple.com/streaming/ (Download section)
* check if I-Frame can be optimized (at the beggining)
  * monitor: http://stackoverflow.com/questions/14005110/how-to-split-a-video-using-ffmpeg-so-that-each-chunk-starts-with-a-key-frame/14011638#14011638
  * After segementation: http://video.stackexchange.com/questions/4904/how-to-force-ffmpeg-to-insert-keyframe-at-first-frame-when-downsampling-a-framer
  * alternative solution to segment: https://www.ffmpeg.org/ffmpeg-formats.html#Examples-5

### HLS Dealer
* Create a small HTTP file server to distribute the M3U8 playlists and chunks
* Proper MIME Type

## RESSOURCES

### EXEMPLE MAIN VARIANT M3U8 PLAYLIST
command: ./segmenter -i /var/www/myvideo.mp4 -u http://localhost/
playlist url: http://localhost/myvideo/myvideo.m3u8
```
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=244400,RESOLUTION=416x234
http://localhost/myvideo/0-ugly/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=421120,RESOLUTION=416x234
http://localhost/myvideo/1-bad/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=733200,RESOLUTION=480x270
http://localhost/myvideo/2-tiny/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=1182145,RESOLUTION=640x360
http://localhost/myvideo/3-low/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2192834,RESOLUTION=640x360
http://localhost/myvideo/4-medium/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=5888165,RESOLUTION=960x540
http://localhost/myvideo/5-high/stream.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=8663048,RESOLUTION=1280x720
http://localhost/myvideo/6-hd/stream.m3u8
```

### BITRATE EVALUATION

FFmpeg can report the bitrate of the ts stream. Use the ffprobe tool and you'll get output like this:

ffprobe ./myvideo/3-low/3-low3.ts (the third chunk for the "low" profile)
```
Input #0, mpegts, from '3-low3.ts':
  Duration: 00:00:02.00, start: 7.423222, bitrate: 851 kb/s
  Program 1 
    Metadata:
      service_name    : Service01
      service_provider: FFmpeg
    Stream #0:0[0x100]: Video: h264 (Constrained Baseline) ([27][0][0][0] / 0x001B), yuv420p, 640x360 [SAR 1:1 DAR 16:9], 24 fps, 24 tbr, 90k tbn, 48 tbc
    Stream #0:1[0x101]: Audio: aac (LC) ([15][0][0][0] / 0x000F), 44100 Hz, stereo, fltp, 93 kb/s
```
The bitrate is being given in kilobits per second, so multiply it by 1024 and you'll have the value you need for the BANDWIDTH tag.
