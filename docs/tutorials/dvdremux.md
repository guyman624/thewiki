---
label: DVD Remuxing
description: Remux DVDs using FFmpeg
---

# DVD Remuxing

This guide can be used to remux DVDs using FFmpeg and MPC-HC

## Required

1. FFmpeg >=7.0 GPL build from [BtbN](https://github.com/BtbN/FFmpeg-Builds/releases)
   - [Quick link for Windows x64](https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-n7.0-latest-win64-gpl-7.0.zip)
3. [MPC-HC](https://github.com/clsid2/mpc-hc/releases)


## Finding Title/Angle/Chapter (MPC-HC)

DVDs can be split into Titles, Angles and Chapters. A title can be seen as a playlist (or group of playlists if the disc has multiple angles). Angles are used for many things from switching between different editions of the video (Broadcast / Home Video, Theatrical / Director's Cut), to switching between English and Japanese Credits/Typesetting. Note that each angle may have it's own set of chapters. The simplest way to find all 3 for the episodes you are trying to remux is to use MPC-HC. (MPV and VLC lack these capabilities)

1. Mount your ISO/insert your DVD.
2. In MPC, click *File > Open DVD/BD* and select the root directory of the DVD or folder containing VIDEO_TS.

!!!
You can click *Navigate > Title Menu* to skip the trailers/warnings and get to the Menu.
!!!

Once you are in the DVD Menu, you should be able to navigate to all the episodes. You will need to take note of a few things about the particular disc you are remuxing.

- Find out if the disc has multiple angles. You can change the current angle by going to *Play > Video Angle*
- Find out if the episodes are all in 1 title, or if each episode has its own title. You can view the current title by going to *Navigate > Titles*
- If each episode does not have its own title, find out where the Chapter marks are for each episode. You can view the current chapter by going to *Navigate > Chapters*


## Checking for PCM

If your DVD has PCM Audio, it needs to be converted to another format during the remuxing process. 
We can use ffprobe to check for PCM streams, and to check their stream ids. To do this, run the command:

`ffprobe -f dvdvideo -title <title> -i <path_to_dvd>`

Replace:
- \<title\> with the title number you got from MPC-HC. This will be an integer from 1-99.
- \<path_to_dvd\> with the path to the disc image, or the VIDEO_TS folder.

==- Example ffprobe output snippet
```
Input #0, dvdvideo, from 'D:\VIDEO_TS':
  Duration: 00:00:35.50, start: 0.000000, bitrate: N/A
  Stream #0:0[0x1e0]: Video: mpeg2video, yuv420p(tv, top first), 720x480, SAR 8:9 DAR 4:3, 29.97 fps, 29.97 tbr, 90k tbn
      Side data:
        cpb: bitrate max/min/avg: 9800000/0/0 buffer size: 0 vbv_delay: N/A
  Stream #0:1[0xa0](jpn): Audio: pcm_dvd, 48000 Hz, stereo, s16, 1536 kb/s
```
==-

In the example case, there is one audio stream and it is PCM. This stream will need to be converted to another lossless format. Since it is `Stream #0:1`, the stream id is 1.


## Remuxing
To remux, you need the latest FFmpeg from the link above. Other builds may not have the proper compile flags to include DVD Demuxing support.

We need to build a ffmpeg command for you to use. You can start with a base of:

`ffmpeg -f dvdvideo -preindex True -title <title>`

Replace \<title\> with the title you are looking to remux. This will be an integer from 1-99*. If neither of the following apply to your disc, skip to the end.

*\*Although 0 is a valid title number, ffmpeg treats it the same as title 1.*

==- DVDs with multiple angles
If you want to rip a non-default angle from the disc, add:

`-angle <angle>`

Replace \<angle\> with the angle number you are looking to remux. This will be an integer from 1-9.

==-

==- DVDs with multiple episodes per Title

If you have multiple episodes per title, you need to add:

`-chapter_start <start_chap> -chapter_end <end_chap>`

Replace \<start_chap\> with the first chapter of the episode you are remuxing. Replace \<end_chap\> with the last chapter of the episode, inclusive. Both accept an integer from 1-99

If you are including the first or last chapter, you can exclude the respective argument.

==-


#### Final Piece

After you have added the rest of the required flags, you can end the command with:

`-i <path_to_dvd> -map 0 -c copy <filename>.mkv`

==- DVDs with PCM Audio

If your disc has PCM Audio, add:

`-codec:<stream_id> flac`

**After** `-c copy` in the final part of the command.

Replace \<stream_id\> with the stream id you got using ffprobe. If you have multiple PCM streams, add another one for each stream. This will convert the PCM audio to FLAC. 

==-

Replace:
- \<path_to_dvd\> with the path to the disc image or the VIDEO_TS folder.
- \<filename\> with what you would like the resulting file to be named.

Here is an example of a working command:

`ffmpeg -f dvdvideo -preindex True -title 2 -angle 2 -chapter_start 4 -chapter_end 8 -i example.iso -map 0 -c copy -codec:1 flac test.mkv`