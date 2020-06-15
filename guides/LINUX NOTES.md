---
tags: [GL]
title: LINUX NOTES
created: '2020-05-03T17:11:48.269Z'
modified: '2020-06-09T15:34:05.389Z'
---

# LINUX NOTES

## Debian Notes

Two main sources :
* debian
* debian-security

Full list of Debian sources:
- `deb https://deb.debian.org/debian buster main contrib non-free`
- `deb https://deb.debian.org/debian-security buster/updates main contrib non-free`
- `deb http://deb.debian.org/debian buster-backports main contrib non-free`
- `deb http://deb.debian.org/debian/ testing main contrib non-free`
- `deb http://deb.debian.org/debian/ testing-updates main contrib non-free`

The first two sources offer ackages that are very stable, albeit a bit old. If you want to install latest software, you must add *-backports.

You can see the Debian version after the words debian or debian-security, buster in this case. Backports in ubuntu and debian sources are usually later software, available for next generation of operating system, compile to run on the edition attached to backports
ie . in debian sid distro, you have aria2 1.35. In mainline buster, you have aria2 1.20. If you want to download a later version, you shouldn't usually add sid sources since it may have packages not compatible with your current distro. You would add backports. In backports, you would probably have a version like aria2 1.27, which would be higher than the version in main source but ususally, it's lower than the version in unstable source, knowing this can save you a great deal of frustration. 

How? If one was to provision servers with some software from apt and running them with specific flags, they would fail. This is because the flags used to execute on the local environment were for a later version of the software and wouldn't be compatable with the older version on the server. Sources should be updated to fix this issue. 

### Using apt

apt maintains a cache of all software and their dependant packages in `*.list` files in `/etc/apt/sources.list.d` and the main source file `/etc/apt/sources.list`
- whenever you make any changes to the above mention file/dirs , you must call apt update so that it would sync with remote sources
- once you run apt install, it would try to gather direct download link for the target package and its dependencies based on whats found in local sources cache
- it would download installer files. debian/ubuntu installer files end in *.deb extension. After downloading them , it would pass the files to dpkg to install them. dpkg is the main installer. There are some cases where you can't find software with apt, so you would download *.deb file and run dpkg -i <*.deb> to install it. -i is a flag that stands for install

These operations need sudo because they make changes to the root file system, usually at:
- `/usr/bin /usr/local/bin` : these are where main executables are stored. They are the default PATH
- `/etc/<software-name>` : this is the default location linux software stores STATIC configuration files, i.e files that are not supposed to change after upgrade. Example: default ansible config files go under `/etc/ansible`. 

There are other dirs in linux fs. For now, we are dealing with dirs that are usually touched by installer , i.e. apt/dpkg/ `/usr/` and `/etc/ dirs` are owned by root, so to make any changes, you must be sudo, thats why you must call theses commands with sudo , so that you elevate you permission to root user.

---
## Linux Directories
- `/usr/bin`: binaries and executables installed with apt/dpkg. The stuff package manager installs for you.
- `/usr/local/bin`: executables that you add manually, i.e when you create a bash-script that you want to have in your path or download a tar.gz or .zip file with the compiled executable( hashicorp software are all distributed as single binary in a zip file). You would put your custom scripts here to help you with day to day ops.
- `/usr/local`: stores stuff that has even a smaller area of impact, it contains your local customizations for example.
- `/bin`: software added at the time the root file system for your distro was created, such as bash,cat,ls. This is where all the core commandline executables are stored. Bare bone essentials.
- `/sbin`: the binaries that deal with low-level things, can be though of as the super admin binaries.
- `/sbin/restart` or `/sbin/shutown`: restarts or shuts down the server/your computer or commands that get information about your IP interface and network.
- `/sbin/ifconfig`: used for reading information about network links.
- `/tmp`: any files put here get erased after reboot. It doesn't need sudo to store or use. Scenarios covered in SSH section.
- `/opt`: all you need to know about /opt is that it is used to hold files for 'non standard'  installations in which you must setup a shortcut on gnome panel/desktop by adding a file there.
- `/dev`: hdd and storage units are formated in : `/dev/sda` or `/dev/sdb`. `/dev/sda1` points at a parition in `/dev/sda`. 
`/dev/null` is used to dump the output ( stdout ) rather than show it on terminal.
`

In many cases, the directories under `/usr/` are the same as in root fs. As far as PATH is concerned, there is no differnce. For exampe: `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` is the PATH on a barebone debian installation. 

- `/lib`: The /lib directory contains kernel modules and those shared library images (the C programming code library) needed to boot the system and run the commands in the root filesystem, ie. by binaries in /bin and /sbin. Libraries are readily identifiable through their filename extension of *.so. Windows equivalent to a shared library would be a DLL (dynamically linked library) file. They are essential for basic system functionality. Kernel modules (drivers) are in the subdirectory /lib/modules/'kernel-version'. To ensure proper module compilation you should ensure that /lib/modules/'kernel-version'/kernel/build points to /usr/src/'kernel-version' or ensure that the Makefile knows where the kernel source itself are located.

#### Static vs Dynamic

Compiling software dynamically does not include external dependancies in the final bompiled binary. This is why when you are installing something with apt, you see apt installing a bunch of other packages. apt tries to determine what libraries that said software needs to run and gets them for you. Dynamic linking expects some libraries to be present at the time of executing the compiled binary. These libraries in linux end in *.so extention, this is the extension of complied c/cpp libraries, this is equivalant to windows *.dll usually. Under `/lib` folder, you have a software's dependant libraries or crucial config files. One reason applications include " *.so" files is to allow other applications and other developers to call functions that they export.

Static linking includes ALL dependancies in a single executable. That's one of the reasons go popularity increased. It only compiles statically (as long as you don't call into C functions/librarie from go). Thats why when you extract a hashicorp software and put it under /usr/local/bin , it would always work.

Keep in mind that when you compile a software, it would compile it for a certain architecture,eg x64 (amd64). So when you compile a binary with go, for linux for x64, it would work on any linux distro, whether it is debian, fedora, arch , alpine, etc. because the only thing that really matters for statically compiled binary is linux kernel, which is shared across all those distros. This is why we compile binaries in go and put them in a docker container and it would work, no matter the distro you choose to run it in. However, since all dependancies are included, statically compiled binaries are larger. This is a drawback for IoT and embedded systems.

Besides libraries, your application might have config files that are not to be modified, they would be stored in `/lib`. Modifiable config file that won't change as you upgrade your app are under `/etc/`. config files that generally should not be touched by user would be stored under lib.

So the next time your application is not running the config file you gave it, you would look into `etc` and the confrim that you ran the right flag for that specific version.

### Container vs VM

A container is an isolated environment with it's own isolated filesystem that shares the underlying os stack. A virtual machine is an isolated environment with it's own isolated filesystem that does not shares the underlying os kernel but rather has it's own kernel.

Kernel is the core of every operating system. It implements features such as task scheduler (such as round robin), threads and parallelism and comminucation with IO prepherals, such as reading /writing to/from File descriptor, socket ( ctual storage of file on disk, connecting to USB key).

---

## Root access and Bash

When making a golden image that has consul and vault, make sure to put config files under `/etc/vault` and `/etc/consul` to be consistent.

Some rough windows vs linux comparisons: 

- `C:\windows\system32 = /lib`
- `C:\windows\system32 = /bin`
- `C:\windows\system32 = /sbin`
- `C:\Program Files = /usr/bin`
- `C:\Users\username = /home/username`

Root user always exists. `/home/` might not have any users. Be mindful of this when running scripts. Try to be in sudo because you can guarantee that root user always exists, whereas others won't, especially when launching containers. If you are running a script that is moving a file,the script is getting executed on debian base image with no users and you would have it move to "${HOME}/blah", it would fail because user does not exist. 

One common way to execute instructions in bash scripts is `bash -c "<instruction>"`. Add `sudo` before `bash` to run as root. There are other shells besides bash, so sometimes you need to make sure you are using the bash shell and force it to by using `bash -c` or `/bin/bash`. `sudo /bin/bash -c "echo $HOME"` and `sudo su` then `/bin/bash -c "echo $HOME"` would return different results. This is because when you open a terminal, whatever is inside `~/.bashrc` is executed by default. `sudo su` forces a new shell instance started as `root` with root's `~/.bashrc` located at `/root/.bashrc`.

Side Note: to execute a command with ssh on a remote machine, use the following snippet :
`ssh -t <user>@<IP> <Command>`.

**Double and Single Quotes**

A "  wraps a variable and translate into what is stored in the variable when bash runs it but ' forces bash to see it as string literal. 

`echo 'FOO env variable is equal to $FOO' `
`echo 'FOO env variable is equal to "$FOO"'`
`echo "FOO env variable is equal to '$FOO'"`

The way bash deals with $Variable is when it is encapsulated inside " vs '
at the time of bash , actually excuting the instruction, it would take $Variable and replace it with its values
if it is encapsulated in single quotes , it would interpret it as a basic bitch string literal
now to wrap this shit up
"PATH" is not an array it's a single string
it gets 'parsed' by shell
shell thinks of every string set before and after : delimiter and a single entry to your Path
it would look into all of those, trying to see if it can find an executable with the name request, if it can find one, then it would run it

---

#### `stream-dl` 

Streaming files on the web have `.m3u8` extension. Tour browser loads the `*.m3u8`file, called a playlist file. All it has inside it are links to a list of chunks that together make up the original stream, so a streaming video is not a single file. It's usually hundreds of smaller chunks, all holding information regarding which url to load them from, what part of video that chunk is (second x to second y), how to decrypt the chunk, etc are all in the playlist file
It's actually plain text, so in case of some streams, even if you open the page's source code, you can only find the m3u8 file, preventing you or any automatic crawler from downloading the video. It's the same with compoundmedia, gasdigital, crunchyroll and funimation, except the last two have encrypted their chunks, which makes ripping their content harder.

The waay people rip a stream and convert it to a normal file is through an application called ffmpeg. The playlist file is passed to `ffmpeg` and through flags , it is asked to merge it.
The playlist file can point to chunks existing on a remote location or local file system.
stream-dl is a bash executable that:
- loads a remote playlist file
- parses it and extracts direct links to chunks
- creates a single file, filled with chunk names
- passes that file for batch download to aria2
- uses aria2 with optimal flags to download chuncks with max speed
- uses ffmpeg to merge them back into a single, normal playable file

The drawback is it is really slow. aria2 can download 16 chunks in parallel with max speed. 

An example of a stream-dl executable has --init flag which:
- adds mkvtoolnix (used for making *.mkv videos) source
- adds nodejs and yarn source
- installs mkvtoolnix, ffmpeg, nodejs and yarn on your device
- fixes path so that npm command and yarn command are available

Issues: apt update and apt install need sudo. The installers for Node and yarn are in a single function inside that funtion. Besides installing node, yarn and ffmpeg progress-bar package, I am modifying $USER's path. $USER is a special 'reserved' environment variable, its the same as $PATH. It points to the user that started the current shell the script is executing in.
Right at the start of that function, you have `[ "$(whoami)" = root ] || exec sudo "$0" "$@"`
where (whoami) returns current user's name. The left hand side is bash's short hand for if statement, basically [ "$(whoami)" = root ] would KIND OF translate to this psuedo code
if ("$(whoami)" == root ).

`[[ $condition ]] || echo "executes when condition is false" && echo "executes when condition is true"`

 The `[ "$(whoami)" = root ] || exec sudo "$0" "$@"` line checks to see if username is root ( we wanna make sure only people with root permission are running because in other cases, apt operations inside the function would fail).

There are two other keyword-like variables in this line:
- "$0" is the base name of what is getting executed. If you put stream-dl into /usr/loca/bin and then in bash, type stream-dl..., "$0" would be "stream-dl"

In bash functions:
- `$1` -> the first arg passed to function
- `$2`-> the second arg passed to function
- `$3` -> third arg passed to function and so on

What if you don't know how many args are getting passed to the function? One challenge with scripting and using interpreted language is that functions do not have a "solid" signature like languages in which the compiler actually compiles the code. This problem exists in matlab bash javascript python but it shows itself differently in all these languages, you can't enforce argument type. For instance in `go`, you would write `func foo (myArg string){}`
where foo takes in a single argument of type string, in python, you would write `def foo(myarg`
Here , you dont know what type myarg is when this function is invoked, the programmer may pass in interger rather than a string, but the function's instructions are based on myarg being a string.

In bash, you would write it as: 
`function foo() {
local myarg=$1
}`

In signature , it doesn't even show how many args it supposed to accept. `"$@"` in bash means = take in whatever number of args that that are passed into this function, whether it has zero args or a thousand. Let's analyze exec sudo `"$0" "$@"` again:
- `exec sudo`: start a new shell as root
- `exec sudo + "$0"`: starts a new shell as root and execute whatever is in `"$0"`
in a stream-dl case, $0 == stream-dl, so bash in root shell would look for an executable called stream-dl. Since stream-dl exists at /usr/local/bin or `/usr/bin` it doesnt throw an error
- `exec sudo + "$0" + "$@"`: pass in all the recieved args to what you just executed.
After this line , it is like you have switched user to root , just like calling sudo su. At the end of this function, I am calling a function called `add_to_path`, which adds whatever is passed to it to `${HOME}/.bashrc`. Since shell switched to root, the main user's bashrc is not modified, i.e /home/username/.bashrc, but rather root's bashrc got modified. 

Look at where the function is defined (inside a github repo for example: `https://github.com/da-moon/core-utils/blob/master/lib/install/node.sh`) because you may need to write scripts that would install nodejs. If a webserver uses node or you want to start a website on remote server, you must provision the server to have node and yarn installed
You decide to use yarn to download your website's dependencies ( `node_modulles`) because yarn is much faster compared to npm, using a script that doesn't account for where yarn is in the path would fail. Even if it finds yarn in path, you cannot return yarn packages installed globally because the location of global yarn packages are not added to username path but rather to root. 

---

### Cautionary Tale

Let's say you are using ansible or terraform and you have set them to revert all changes in case of failure, then all the time it took to provision a raw server (may be 10 minutes) is lost. Worst case is you won't detect failure and in the future, it has a domino effect because in some edge case, your application needs a package installed with yarn globally, it may be 3-6 month past when you deployed the fleet and you have forgotten how you set it up and which scripts you used. 

At that point the edge case is triggered and your server would fail. Imagine the failing server is a core component of the service mesh, what all other services depend on, my login component, then your serrver will go offline and it may take you up to a day to detect the failure. During that period you will lose revenue, customers and your reputation and possibly your job, all because you messed up in a single line of code. This is why one must master bash.
It would help  with python and other scripting languages since they share the same paradigms, and make you a better programmer.

This is precisely the reason why in every script you must tell the executing shell to fail when there is a `return 1`. It is better to fail early and fail hard so that you can fix those bugs before moving to production environment. 

---

## `chroot` Command

chroot (tcho-root) is a command to change a specific directory to become the new root directory. This means once you are inside a "chrooted" directory, you cannot access any directory above it. It is the root now. You cannot access programs outside of it; this is referenced as a Directory Jail. It can be an alternative to VMs, but there are differences.

As sudo, it allows you to use another folder/partition/ image within the shell you are working in. `chroot . bash` would you put you in this file system: run `uname -a` to view. You are no longer looking at your own file system, but rather the "chrooted" directory.  

If you try to `chroot` into an empty directory, it will try to launch `/bin/bash`, but since it is empty it will fail. Make sure you can run the bash shell, along with its dependancies (bin directories, lib and lib64 directories or whatever your arch is). 

So now when you chroot, you are inside the 'jail'. You have access only to what is within. This is useful for isolating apps in separate containers so they do not cause damage outside the directory. Additionally, you would only be able to run commands available in your `/bin/` folder like echo or cat or grep etc. 

The host OS can still give selective access to system resources through a mount bind. This is a method of passing resources from the external environment into the isolated chroot. 

You can have the "chrooted" directory immediately execute a script within it once it is called. Create a script and give it permission (using chmod). Now, once you chroot and call the bash script, `chroot` will execute it.

### chroot vs VM

LXD is very close to chroot, but this comparison focuses on VMs such as VirutalBox:

| **chroot** | **VM** |
|--------|----|
|  uses same processor/kernel      |  emulates a processor |
|  less isolation     |  more isolation |
|  lower overhead(running inside same space as host)     |  more overhead |r

---
## SSH

One minor purpose is remote login. It is always on port 22 so make sure it is not blocked when using. Main use is execution of commands on a fleet of remote machines. The `-t` flag makes ssh execute a command or script on a remote target. 

One way you can provision a fleet is to have your script ready, whether in your machine or on some cloud, like github , then create an array of ip addresses in bash of your fleet and use gnu parallel or a simple bash for loop to run the scripts. They follow this general form: 
`ssh -t <user>@ip /usr/bash -c "echo 'hello world'"` or `ssh -t <user>@ip echo 'hello world'`

In Ansible's case for example, all it does it ssh into servers and execute commands there. 

#### SCP

Used for file transfer: `scp root@pascal.ee.ryerson.ca:/path/on/remote /path/on/local` or `scp /path/on/local root@pascal.ee.ryerson.ca:/path/on/remote`

In ansible and terraform you can move a script file from local file system to remote and have it execute there. In terraform , it's called `remote provisioner`.
What it does is that it reads a script, copies it with scp to /tmp dir of remote, then uses ssh -t mechanism to tell shell in remote to execute the script
Pitfall: you can in theory move thef iles anywhere with scp but that not what happens in practice. Say your target folder in target machine is `/usr/local/bin`.
It will fail because this directory needs sudo permission. Instead transfer them to /tmp because it is the only dir gauranteed to exist on every linux distro and can be accessed without sudo. Then I use Terrafrom's remote exec to move the script from tmp to target.
Remote exec in terraform just executes command on remote, just like ssh -t but with some fail safe mechanism.

Flow of moving config files scripts, binaries, etc from local file system to remote system:

1) transfer it to /tmp
2) tell remote shell to do what it wants to do with it and/or move it to its final destination
for instance for config files, first they go into tmp then on remotes as admin , you mv them into etc

`tmp` has another purpose too. If you want to download an installer but don't wanna keep it on disk after you installed the software or a script that's supposed to run once and be done with it, download them to `/tmp install/execute`. They'll be gone after reboot.

<span style="color:blue"> Common Devops Task<span>: 

Downloading statically linked binaries that do not have an installer and making them available to users / in path

Let's try it with two common software: terraform and traefik

- traefik is a popular load balancer/reverse proxy like envoy
- terraform is a tool that lets you write your infrustructure in code, making it be reproducible and quick to deploy, scale up or down
there are 3 main tasks involved in these 'installations'
1- download the archive (it's almost never uploaded as barebone binary, its always either in a zip file or some tar* file)
2- extract the archive
3- move the extracted binary to path / install it / create a symlink

<span style="color:orange"> **Useful Tip** <span>: 

you may want to deploy something on a remote but you are not sure of it's architecture. You need to be sure because arm binaries won't run on amd64 binaries:
x64 = amd64 = 64 bit intel architextur
x86 = i <something!> = 32 bit intel architecture
aarch64 = arm64= armv8

`cat /etc/os-release` would retrieve architecture type.


This matters when you are adding apt sources. In many cases debian main sources may cause issues with ubuntu or newer codename sources may not work on old ones, so
you must make your script extract os name and codename when adding a source. 

#### PATHS: Addressing files and directories

- absolute path : how to get to target from rootfs (/) ie : /etc/apt/sources.list or /tmp/terraform
- relative path : location of the file with regards to where you are now . . means directory shell is in at moment ; .. means parent dir

When using addressing files, dirs , try you best to use absolute path. Relative sometimes breaks.

`export PATH="/tmp:$PATH"`: add /tmp to path, making all executables under this directly callable

- add a single line without opening a file
`echo 'export PATH=/tmp' » ~/.bashrc`
`>` OVERWRITES he file while `»` appends to file. Both will create a file if it doesn't exist.

After that you have to source the file (import): 

when you source a file in bash, you tell bash to read that file line by line. If there are function declerations, load them in memory, if there are global instructions, run them. The functions get loaded into memory and the instructions would run.

To source: `source <filename>` or `. <filename>`

<span style="color:orange"> **Useful Tip** <span>: 

Always encapsulate strings that a part of the is a variable with a "

Btw removing `$PATH` from a declaration you override the path environment with a single entry. Adding the `$PATH` expands to a real path that has all the binaries, then , it would assign the whole , 'concatenated' string to variable on left hand side. 

[Click me for more info on paths!](https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux-unix)

### `symlinks`

Think of them as shortcuts in WINDOWS. You can put an application under a folder then make a shortcut to it using symlink. Example:

`sudo mkdir -p /opt/hashicorp`
`sudo mv /tmp/terraform /opt/hashicorp/`
`sudo ln -s /opt/hashicorp/terraform /usr/local/bin/`

then list: `ls -lah /opt/hashicorp`

Careful with it tho, moving to target device could cause all symlinks to get lost and installer will panic. 

---

### nginx

Just like envoy. It redirects ports (from 80 to 8080 for example). It is best suited for single deployments because it's light so it is always available; host simple website or bind ports and domians for a single server.

---

#### first lesson: 

Some common tools and snippets with gnu coreutils and busybox

- head : head and tail utilities can help read first n lines and last n lines of a file . 
  - -n flag specifies how many lines we want to extract e.g. head -n 5 /path/to/file wil read first 5 lines.
  - -c flag specifies how many bytes we want to extract e.g. head -c 5 /path/to/file  will return 5 characters
- tr is used to trim a string that is piped into it from standard input and pushes the result to standard output . it takes in sets of characters to filter stuff.
  - -d or --delete flag : delete characters in the set
  - -c or --complement flag : complements result of set . for instance -dc flag will take out all characters from string that is piped into it but the pattern in passed set to tr. as an example , the following snippet removes all non letter and digit characters from a file cat /path/to/file | tr -dc A-Za-z0-9 > /path/to/file-trimmed
  - -s or --squeeze-repeats is used to remove character repetition. for instance , the following snippet will remove repeated empty lines from file and store it in another file cat /path/to/file | tr -s '\n' '\n' > /path/to/file-no-empty
- dd used for bit by bit copying
  - bs=<?> argument : chooses the block size dd uses. dd copies block by block. if this number is too small ( default value is small ) and source file is large, then it would take a long time to copy
  - if=<?> argument : source file to copy from
  - of=<?> argument : target to store result of copy
  - count=<?> argument : the amount of data (in bytes) to copy
- sha256sum : finds sha256 hash of a file or data passed in through piping
- md5sum : finds md5 hash of a file or data passed in through piping
- xargs : sometimes , you work with commands that you cannot pipe data into , for instance mkdir . we use xargs to make them pipable
  - -I flag : MUST be provided to act as variable place holder for the data that is getting piped in. the value in front of it can either be {} or a variable name , eg foo
  as an example , lets use ls and xargs to create subdirectories that have files in them with a certain extension: ls ../bin | xargs -I {} sh -c "mkdir -p {};touch {}/{}.sh" . this snippet looks at items under ../bin . we are assuming there are a bunch of files with .sh just laying there is a flat hirerachy and we want to organize them by giving each file it's own parent dierctory. reuslt of ls will get pipid to xargs which runs sh -c ... command, meaning it runs the following using sh shell

- find command is exteremely powerful way to recursively search for files or directories with a pattern.
  - first argument that comes right after find before any other flags is the root directory we want to run search on . for instance find . <...> will search recursively under current directory shell is in now
  - -type flag : it's value can either be d ( to search for directory ) or f (to search for files). leaving this flag cause find to iterate and search over both files and directories. e.g. find . -type f <...> will look for files under current directory
  - -name and -iname flags : specifies a naming _pattern_ . it accepts wild cards. keep in mind that what comes after it must be wrapped in single or double quotes. for instance to search for all .sh files ( shell scripts ) under current directory , run the following find . -type f -name '*.sh'.
  if you want to seach for multiple patterns in names , use iname flag and repeat it as often. If you put a NOT before iname it would exlude it . for instance , the following would search fo all .sh and .go files that do not have _test in their name find . -type f -iname '*.sh' -iname '*.go' NOT -iname '*_test*'
  - exec and execdir runs a command on every file find  returns. these are exteremly powerful and useful. make sure to terminate the command passed to these to with \;
  for instance the following snippet removes gotemp directories which are useless directories created by go toolchain when running go build to compile an application

`sudo find /tmp -type d -name 'go-build*' -prune -exec rm -rf {}\;`

  another example is how to recursively remove all node_modules folder. If you keep those, it will make process of moving or copying a node librayr much much longer and make the folder very large in size. 
  `find . -name "node_modules" -type d -prune -exec rm -rf '{}' \;`
  
  another example is to fix file names. lets say you have files with empty spaces in their name, which isn't good because it makes opening and interacting with them harder since you must encapsulate their path in single or double. first make sure rename is installed by running `sudo apt install -y rename` and then we will use it with sed to replace all white spaces in file names with a `-` .

  
`delimiter="-"; \`
  `find . -depth -name "* *" -execdir rename "s/ /$delimiter/g" "{}" \;`

#### second lesson:

generating random string with coreutils and kernel api

/dev/ , /sys/ and /proc are psudeo-filesystems. they are essentially ways for kernel to expose some api. we would use binaries shipped with gnu coreutils and busybox to generate random data. 

/dev/urandom is a psuedo-file and part of kernel api that returns random data.

the following snippet generates a random string of length 10.

`length="10" ;\`
`head /dev/urandom | tr -dc A-Za-z0-9 | head -c "$length";`

#### third-lesson:

generating random file with dd and kernel api

the following generates a random file based on what you put in size variable. the unit is controlled with bs argument which in this case is equal to 1 MB. we also would use /dev/urandom here.

so to generate a 100 MB random file , use the following snippet

`size=100; \`
`dd if=/dev/urandom of=/path/to/store/random-file bs=1048576 count="$size"`


#### fourth-lesson:

burning/mounting iso image with dd and kernel api

first, find the target usb device using fdisk

`sudo fdisk -l`

analyze the result and find usb key, in many cases it is in /dev/sdb

then unmount the device

`sudo umount /dev/sd<?><?>`

then use the following to make a bootable usb key

`sudo dd bs=4M if=path/to/input.iso of=/dev/sd<?> conv=fdatasync status=progress`

if you just want to checkout the iso file (maybe copy something out of it), you can mount it with the following snippet

`sudo mkdir /mnt/iso/`
`sudo mount -o loop /path/to/ubuntu.iso /mnt/iso`

do not forget to unmount after you are done. unmount with the following snippet

`sudo umount -l -v -f /mnt/iso`

deleting iso folder:
`sudo rm -r /mnt/iso`


Most of what you use, tools such as `ls` `tar` and so on are part of a package called `coreutils` . These files also come through a software called `busybox` which is the swiss army knife tool of Linux since it is statically compiled. For instance, you can modify your android device by pushing `busybox` to it since you can invoke commands such as `busybox ls` or `busybox fdisk`

#### Fifth lesson:

systemd quickstart

you can use systemd to control application lifecycle and how and when it executes. what happens when it fails and so on. its the granddady of all orchestrators and managers such as nomad.
every interaction with systemd must be through root user, i.e with sudo.
everything (any binary) systemd manages is called a unit


- list all units

`systemctl list-units --type target --all`

- adding a unit

first put it under /lib/systemd/system/ and use .service extension
the following can be used as a template for most binaries you would like to
manage with systemd which needs to start after the machine has internet access
and nginx reverse proxy has started. just use the command that starts the program
in front of 'ExecStart'. try to use absolute path.
most of this file is self explanatory. keep in mind that in systemd 
definitions , we also can use '$' . so to have dollar sign in a service file
we are populating with 'cat' , we must escape it , i.e '\$' . 
in case you use an editor like nano you wont have to escape it. look at what's
in front of ExecReload

cat > /lib/systemd/system/my-unit.service <<EOF
[Unit]
Description=Some service
After= syslog.target network.target nginx.target
[Service]
Type=simple
Environment=FOO=bar
WorkingDirectory=/my/working/dir
ExecStart=/my/working/dir/dj-khaled --another-one 'you loyal' \
                                 --another-one 'you genius' \
                                 --and-another-one 'go buy yo mama a house'
ExecReload=/bin/kill -s HUP \$MAINPID
User=root
Restart=on-failure
RestartSec=5
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
KillMode=process
[Install]
WantedBy=multi-user.target
EOF


then you should create a symlink from file to `/etc/systemd/system/multi-user.target.wants/` directory 

`ln -s /lib/systemd/system/my-unit.service /etc/systemd/system/``multi-user.target.wants/my-unit.service`

make sure to reload the daemon and enable the service

- reloading daemon. after adding or modifying an existing service file, which is usually under `/lib/systemd/system/` directory.

`systemctl daemon-reload`

- to enable a service (for instance in case you just added it), after reloading
systemd , run the following

`systemctl enable --now my-unit`

- to quickly check a unit's status and it's stdout , use the following

`systemctl status my-unit`

- in case a unit is acting strangely or you changed config files of the unit (lets say in systemd, in front of ExecStart, you are invoking an application that takes in path to a config file to use for its configuration) or you changed service file and reloaded the daemon already, use the following snippet to restart it 

`systemctl restart my-unit`

- to see stdout of a unit , use the following snippet

`journalctl -e -u my-unit`


