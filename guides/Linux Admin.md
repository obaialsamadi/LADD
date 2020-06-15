---
tags: [LA]
title: Linux Admin
created: '2020-06-06T11:05:09.731Z'
modified: '2020-06-15T23:26:51.036Z'
---

# Linux Admin
Always check the manual pages: `man sort`
- `ll -h` lists file size

## Comparing & Manipulating File Content & Input/Output redirection

- cat (hit tab when writing file name, could autocomplete)
  - paginate: pipe into `more` or `less` --> `cat file.txt | more`
- sort: `sort list.txt | more`
  - add `-r` to go in reverse order
- you can cat multiple files then pipe them into sort
- then you can use `>` to create a combined file, `>>` appends to another file
- fmt: formats texts
- nl: numbers your lines without saving it. You can `>` it into a numbered file
- cut

## Basic Regular Expressions

- Anchor characters: `^` starter character, `$` ending character
  - ^A: means look for something that starts with A
- grep: `grep '^the' searchinthistext.txt`
  - you can get super specific: `grep '^T[a-z][aeiou]' filename.txt ` means I want the first letter capital T, second letter is a lowercase, and the third letter is a vowel. It'll find that for you.
  - `grep '^T[a-z][^e]`: same but ^e means NOT e. `^` becomes a not inside square brackets

## Archive, Backup, Compress, Restore files

- copy: `cp <source> <destination>` --> `cp /data .` the . donates "copy here"
  - recursive copy would also copy directorys: add `-rf`
- tar: tape archive - `tar cvf <name of file.tar> <source>`
  - to compress: `gzip` a tar file. you can also do it in one step, add zip, `z`: `tar cfvz <name.tar.gz> <path u want backed up>`
  - extract: add x to tar command

## Create, delete, copy, move files & directories
- touch
- mkdir: you can make sub directories using `-p` in one go `mkdir -p main/second/third/fourth`
- mv: add slash after directory name to indicate that the file you are moving goes in there `directory/`. Used for renaming too.
- rm: when you delete it is gone forever. rmdir removes directories. add `-rf` to force removal. Add `*` after to delete everything in directory.

## Create and manage hard and soft links
Pointers

Hard link is the way when you cat a file your computer knows how to find the link

Say you have two directories. You want to access file 2 from directory 1: `ln ../dir2/file2 name-hard-link-to-file2`

Edits are reflected in original file and link, but deleting one doesnt delete both.

Soft link only links to the name of the file and not the actual location on disk like hard links, symbolic links:
` ln -s ../dir1/file1 file1`. you break the link if you make changes.

## File permissions

`ls -la` or `ll` shows permissions: youve seen `drwxr-xr-v`: each dash shows you each set of permissions (here there are 3, user-group permissions-everyone else): r:read w:write x:execute dr:directory

To make changes:
- change all rights: chmod--> command change mode: `chmod u+rwx <name-of-file-you-wanna-change>` the + adds those writes to `u` which is the user. do `g-rwx` and itll take it away from group users, everyone: `o`
Other way:
- use `a` all: `chmod a-rwx <file>`
- read:4 write:2 execute:1 `chmod 777 everyone` gives all of them all the rights. ``chmod 600 <filename>` gives user read write and nothing for everyone else
- x bit on directories allows navigation into that directory. The read bit allows contents to be listed.

## Read/use system documentation

- `man` pages: mandb updates database
- search for command: `apropos <command>`
- `info`
- `locate`: `locate sysctl.conf`
- `/usr/share/doc` shows you a list of docs

## Manage access to Root account

**DO NOT DO THIS:** this is how you take away or give root access

Give: `sudo visudo`. Go down to users section and add the name of the user and define the rights like you see `root` already has.
Take: same file then remove the user name from users section.

Wheel group also uses root and can use wheel group to do the same thing:
- `usermod -aG wheel <username>`

## Finding Files

- `find`: search by name, `find -name "name.extension"`, or `find / -name "name.extension"` where the slash searches from the root. If you want it to ignore case sensitivity: `find / -iname "name.extension"`
Sometimes you need to find all the files except a certain one: `find / -not -name "name.extension"`

Extra note: `df -h` shows you where your partitions are (diskfree -humanreadable)

Everything in Linux is a file, so you can specify a type and use `find` to find them: 
`find / -type l` finds all symbolic links. You specified the type using -type l. Replace `l` with `d` to find all directories.

Example: find all files that end in `.log`

`find / -type f -name "*.log"` f specifies type to be file.

Example: find by size
`find /usr/bin -size +200c` c is bytes or characters. Itll find 200c in /usr/bin. m is megabytes. k is kilo

Example: find by time created(timestamp)
`find / -type f -mtime 1` itll show when created more than a day ago

Example: find by user owned file
`find /etc -user <username> | more` finds user owned files in /etc and pipes it into more that pauses it after every result

Example: file by permission
`find /usr/bin -perm 755`

You can execute command based on what you "find"

`find / -name "test.txt" -exec chmod 700 {} \;` 
-exec makes it execute change mode or chmod and change permission using 700, {} is just a place holder
`644`: read/write by owner and read only for others


- `locate`: relies on a special database being populated. To find all current files: `updatedb`.


## Basic File System Features

- A block device is a set of addressable blocks used to store and retrieve data: SSD, USB

- A file system is where a computer system persists general data for users/apps

## Input and Output manipulation of files

use `cat` then pipe into `sort`
`cat hardwarelist.txt shopping.txt | sort > combined.txt`
cats the file sorts it then created combined.txt and throws them there

Extra: `fmt -u filename.txt` formats the file
`man fmt` gives you manual info about fmt
Extra: `nl filename.txt` numbers the output

