---
tags: [LA]
title: 'VIM: A Guide To Using VIM'
created: '2020-06-14T17:20:15.548Z'
modified: '2020-06-14T20:20:22.212Z'
---

# VIM: A Guide To Using VIM

**Disclaimer:** These are my notes taken from watching Linux Academy's _Vim - The Improved Editor_ course. I keep them for reference.

## Why VIM

VIM is good for working in terminals in servers. Obviously if you're editing text in other contexts this isn't necessary, but for working with servers, VIM is great for getting things done. All the commands given here are general but cover a lot of scenarios you will find yourself in when doing editing in a server. This guide doesn't cover everything but gives you a pretty good introduction. 

## VIM vs VI

VI: Derived from old unix based editors. Much more limited that VIM.

VIM: Improved editor. Everything available in VI is available in VIM and more. Multi level undo and redo in VIM is a popular feature that VI didn't have. You can split the screen to edit multiple files as well as plugin support. More plugin info on www.vimawesome.com

A minimal Linux installation would probably have VI only, but you can easily install it using your distros package manager.

There's a GUI version of vim: `install gvim`
But get to learn the CLI version.

## Creating Files

`vim test.txt` will create a new test.txt file if it doesn't exist and allow you to edit it.

## VIM Modes
When inside VIM on the CLI, there are different modes that VIM could be in. 

1) command mode: this is when you first enter VIM, you are not editing anything. Any command you enter here doesn't edit the document. The keyboard in this mode is assigned to different macros (like x deletes a character). Keys are assigned to do things rather than display things.

2) insert (edit) mode: press `i` to enter edit or insert mode to be able to actually edit the text in the file. 

To get out of insert mode, hit the `esc` button, then `:` followed by `q`(quit without save) or `wq` (save and quit)

## Navigating Your Created File

IN COMMAND MODE:
Moving around with the keyboard alone:

You could use the arrow keys to move up, down, left, right, but in command mode, `h,j,k,l` moves you in those directions as well. 

Additionally, if you want to move one word at a time:
`w` moves your cursor ahead by a word, `e` moves to the end of a word, `b` takes you to the beginning letter of the next word. 

If you want to go directly back all the way to the beginning, `0` goes to the beginning, `$` takes you to the end. To skip down or up paragraphs, the square brackets can do that. 

If you are editing code and want to know which opening or closing brackets belong together, press `%` and it'll take you to the matching pair. 

If say you want to go 5 words forward: hit `5` then `w`. So basically a combination of numbers and directions.

## Manipulating Text

To enter insert mode, enter `i` when in command mode. 

IN COMMAND MODE:

If you want to insert at the end of a line, hit `a` it'll append and you can insert at the end of the line.

If you want to insert on a new line after where your cursor is, hit `o`

If you want to replace a character, say replace space with comma, put the cursor on the space, press `r`, then comma.

To undo, hit `u`. `x` deletes one at a time.

To redo, hit `.` would basically reinsert the last thing you wrote or rewrite what you deleted.

The `d` key alone does nothing but combined again with another key it could:

- delete an entire line no matter where your cursor is by hitting `d`, then `d` again
- `d` then `w` deletes a word
- `d` then `0` (zero) will delete from the point where your cursor is and everything before it (beginning of line)
- `d` then `$` will delete from the point where your cursor is and everything after it (end of line)


## Copy, Paste, Search, Replace

In COMMAND mode:

copy: where your cursor is, hit `y` twice to "yank the line"
paste: go where you want to paste and hit `p` for paste

To copy a portion, you use mark mode:
hit `v`, it'll allow you to highlight what you want to copy, hit `y` to insert it into the copy buffer, then hit `p`

To indent: `>` twice (8 spaces tab), hit `<` twice to remove indent

To search and replace: hit `/`,it'll show at the bottom,this is a top down search, type what you want to search for, to go to next iteration, hit `n`. To go back, hit `N`. Instead of `/` hit `?` to do bottom up search

From there hit the `:` to execute command: %s/ ex/EX/g
%s means replace ex with EX in every instance (indicated by g), put gc forces vim to ask you for confirmation

## Executing External Commands

You can make VIM execute a bash command without leaving VIM. So you can `ls` while you're in VIM, shows you the result then brings you back to VIM:

In command mode:
press `:` to indicate you want to enter a command, all together it would be
`: ! cat /etc/os-release` this will show you the OS info as if you ran it in a normal terminal and not VIM. Continue and you'll be brought back to VIM. So you could do this to check stuff quickly or whatever while you're on VIM (happens a lot).

Even cooler is you can use this to read in values so you could do: `:r ! cat /etc/os-release` and this time it will copy the result of that command INTO your vim file! the `r` means `read`.

Another example is say you want to record the number of files in a directory:
`:r ! ls -al /etc | wc -l`
It'll return the number of files and store it in your VIM file.

## Files and Buffers

Use buffers to work with multiple files.

To save files:
- :wq means write and quit (save and quit)
- :q! to quit without saving
- ZZ means same thing as :wq
- to save changes to ANOTHER file, use `:saveas newfile.txt`

To add a new buffer:
- :bad myfile.txt means buffer addres and then supply the file to the buffer
- to switch to next buffer `:bn` buffer next or switch buffer, `:bp` goes to previous file. You can use `ctrl 6` it goes back and forth.
- `:f` tells you what file you are in
