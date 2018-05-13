.. sed_guide documentation master file, created by
   sphinx-quickstart on Sun May 13 15:15:00 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to sed_guide's documentation!
=====================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:



Indices and tables
==================

* :ref:`genindex`

Let file.txt be the file we want to edit.


Basic syntax
==================

```bash
sed 's/one/ONE/' file.txt > new.txt
```

Explanation:
- `s`		Substitute command
- `/../../`	Delimiter (slash is the default, but it can be any character, eg `"|": |..|..|`)
- `one`		Regular Expression Pattern Search Pattern
- `ONE`		Replacement string


 Advanced syntax
==================

- More than one file can be used as input:
```bash
sed 's/^#.*//'  f1 f2 f3
```
- /PATTERN/ is an adress that can be used to give commands to the matching occurences.
```bash
sed '/PATTERN/ p' file
```
- In a bash-script, a quote can cover several commands (instead of using the -e flag)
```bash
#!/bin/bash
sed '
s/a/A/g 
s/e/E/g 
s/i/I/g 
s/o/O/g 
s/u/U/g'  <old >new
```
- You can create a sed interpreter script. Can be run with for example: `mySedScript <old >new` (make it executable with `chmod +x` first)
```bash
#!/bin/sed -f
s/a/A/g
s/e/E/g
```
- Pass agruments inside a shell script:
```bash
#!/bin/sh
sed -n 's/'"$1"'/&/p'
```
- Restrict search to a specific line number (e.g. line 3):
```bash
sed '3 s/[0-9][0-9]*//' <file >new
```
### Patterns:
- deletes all lines beginning with #
```bash
sed '/^#/ s/[0-9][0-9]*//' 
```
- To use a comma instead of a slash, use:
```bash
sed '\,^#, s/[0-9][0-9]*//'
```
- Restrict a substitution to the first 100 lines:
```bash
sed '1,100 s/A/a/' 
```
```bash
sed '101,$ s/A/a/' $=to the end of the file
```
- Ranges by patterns:
```bash
sed '/start/,/stop/ s/#.*//' remove all lines starting with # between start and stop
```

Grouping:
 If you wanted to restrict the removal to lines between special "begin" and "end" key words, you could use: 
 ```bash
#!/bin/sh
# This is a Bourne shell script that removes #-type comments
# between 'begin' and 'end' words.
sed -n '
	/begin/,/end/ {
	     s/#.*//
	     s/[ ^I]*$//
	     /^$/ d
	     p
	}
'
```
- Can be nested, eg first 100 commands:
```bash
#!/bin/sh
# This is a Bourne shell script that removes #-type comments
# between 'begin' and 'end' words.
sed -n '
	1,100 {
		/begin/,/end/ {
		     s/#.*//
		     s/[ ^I]*$//
		     /^$/ d
		     p
		}
	}
'
```
- You can place a "!" before a set of curly braces. This inverts the address, which removes comments from all lines except those between the two reserved words: 
```bash
#!/bin/sh
sed '
	/begin/,/end/ !{
	     s/#.*//
	     s/[ ^I]*$//
	     /^$/ d
	     p
	}
'
```
- If you did not want to make any changes where the word "begin" occurred, you could simple add a new condition to skip over that line:
```bash
#!/bin/sh
sed '
	/begin/,/end/ {
	    /begin/n # skip over the line that has "begin" on it
	    s/old/new/
	}
'
```
You can use the curly braces to delete the line having the "INCLUDE" command on it: 
```bash
#!/bin/sh
sed '/INCLUDE/ {
	r file
	d
}'
```
The order of the delete command "d" and the read file command "r" is important. The first is the "r" command writes the file to the output stream. The file is not inserted into the pattern space, and therefore cannot be modified by any command. Therefore the delete command does not affect the data read from the file. 

"d" command deletes the current data in the pattern space and aborts every further action. Eg needs to be the last command.

### Adding, Changing, Inserting new lines

- This example will add a line after every line with "WORD:" 
```bash
#!/bin/sh
sed '
/WORD/ a\
Add this line after every line with WORD
'
```
- You can insert a new line before the pattern with the "i" command: 
```bash
#!/bin/sh
sed '
/WORD/ i\
Add this line before every line with WORD
'
```
- You can change the current line with a new line. 
```bash
#!/bin/sh
sed '
/WORD/ c\
Replace the current line with the line
'
```
- Can be combined with curly braces:
```bash
#!/bin/sh
sed '
/WORD/ {
i\
Add this line before
a\
Add this line after
c\
Change the line to this one
}'
```
- Add more than one line:
```bash
#!/bin/sh
sed '
/WORD/ a\
Add this line\
This line\
And this line
'
```

- Multi-line patterns


- if `/` is used in the regular expression it needs to be "escaped" with `"\"`: `\/`
- `&` : matches the pattern found eg: `sed 's/[a-z]*/(&)/' file.txt`


Pattern flags:
==================

/g  Global replacement. Matches all occurences on a line, not just the first.
/p  Prints each match on a line
/w  Write to a file. Eg: `sed -n 's/^[0-9]*[02468] /&/w even' <file ` writes to a file named "even" (-n=no print). You can also have ten files open with one instance of sed. Useful for logging, debuging or storing results in temporary files. 
/I  Ignore case. This will for example match abc, aBc, ABC, AbC, etc.
d  delete the matched pattern, eg: `sed '11,$ d' <file ` deletes all lines exept lines 1-10.
!  reverses the restriction, eg: `sed -n '/match/ !p' </tmp/b` matches all lines that doesn't contain the pattern.
q  quits search when condition finished, eg reaches line 11: `sed '11 q'`
r  reads in a file. Eg: `sed '/INCLUDE/ r file.txt' <in >out` will insert file.txt after the line with the word INCLUDE.
It is possible to combine substitution flags. E.g: `sed -n 's/a/A/2pw /tmp/file' <old >new` (note that w have to be the last flag).


Useful flags:
==================

-i  save changes directly to file.txt (-i=in place) (only use this if you've tested first that the output is ok)
-r   extended regular expressions, for example the + symbol (-E on mac)
-e  used to add multiple commands. E.g: `sed -e 's/a/A/' -e 's/b/B/' <old >new`
-f  use a sed script file, eg: `sed -f sedscript <old >new`
- TODO


Regular expressions:
==================

Anchors:
 `^ $` beginning and end of line 

Counters:
 `+` one or more 
 `*` zero or more 
 `?`  zero or one 
 `{n,m}` at least n, max m

Disjunction:
 `.` any single character
 `[]` any character(s) within the brackets

Combination of above:
 `.*` any character, zero or more

Precedence operators:
 `\(\)` for capturing a group. referenced by \1 \2 \3 (up to 9)  in the replacement string

Special characters:
 \s  whitespace characters
 \S  non-whitespace characters
 \w Match a "word" character (alphanumeric plus "_")
 \W Match a non-word character
 \d Match a digit character
 \D Match a non-digit character
 \b Match a word boundary
 \B Match a non-(word boundary)
 \A Match only at beginning of string
 \Z Match only at end of string, or before newline at the end
 \z Match only at end of string
 \G Match only where previous m//g left off (works only with /g)``*

More special characters:
 \t tab (HT, TAB)
 \n newline (LF, NL)
 \r return (CR)
 \f form feed (FF)
 \a alarm (bell) (BEL)
 \e escape (think troff) (ESC)
 \033 octal char (think of a PDP-11)
 \x1B hex char
 \c[ control char
 \l lowercase next char (think vi)
 \u uppercase next char (think vi)
 \L lowercase till \E (think vi)
 \U uppercase till \E (think vi)
 \E end case modification (think vi)
 \Q quote (disable) pattern metacharacters till \E



Useful commands:
==================

Remove lines with whitespace characters:

```
sed "/^\s*$/d" file.txt
```



Sources:
http://www.grymoire.com/Unix/Sed.html 