*Typing a number before a motion or operator repeats it nth times*
   
## Motions

`w` - Move to the beginning of the next word. 

`e` - Move to the end of the next word. 

`b` - Move to the beginning of the previous word.

`ge` - Move to the end of the previous word. 

`0` - Move to the beginning of the line (zero).

`$` - Move to the end of the line. 

`gg` - Move to top of file. 

`shift + g` - Move to bottom of file. 

`line  + shift + g` - Moves you back to the line before you `gg` or `shift g`


## Deleting

`dw` - Deletes FROM cursor to beginning of next word (excluding it's first chr)

`de` - Deletes FROM cursor to the end of the current word (including last chr)

`d$` - Deletes FROM curso to the end of the current line. 

`dd` - Deletes the entire line. 

`x` - Deletes a sinlge element on the cursor

**Deleting a column**
`CTRL v` - To enter visual mode
`j` or `k` - To expand the visual selection down/up of the amount of rows you want to delete in the column
`h` or `l` - To expand the visual selection of the # of columns you want deleted
`d` - To delete the selected columns/rows

## Undo / Redo

`u` - Undo the last command.

`U` - Returns the line to it's original state. 

`CTRL-R` - Redo the last command.

## Moving Text

`p` - puts previously deleted text after the cursor or previously deleted line below the cursor.

## Inserting Text

`i` - Insert text before the cursor

`a` - Insert text after the cursor

`I` - Insert text at the beginning of the line

`A` - Insert text at the end of the line

## Replacing Text

`r[x]` - Replaces element on cursor with x.  

`ce`or `cw` - Deletes FROM the cursor to the end of the word and goes into *insert* mode, to make changes. 

`ciw` - Deletes word on cursor and goes into INSERT mode. Cursor can be anywhere on the word. Remember `c[num][motion]`

`:s/old/new` - Changes the first occurrence of the old word on that line. 

`:s/old/new/g` - Changes the old word globally within the line. 

`:%s/old/new/g` - Changes every the old word globally within the file. 

`:%s/old/new/g` - Changes every the old word globally within the file w/ a prompt to substitute or not. 


## Search

`/ + [search_term]` - Search within the file. 

`n` - move to the next found search term

`N` - move to the previous search term

`CTRL + o` - go back where you came from

`CTRL + i` - go forward

`%` - Finds the **matching** `( { [ ] } )`

## Selecting Text

`v` - Puts you in Visual mode, use motion commands to select text.

## Window Panes

`CAPS (CTRL) c` - Close a window pane

`CAPS (CTRL) q` - Quit a window pane

`CAPS (CTRL) s` - Open a window pane

`[num] CAPS (CTRL) K|J|H|L` - Move to nth window pane up|down|left|right (`[num]` is optional)

## Tabs

`t` - Open file in new tab (new cursor is on a filename)

`gt` - Move between tabs (`go` to `tab`)

