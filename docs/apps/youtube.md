Youtube etc.
===

***Youtube Video / Music and related stuff***
**Author**: *Markus Rathgeb*

# Youtube Music

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
