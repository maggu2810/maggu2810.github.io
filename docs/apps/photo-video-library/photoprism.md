PhotoPrism
===

***PhotoPrism related information***

**Author**: *Markus Rathgeb*

---

# Get file names for / per album

```mariadb
use photoprism;
select
	albums.album_title, CONVERT(files.file_name using utf8)
	from albums
	inner join photos_albums on albums.album_uid = photos_albums.album_uid
	inner join files on files.photo_uid = photos_albums.photo_uid
	where albums.album_type = 'album' /* and albums.album_title = "..." */
	ORDER BY albums.album_title ASC;
```
