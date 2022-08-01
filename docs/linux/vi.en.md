---
title: "Vim"
---

Vim is a very powerful editor that has many commands, too many to explain in a tutor such as this.  This tutor is designed to describe enough of the commands that you will be able to easily use Vim as an all-purpose editor.

### Exiting Vim                                                                             
1. Press the ++esc++ key (to make sure you are in Normal mode).
                                                                            
2. Type	:q! ++enter++
   This exits the editor, DISCARDING any changes you have made.

### Text Editing - Deletion
                                                                
1. Press  x  to delete the character under the cursor.
                                                                                                                               
### Text Editing - Insertion

1. Press  i  to insert text.

### Text Editing - Appending

1. Press  A  to append text.

It moves to end of line in insert mode.

### Save and Quit

1. Use  :wq  to save a file and exit.

### Deletion Commands
                                         
1. Type  dw  to delete a word.

2. Type  d$	to delete to the end of the line.

### Operators and Motion

Many commands that change text are made from an operator and a motion.
The format for a delete command with the  d  delete operator is as follows:

<mark>d and motion</mark>

* d      - is the delete operator.
* motion - is what the operator will operate on (listed below).

A short list of motions:

* w - until the start of the next word, EXCLUDING its first character.
* e - to the end of the current word, INCLUDING the last character.
* $ - to the end of the line, INCLUDING the last character.                
* de - will delete from the cursor to the end of the word.     

Using a Count for Motion:

1. Typing a number before a motion repeats it that many times.
2. 2w  to move the cursor two words forward.
3. 3e  to move the cursor to the end of the third word forward.
4. 0  (zero) to move to the start of the line.

Using a Count to Delete More:

1. Typing a number with an operator repeats it that many times.
2. d   number   motion
3. Type d2w or 2dw to delete two words.

Operating on Line:

1. Type  dd   to delete a whole line.
2. Type  2dd  to delete two lines.

### Undo

1. Press  u	to undo the last commands,  U  to fix a whole line.
2. Now type CTRL-R (keeping CTRL key pressed while hitting R) a few times to redo the commands (undo the undo's).

### Delete Summary 

1. To delete from the cursor up to the next word type:        dw
2. To delete from the cursor up to the end of the word type:  de
3. To delete from the cursor to the end of a line type:       d$
4. To delete a whole line type:                               dd

### Put Command

1. Type  dd  to delete the line and store it in a Vim register.
2. Type	p  to put previously deleted text after the cursor.

### Replace Command

1. Type  rx  to replace the character at the cursor with  x .
2. Another way to replace Type a capital  R  to replace more than one character.
3. Replace mode is like Insert mode, but every typed character deletes an existing character.

### Change Operator

1. To change until the end of a word, type  ce .
2. Type  ce  and the correct word.
3. cc  does the same for the whole line.
4. The change operator works in the same way as delete.  
5. The format is:  c    [number]   motion
6. The motions are the same, such as   w (word) and  $ (end of line).

### Cursor Location and File Status

1. Type CTRL-G to show your location in the file and the file status.
2. Press  G  to move you to the bottom of the file. Type  gg  to move you to the start of the file.
3. Type the number of the line you were on and then  G, ex. 34G

### Search Command

1. Type  /  followed by a phrase to search for the phrase.
2. To search for the same phrase again, simply type  n .
3. To search for the same phrase in the opposite direction, type  N .
4. To search for a phrase in the backward direction, use  ?  instead of  / .
5. To go back to where you came from press  CTRL-O  (Keep Ctrl down while pressing the letter o).  Repeat to go back further.  CTRL-I goes forward.

### Matching Parentheses Search

1. Type  %  to find a matching ),], or } .
2. The cursor will move to the matching parenthesis or bracket.

### Substitute Command

1. Type  ==:s/old/new/g==  to substitute 'new' for 'old'.
2. Type  ==:s/thee/the== ++enter++  .  Note that this command only changes the first occurrence of "thee" in the line.
3. Now type  ==:s/thee/the/g== .  Adding the  g  flag means to substitute globally in the line, change all occurrences of "thee" in the line.
4. To change every occurrence of a character string between two lines,
5. Type  ==:#,#s/old/new/g==    where #,# are the line numbers of the range of lines where the substitution is to be done.
6. Type  ==:%s/old/new/g==      to change every occurrence in the whole file.
7. Type  ==:%s/old/new/gc==     to find every occurrence in the whole file, with a prompt whether to substitute or not.

### Execute External Command

1. Type  :!	followed by an external command to execute that command.
2. :!ls :!pwd for example.

### Writing Files

1. To save the changes made to the text, type  :w FILENAME
2. Now remove the file by typing (Windows) ==:!del TEST==	or (Unix):	:!rm TEST

### Selecting Text to Write

1. To save part of the file, type  v  motion  :w FILENAME
2. Press  v  and move the cursor to the fifth item below.  Notice that the text is highlighted.
3. Press the  :  character.  At the bottom of the screen  :'<,'> will appear.
4. Type  w TEST  , where TEST is a filename that does not exist yet.  Verify that you see  :'<,'>w TEST  before you press ++enter++

### Retrieving and Merging Files

1. To insert the contents of a file, type  :r FILENAME
2. You can also read the output of an external command.  For example, :r !ls  reads the output of the ls command and puts it below the cursor.
3. :r FILENAME  retrieves disk file FILENAME and puts it below the cursor position.
4. :r !dir  reads the output of the dir command and puts it below the cursor position.

### Open Command

1. Type  o  to open a line below the cursor and place you in Insert mode.
2. To open up a line ABOVE the cursor, simply type a capital	O

### Append Command

1. Type  a  to insert text AFTER the cursor.
2. Use  e  to move to the next incomplete word.
3. a, i and A all go to the same Insert mode, the only difference is where the characters are inserted.

### Copy and Paste

1. Use the  y  operator to copy text and  p  to paste it.
2. You can also use  y  as an operator:  yw  yanks one word, yy  yanks the whole line, then  p  puts that line.

### Set Option

1. Set an option so a search or substitute ignores case.
2. Typing ":set xxx" sets the option "xxx".  Some options are:
3. 'ic' 'ignorecase'	ignore upper/lower case when searching
4. 'is' 'incsearch'	show partial matches for a search phrase
5. 'hls' 'hlsearch'	highlight all matching phrases, You can either use the long or the short option name.
6. Prepend "no" to switch an option off.   ==:set noic==

### Getting Help

1. :help w
2. :help c_CTRL-D
3. :help insert-index
4. :help user-manual
5. Type  CTRL-W CTRL-W   to jump from one window to another.
6. Type    :q ++enter++    to close the help window.

### Completion

1. Command line completion with CTRL-D and ++tab++
2. Type the start of a command  :e   
3. CTRL-D  and Vim will show a list of commands that start with "e".

### Comment multiple lines

1. Press the Ctrl + V keys to enable visual mode.
2. Select all the lines you wish to comment out.
3. With the target lines selected, press Shift + I to enter insert mode.
4. We need to insert the pound (#) symbol.
5. Finally, press the ESC key, and Vim will comment out all the selected lines.

### Notes

* :e! -- Discard unsavedchanges in file.
* :sh -- Switch to shell temporary, copy something. ==ctrl+d==  return to vi file.
* X -- Remove left side chars.
* D -- Remove everything from cursor right side.
* dG -- Remove everything below
* d1G -- Remove everything above
* :set nu -- Set numbers
* :set nonu  -- Remove numbers
* :11,15d -- Remove everything between 11 to 15
* G -- last line
* 1G -- first line
* 60G -- to go to line
* L -- last line of display
* H -- top line of display
* M -- middle line
* cc -- Remove line and insert
* C -- Remove everything from cursor right side and insert.
* J -- Join 2 lines
* :1,5 co 9 -- after 9 th line 1 to 5 lines copied
* :4 m 2 -- after 2nd line 4 line will be moved.
* shift + o -- new line on top of cursor.
* :set ic -- ignore case sensitive
* :set noic -- case sensitive
