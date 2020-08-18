```
           4000    (the setuid bit).  Executable files with this bit set
                   will run with effective uid set to the uid of the file
                   owner.  Directories with this bit set will force all
                   files and sub-directories created in them to be owned by
                   the directory owner and not by the uid of the creating
                   process, if the underlying file system supports this
                   feature: see chmod(2) and the suiddir option to
                   mount(8).
           2000    (the setgid bit).  Executable files with this bit set
                   will run with effective gid set to the gid of the file
                   owner.
           1000    (the sticky bit).  See chmod(2) and sticky(7).
           0400    Allow read by owner.
           0200    Allow write by owner.
           0100    For files, allow execution by owner.  For directories,
                   allow the owner to search in the directory.
           0040    Allow read by group members.
           0020    Allow write by group members.
           0010    For files, allow execution by group members.  For direc-
                   tories, allow group members to search in the directory.
           0004    Allow read by others.
           0002    Allow write by others.
           0001    For files, allow execution by others.  For directories
                   allow others to search in the directory.
```
