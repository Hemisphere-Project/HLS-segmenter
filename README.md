# HLS-server
Linux HLS Server including uploader, segmenter, chunks dealer and media manager

This project is in early development status.
For now, only the Segmenter - based on FFMPEG - is available.

## Segmenter

DEPENDENCIES: FFmpeg

* Tested with FFmpeg 2.5.8 on Ubuntu 15.04 
* Tested with FFmpeg 2.6.4 on Debian 8 (http://www.deb-multimedia.org/)


USE:

./segmenter -i *video_path* [-u *url_prefix*] [-p]

* -i *path* indicate the path to the input video file. it can be either relative or absolute.
* -u *url* allow to prepend an url to the variants playlists path, i.e. : http://yourserver/
* -p only regenerate the variant playlist but do not re-encode the chunks (dryrun)

It will create a directory near the input video location, with
* subdirectories for each quality profile, hosting the chunks
* the m3u8 playlist files

## TODO

### Segmenter
* Add more options to the segmenter command line:
  * Output path
  * Segments size (now it's the 10s default)
  * FFmpeg preset for h264 encoding (now it's the 'veryslow' default)
  * Ability to choose profiles
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

### HLS Uploader
* WebApp which can receive file upload, check for validity and push it to the segmenter

### HLS Media Manager
* WebUX to manage uploaded files, control segmentation validity and give info on available streams

## RESSOURCES

### EXEMPLE M3U8
```
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=512376
500/500k_512x384_x264_372_quicktime_128.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=68795
64/64k_256x192_x264_32_quicktime_32.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=155580
150/150k_256x192_x264_118_quicktime_32.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=308895
300/300k_512x384_x264_172_quicktime_128.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=811310
800/800k_512x384_x264_672_quicktime_128.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=1210182
1200/1200k_1024x768_x264_1072_quicktime_128.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=2408106
2400/2400k_1024x768_x264_2272_quicktime_128.m3u8
```

### BITRATE EVALUATION

FFmpeg can report the bitrate of the ts stream. Use the ffprobe tool and you'll get output like this:
```
Input #0, mpegts, from 'foo.ts':
  Duration: 00:04:50.87, start: 2.000011, bitrate: 10381 kb/s
  Program 1
    Stream #0.0[0x810]: Video: h264 (High), yuv420p, 1280x720 [PAR 1:1 DAR 16:9], 25 fps, 25 tbr, 90k tbn, 50 tbc
```
The bitrate is being given in kilobits per second, so multiply it by 1024 and you'll have the value you need for the BANDWIDTH tag.
