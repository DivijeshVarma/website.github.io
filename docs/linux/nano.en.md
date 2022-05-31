---
title: "Nano"
---

GNU nano is an easy to use command line text editor for Unix and Linux operating systems. It includes all the basic functionality you’d expect from a regular text editor, like syntax highlighting, multiple buffers, search and replace with regular expression support, spellchecking, UTF-8 encoding, and more.

At the bottom of the window, there is a list of the most basic command shortcuts to use with the nano editor.

All commands are prefixed with either ^ or M character. The caret symbol (^) represents the Ctrl key. For example, the ^J commands mean to press the Ctrl and J keys at the same time. The letter M represents the Alt key.

* To start nano in very specific line number:
nano +16 textfile

* To start nano in view only mode:
nano -v textfile



### Cheat Sheet

* `Ctrl+g`  Help
* `Ctrl+o`  Save file
* `Ctrl+x`  Exit
* `Ctrl+w`  Search text
* `Ctrl+k`  Cut text
* `Ctrl+u`  Paste text
* `Ctrl+_`  Goto specific line Number
* `Ctrl+r`  insert existing file
* `Alt+a`  `Alt+^`  Select text or Mark set and copy text
* `Ctrl+u`  To paste data which you copied
* `Alt+\`   Move to Top of File
* `Alt+/`   Move to bottom of File
* `Ctrl+Shift+v`  To paste text from Outside
* `Shift+→` To select text
* `Alt+u`   Undo changes
* `Alt+e`   Redo changes
* `Ctrl+a`  Move to the beginning of a line
* `Ctrl+e`  move to the end of a line
* `Ctrl+y`  Start of file
* `Ctrl+v`  End of file
* `Ctr+\`   Search and replace
* `Alt+del` Delete current line 


### List Commands

* You can get a list of all commands by typing Ctrl+g.

### Editing Files

* To move the cursor to a specific line and character number, use the ==Ctrl+_==  command.

### Searching and Replacing

* To search for a text, press Ctrl+w, type in the search term, and press Enter. The cursor will move to the first match. To move to the next match, press Alt+w.
* If you want to search and replace, press Ctrl+\. Enter the search term and the text to be replaced with. The editor will move to the first match and ask you whether to replace it. After hitting Y or N it will move to the next match. Pressing A will replace all matches.

### Copy Cut and Paste

* To select text, move the cursor to the beginning of the text and press Alt+a. This will set a selection mark. Move the cursor to the end of the text you want to select using the arrow keys. The selected text will be highlighted. If you want to cancel the selection press Ctrl+6.
* Copy the selected text to the clipboard using the Alt+6 command. Ctrl+k will cut the selected text.
* If you want to cut whole lines, simply move the cursor to the line and press Ctrl+k. You can cut multiple lines by hitting Ctrl+k several times.
* To paste the text move the cursor to where you want to put the text and press Ctrl+u

### Saving and Exiting

* To save the changes you’ve made to the file, press Ctrl+o. If the file doesn’t already exist, it will be created once you save it.
* To exit nano press Ctrl+x. If there are unsaved changes, you’ll be asked whether you want to save the changes.

