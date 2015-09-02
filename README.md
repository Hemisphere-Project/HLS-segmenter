# HLS-server
Linux HLS Server including uploader, segmenter, chunks dealer and media manager

This project is in early development status.

Available features:
## Segmenter


## TODO
= add BASEURL while creating m3u8 playlist
= validate HLS flux compatibility

= check if I-Frame can be optimized (at the beggining)
  = monitor: http://stackoverflow.com/questions/14005110/how-to-split-a-video-using-ffmpeg-so-that-each-chunk-starts-with-a-key-frame/14011638#14011638
  = After segementation: http://video.stackexchange.com/questions/4904/how-to-force-ffmpeg-to-insert-keyframe-at-first-frame-when-downsampling-a-framer
  = alternative solution to segment: https://www.ffmpeg.org/ffmpeg-formats.html#Examples-5
