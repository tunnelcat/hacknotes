DOS Cheatsheet

|   |   |
|---|---|
|CD|Change Directory|
|DIR|Lists all files and directories in the current directory|
|TYPE|Displays the contents of a text file|
|CLS|Clears the screen|
|HELP|Provides help information for DOS commands|
|EDIT|The MS-DOS Editor|

You can run executables like so: `SOME.EXE C:\SOMEFILE.TXT`

See the magic bytes? MZ (ASCII representation of the bytes) means it's a Windows or DOS executable.
![](Pasted%20image%2020231205205649.png)

Most common files and magic bytes: 

|   |   |   |
|---|---|---|
|File Format|Magic Bytes|ASCII representation|
|PNG image file|89 50 4E 47 0D 0A 1A 0A|%PNG|
|GIF image file|47 49 46 38|GIF8|
|Windows and DOS executables|4D 5A|MZ|
|Linux ELF executables|7F 45 4C 46|.ELF|
|MP3 audio file|49 44 33|ID3|

Task: replace the magic bytes with AC (41 43) to make the backup executable work on it. Then `ALT+F` to select the file menu, and save.
![](Pasted%20image%2020231205210045.png)
![](Pasted%20image%2020231205210125.png)
![](Pasted%20image%2020231205210200.png)

