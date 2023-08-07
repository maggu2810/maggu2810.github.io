rsync
===

***rsync related information***

**Author**: *Markus Rathgeb*

---

# Arguments (for me important ones)

```text
-z                          Compress file data during the transfer

-a, --archive               archive mode; equals -rlptgoD (no -H,-A,-X)
-H, --hard-links            preserve hard links
-A, --acls                  preserve ACLs (implies -p)
-X, --xattrs                preserve extended attributes
-v, --verbose               increase verbosity
-i, --itemize-changes       output a change-summary for all updates
-P                          same as --partial --progress
```

# Synchronize

```shell
rsync -aHAXviP <src> <dst>
``` 
