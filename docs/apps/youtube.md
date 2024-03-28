Youtube etc.
===

***Youtube Video / Music and related stuff***

**Author:** *Markus Rathgeb*

---

# Common Clients

| Name       | URLs                                           | Comment                                                                                                        |
|------------|------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| youtube-dl | [Repo](https://github.com/ytdl-org/youtube-dl) |                                                                                                                |
| yt-dlp     | [Repo](https://github.com/yt-dlp/yt-dlp)       | found by [fix uploader id error](https://appuals.com/youtube-dl-error-unable-to-extract-uploader-id/) |

# Android

If you would like to use the command line version you could use termux and e.g. youtube-dl or yt-dlp.

You need to install python and after that you could use pip to install the respective tool of your choose.

## Alternatives

| Name            | URLs                                                                                                                                                                                                                      | Comment                                                                            | Shorts on Channel of Fabi Rommel |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|----------------------------------|
| NewPipe         | [Page](https://newpipe.net/), [Repo](https://github.com/TeamNewPipe/NewPipe)                                                                                                                                              | A libre lightweight streaming front-end for Android.                               | available, downloadable                          |
| PipePipe        | [Repo](https://codeberg.org/NullPointerException/PipePipe)                                                                                                                                                                | A DIVERGED fork of NewPipe with more services, client features and bug fixes.      | available, downloadable          |
| SkyTube         | [Repo](https://github.com/SkyTubeTeam/SkyTube)                                                                                                                                                                            | Fully open-source and free software.                                               | missing                          |
| SkyTube Extra   | --                                                                                                                                                                                                                        | Contains extra features that are powered by non-OSS libraries.                     | missing                          |
| Youtube Vanced  | [Info](https://www.xda-developers.com/best-youtube-vanced-alternatives/), [Repos](https://github.com/TeamVanced)                                                                                                          | Discontinued                                                                       | --                               |
| ReVanced ...    | [Repos](https://github.com/revanced), [ReVanced Megathread](https://forum.xda-developers.com/t/app-guides-unofficial-revanced-megathread.4523967/), [Unofficial Builds](https://androidfilehost.com/?w=files&flid=337187) | Seems currently the best option to use an enhanced / modified official Youtube App | available, not downloadable      |
| YouTube Vanced+ | [Repo](https://github.com/cuynu/ytvancedx)                                                                                                                                                                                | Not tested, did not get the whole store till now.                                  | --                               |

# PC

## Youtube Music

* [YouTube Music Desktop App bundled with custom plugins (and built-in ad blocker / downloader)](https://github.com/th-ch/youtube-music/releases)

# youtube-dl

## Download with youtube-dl

### Common Information

[Output Template](https://github.com/ytdl-org/youtube-dl/blob/master/README.md#output-template)

### Youtube Music

#### Album

```
youtube-dl --download-archive dl-archive --ignore-errors --format bestaudio --extract-audio --audio-format mp3 --audio-quality 160K --output "%(album)s/%(playlist_index)02d. %(track)s [%(artist)s].%(ext)s" --yes-playlist 'https://music.youtube.com/playlist?list=OLAK5uy_nLNcbPrtDMXDrj55lF8WwxAyAapQ1ZjqE&feature=share'
```

### Youtube

#### Example

```
youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio' --merge-output-format mp4 "${@}"

youtube-dl -f bestvideo[ext=webm][width=1920][height=1080]+bestaudio[ext=webm] --merge-output-format webm --cookies /tmp/cookies.txt 'https://www.youtube.com/watch?v='"${i}"
```

# External Information

Taken from: [stackoverflow comment](https://stackoverflow.com/a/64526840)

## If you want to download audio in `opus` format - with best possible quality and without any conversion

In most of the cases best quality audio is oryginally provided by youtube in `opus` format. You can download it in
highest possible quality using this command:

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio https://www.youtube.com/watch?v=c29tZVZpZGVvUGxheWxpc3RQYXJ0
```

However `opus` format might be inconvenient for many reasons. For example some media players especially in cars and
telephones might not support it. Probably you just want to have audio in `mp3` file. Below there is a solution on how
to (using one command) download audio and convert it to probably most popular `mp3` format with lossing as little
quality as possible.

## If you want to download audio and convert it to mp3 or any other lossy format.

### To download audio from a single movie:

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-quality 0 --audio-format mp3  https://www.youtube.com/watch?v=c29tZVZpZGVvUGxheWxpc3RQYXJ0
```

To perform this command you need `ffmpeg` installed (audio and video converter which youtube-dl uses for convertion)

It downloads **only audio** (without video) and converts it to `mp3`.

**Option `--audio-quality 0` is very important there!**
Without this option you lose a lot of sound quality during `mp3` compression.

`--audio-quality 0` tells youtube-dl to save audio file in the best quality (when converting to `mp3`).
Without this option `mp3` audio-quality is set by default to `5` in `0-9 scale` where `0` is the best quality and `9`
the worst quality. So by default quality is worse. Youtube streams for nonpremium users with variable bitrate up
to `160kbps` in `opus` format. `Opus` format is newer than `mp3` and has better compression than `mp3` preserving the
same quality. So `160kbps` opus = `~256kbps` mp3. When audio-quality is default (`5` in `0-9 scale`) mp3 bitrate is
limited to `160kbps` which means that some sound quality is lost during compression. When audio-quality is set to `0`
mp3 goes up to `300kbps` preserving almost original quality. Almost original quality because `mp3` is a lossy format so
something is lost when converting to it. By using `--audio-quality 0` option we just make sure that we loose as little
as possible during this conversion. So difference between original `opus` audio file and audio file converted to `mp3`
is so small that it might be hard to spot by ear.

### To download audio from all movies from a channel:

Command is the same, but you should put link to the channel instead of link to a single video:

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-quality 0 --audio-format mp3 https://www.youtube.com/c/someChannelName1232143/videos
```

### To download audio from a playlist:

You have to add `--yes-playlist` option.
You can put a link to a playlist (link with `playlist` word):

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-format mp3 --audio-quality 0  --yes-playlist https://www.youtube.com/playlist?list=c29tZVZpZGVvVVJMUGFy
```

Or the link to one of the songs from playlist while playing playlist (link with `list` word):

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-format mp3 --audio-quality 0  --yes-playlist "https://www.youtube.com/watch?v=c29tZVZpZGVvUGxheWxpc3RQYXJ0&list=c29tZVZpZGVvTGlzdFBhcnRzc29tZVZpZGVvTGlzdFBhcnRz&index=4"
```

## If you want to download audio and convert it to flac or any other lossless format

In this case you don't have to specify `--audio-quality` option as it is ignored by youtube-dl during conversion to
lossless formats.

### To download audio from a single movie:

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-format flac  https://www.youtube.com/watch?v=c29tZVZpZGVvUGxheWxpc3RQYXJ0
```

### To download audio from all movies from a channel:

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-format flac https://www.youtube.com/c/someChannelName1232143/videos
```

### To download audio from a playlist:

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-format flac --yes-playlist https://www.youtube.com/playlist?list=c29tZVZpZGVvVVJMUGFy
```

or

```
youtube-dl -f "bestaudio/best" -ciw -o "%(title)s.%(ext)s" -v --extract-audio --audio-format flac --yes-playlist "https://www.youtube.com/watch?v=c29tZVZpZGVvUGxheWxpc3RQYXJ0&list=c29tZVZpZGVvTGlzdFBhcnRzc29tZVZpZGVvTGlzdFBhcnRz&index=4"
```

## Command options explanation:

```
-f "bestaudio/best" <- Choose the best audio format.
As there is only audio format listed only the audio is downloaded.

-c <- (--continue) Force resume of partially downloaded files.
By default, youtube-dl will resume downloads if possible.
As docs state maybe it is default, but I put it to make sure it is set.

-i <- (--ignore-errors) Continue on download errors,
for example to skip unavailable videos in a playlist.

-w <- (--no-overtwrites) Do not overwrite files
(If something was already downloaded
and is present in the directory then continue with the next video)

-o "%(title)s.%(ext)s" <- (--output) Output filename template,
in this case it gives you file named movieTitle.mp3
where movieTitle is the title of the video on youtube.

-v <- (--verbose) Print various debugging information

--extract-audio <- (-x) Convert video files to audio-only files
(requires ffmpeg or avconv and ffprobe or avprobe)

--audio-quality 0 <- Specify ffmpeg/avconv audio quality,
insert a value between 0 (better) and 9 (worse) for VBR or a specific
bitrate like 128K (default 5).
Youtube streams for nonpremium users with variable bitrate up to 160kbps in opus format.
Opus format is newer than mp3 and has better compression than mp3
preserving the same quality. So 160kbps opus = ~ 256kbps mp3.  
When audio-quality is default (5 in 0-9 scale) mp3 bitrate
is limited to 160kbps which means that some sound quality is lost during compression.
When audio-quality is set to 0 mp3 goes up to 300kbps preserving original quality.

--audio-format mp3 <-  Specify audio format: "best", "aac", "flac", "mp3", "m4a",
"opus", "vorbis", or "wav"; "best" by default; No effect without -x (--extract-audio).
In this case we choose mp3.
Alternatively you could choose for example flac which is loseless codec.
```

All links that I provided there are fake. I just put some random words encoded by base64 in them. So you have to replace
them by your own links to make it work.

## Hint:

```
Youtube-dl gives you opportunity to use your own youtube account to download stuff.
If your account is a premium account you can get
higher 320kbps opus bitrate which is equivalent of ~512kbps mp3.
Using your own account might be possible by setting --username and --pasword
(See Authentication Options in --help)
```

## Note:

All of the above should be relevant for version `2021.12.17` of youtube-dl. Newer versions might change something, so be
aware of that.
