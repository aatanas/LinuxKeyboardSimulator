# LinuxKeyboardSimulator

The goal of the program is to create a keyboard simulation. The program is written in C and assembly.

The following functionalities are supported:

- Reading character layout per scan
   codes.

- Reading the mnemonic table.

- Reading a series of scan codes and their interpretation
  
   - Special functionalities for shift, ctrl and alt

- Display the corresponding characters on the screen.

### System description

With every keyboard, key presses and releases are bound to
unique number - scan code (*sc* hereinafter). Keyboard
passes *sc* to the operating system on every press or release
keys, the operating system interprets them and passes them on to applications. 
In this simulation, special keys - shift, ctrl and alt will have fixed *sc*. 
All other keys are listed in the file scancodes.tbl which lists the corresponding 
*sc* for all "lowercase" and "uppercase" letters.



Fixed *sc* values:

| \-   | shift | ctrl | alt |
| ---- | ----- | ---- | --- |
| down | 200   | 201  | 202 |
| up   | 300   | 301  | 302 |



Scan code 400 in this system represents the end of the file.

In addition, this system supports the printing of arbitrary ASCII characters
while holding alt, and printing predefined messages using shortcuts
with the ctrl key.

The program should first read two files that describe the configuration
system - scancodes.tbl and ctrl.map. After that the program starts to be
interactive and the user enters the file name (for example:
test1.tst) which contains an array of *sc* values. These *sc* are interpreted i
the corresponding result is printed on the screen. The keyboard has four modes
work:

- Without control keys - "small" letters read from are printed
   scancodes.tbl.

- Pressed shift - "big" letters read are printed
   from scancodes.tbl.

- Ctrl pressed - the mnemonic specified in is entered
   ctrl.map.

- Pressed alt - ASCII code is entered.

*Note:* with the alt control key, the result is displayed only when
read *sc* to release that key (302).

### Input data

Input data is provided via three separate files:

- scancodes.tbl

- ctrl.map

- test1.tst - the name of this file is entered at the start of the program

#### scancodes.tbl

In this system, the scancodes.tbl file will have a list of characters that
should be bound to *sc*, while the *sc* character is given implicitly -
starts at 0 and increments by 1 for each character. List of characters
"lowercase" and "uppercase" letters are separated by a newline character.

Example file:

```
abcd()1234
ABCD{}!"\#\$
```

This would be a system description file with 10 *sc* . The character 'a' is
described by *sc* 0, character 'b' by *sc* 1, and so on, up to the character
'\$' which is described by *sc* 9 while holding down shift.

The maximum number of *sc* in the system is 128.

#### ctrl.map

In addition to ordinary letters, the program also supports mnemonics, or so-called. shortcuts.
Mnemonics are specified in a special file - ctrl.map - which is of the form:

```
3
q text that is printed when ctrl+q is pressed
d good morning
Q some third text
```

At the beginning of the file, the number N (1 ≤ N ≤ 16) that it represents is stated
how many mnemonics there are. This is followed by an N line where the first character
represents a shortcut, and the rest of the text is what should be printed when
enter that character with ctrl pressed. The text to be printed will be
maximum length 64 characters. The shortcut is specified as an ASCII character.
If the ASCII character is not in scancodes.tbl, then it cannot be accessed
mnemonic.

#### \*.tst

When the system is successfully configured, the program prompts the user to
specifies the name of a .tst file that will contain an array of *sc* values. These
the values are interpreted according to the previous configuration and the result of the interpretation
is printed on the screen. A value of 400 represents the end of the file.

Example file:

```
200
1
300
0
1
0
202
8
7
302
201
3
301
400
```

In addition to the configurations listed in the previous sections, the printout for this one
the file will be:

```
Baba good morning
```

Interpretation:

```
200 - shift down
1 - B
300 - shift up
0 - a
1 - b
0 - a
202 - alt down
8 - 3
7 - 2
302 - alt up (32 is space)
201 - ctrl down
3 - ctrl + d is "good morning"
301 - ctrl up
400 - the end
```

The program works according to the following flow:

1. Reading configuration files and filling global arrays / tables is executed once at the beginning of the program.

2. The program asks the user for the name of the file containing the *sc* values.

3. Scan code values are read one by one.

4. Parsing of *sc* values is done entirely in assembly. C function
which does the parsing has the signature: <br> *int process\_scancode(int scancode, char\* buffer)*

	1. The first argument represents the scancode to be parsed.

	2. The second argument is the buffer in which the parsing result should be placed (typically we will place only one character in the buffer - more characters are placed only when entering the mnemonic).

	3. The return value represents the number of characters written into the buffer.
	
	4. There is only one int variable that will be the return value for the function. Other than that, all logic functions are written in an inline assembly block.

5. After parsing one *sc* value, the resulting buffer is
is printed to the terminal, and the next value is read from the file.

6. When the file is read in its entirety, we return to step 2.

7. If the command "exit" is entered as the file name, the program ends.
