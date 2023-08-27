Youtube etc.
===

***Youtube Video / Music and related stuff***

**Author:** *Markus Rathgeb*

---

# Android

## Alternatives

| Name            | URLs                                                                                                                                                                                                                      | Comment                                                                            | Shorts on Channel of Fabi Rommel |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|----------------------------------|
| NewPipe         | [Page](https://newpipe.net/), [Repo](https://github.com/TeamNewPipe/NewPipe)                                                                                                                                              | A libre lightweight streaming front-end for Android.                               | missing                          |
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
