# Useful FFmpeg Commands for Video Conversion and Manipulation

FFmpeg is a highly versatile command-line tool that enables users to manipulate digital media. With its extensive capabilities, FFmpeg can perform a wide range of tasks, including video and audio conversion, editing, streaming, and more. 

This article explores various useful FFmpeg commands for converting between different video formats, adjusting quality, splitting and concatenating videos, and creating GIFs.

### Convert mov to mp4

```bash
ffmpeg -i  original_video.MOV -map_metadata 0:g -q:v 0 output_video.mp4
```

### Convert webm to mp4
```bash
ffmpeg -i  original_video.webm -map_metadata 0:g -q:v 0 output_video.mp4
```

### Lower Quality (with specific resolution)

```bash
ffmpeg -i  original_video.mp4 -map_metadata 0:g -r 30 -s 960x540 -vcodec libx264 output_video.mp4
```

### Lower Quality (with specific codec and rate)

```bash
ffmpeg -i original_video.mp4 -vcodec libx265 -crf 20 output_video.mp4
```

Additional options that might be worth considering is setting the Constant Rate Factor, which lowers the average bit rate, but retains better quality. Vary the CRF between around 18 and 24 — the lower, the higher the bitrate.

Options for ffmpeg

- -vf "transpose=2" (rotate)
- -map_metadata 0:g (keep metadata)
- -q:v 0            (lower quality)
- -r 30             (rate)
- -s 960x540        (resolution)

### Convert all files in a folder

```bash
for f in *.webm; do
  ffmpeg -i "$f" -c:v copy -c:a copy "${f%.*}.mkv"
done
```

### Split Video

```bash
ffmpeg -i original_video.mkv -t 00:10:00 -c:v copy -c:a copy output_video.mkv
```

### Keep Part of VIdeo

The following command keeps the part of video between the 1 minute and for **duration** of 5 minutes

```bash
ffmpeg -i original_video.mkv -ss 00:01:00 -t 00:05:00 output_video.mkv
```


### Concat Videos

```bash
ffmpeg -i video_1.mkv -i video_2.mkv -i video_3.mkv -i video_4.mkv -filter_complex "[0:v] [0:a] [1:v] [1:a] [2:v] [2:a] concat=n=4:v=1:a=1 [v] [a]" -map "[v]" -map "[a]" output_video.mkv
```

### Create gif

```bash
ffmpeg.exe  -i original_video.mkv -f gif image.gif
```

### Create gif scaled down

```bash
ffmpeg.exe  -i original_video.mkv -vf fps=15,scale=1024:-1 scaled_image.gif
```

- -fps=15 (number of fps)
- -scale=1024:-1 (scale resolution)
