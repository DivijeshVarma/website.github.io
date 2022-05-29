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
