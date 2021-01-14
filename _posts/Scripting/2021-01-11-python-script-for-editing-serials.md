---
title:  "Python script to edit serial numbers in hex files."
categories:
  - Scripting
tags:
  - python
  - production
  - automation
  - hex
  - serial number

author_profile: false
---
###### INTRODUCTION
Scripting is something that helps engineers of all types (software/test/electrical/etc). It truly can be done in just about any language you are familiar with, but I do feel that some languages are better than others and interpreted languages are one of the best options. However, much like any other engineering task, the best language to use can really depend on what languages you are familiar with and what end result you want to achieve. For this task I just needed a script that I could access from bash and I feel like python is the perfect language for such a task. If I needed a GUI for production staff to utilize, I might go with a .NET language because my familiarity with Python based GUI apps is nil to none.

The goal of this project is to replace a 10 ascii character serial number with a string provided to the script. The final code for this project can be found over on my [GitHub][serialhexedit-codebase] and introduces a few extra options that we won't discuss here, but would be a good exercise on expanding functionality.

###### PREPARING MICROCONTROLLER CODE
In order to get this to work we reserve a location for our character array in the RAM of our microcontroller. The full example linker file is [here][macro9pad-linkerfile]. First we want to carve out the actual memory section in our memory list.
According to the datasheet our memory section should look something like this:
```c
MEMORY
{
  rom      (rx)  : ORIGIN = 0x00000000, LENGTH = 0x00004000
  ram      (rwx) : ORIGIN = 0x20000000, LENGTH = 0x00001000
}
```
Then we will want to carve off a few areas for specific purposes. Below we have carved out a future bootloader area, but currently set its length to 0. Next we have our normal ROM space from 0x0000 to 0x3EF4. Then we reserve 10 bytes (0xA) for the serial. Next I have an eeprom emulation area at 0x3F00 with a length of 0x0100. Finally we have our RAM section mapped. *NOTE* These sections do not use the full 0x4000 of ROM available and thats okay. I do not need it and I don't want to change it now that I have units in the wild with this code.
Here is what we end up with:
```
MEMORY
{
  boot     (r)   : ORIGIN = 0x00000000, LENGTH = 0x00000000
  rom      (rx)  : ORIGIN = 0x00000000, LENGTH = 0x00003EF4
  serial   (rw)  : ORIGIN = 0x00003EF4, LENGTH = 0x0000000A
  eep	   (rw)	 : ORIGIN = 0x00003F00, LENGTH = 0x00000100
  ram      (rwx) : ORIGIN = 0x20000000, LENGTH = 0x00001000
}
```

The very next thing we need to do is tell the linker that we should keep this space free even if we don't use the variable in our code. We do this by creating a section in the linker file for the memory we carved away previously.
```
SECTIONS
{
	/* eeprom sim area */
	.eepromBlock :
	{
		KEEP(*(.deviceProfileSection))
	} > eep

    /* serial number section */
    .serialNumber :
    {
        KEEP(*(.serialNumberSection))
    } > serial

    .text :
    {
    ...
```
We tell the linker to keep the serialNumberSection even if not referenced by code and then tell the linker to put that into the "serial" MEMORY area. Now we have the section correctly reserved and if you compiled you should be able to see that. Next we should create an array of ascii characters at serialNumberSection to represent our serial number. For this I created the file, [serialnumber.c][m9p-serialnumberdotc] and put the following code into it:
```c
const char __attribute__ ((section (".serialNumberSection"))) DeviceSerialNumber[DEVICESERIALNUMBERLENGTH] = 
{
	"0123456789"
};
```
This simple bit of code creates a const char array DeviceSerialNumber of "0123456789" and stores it at the serialNumberSection we created previously (DEVICESERIALNUMBERLENGTH is defined in the header as 10).
###### Understanding the Hex File
With all of this done, we can compile our code and we get this very interesting section towards the end of our hex file:
`:0A3EF40030313233343536373839B7`
<br />
Lines in an intel hex file are quite easy to read. They begin with a start code which is a single colon character (":"). Then two hexadecimal characters describe a byte that represents the length of just the data in this line, here it is 0x0A or 10 in decimal. The next four characters describe the two bytes that represent the location in memory for this line of data. These hex characters should look familiar because 0x3EF4 is the origin we specified in our linker for the serial number to exist at. Following the address bytes is a byte that describes the type of information on this line. Here we see 00 (0x00) which tells us that this line has DATA on it. You should also recognize the next 10 bytes (0x30313233343536373839) as the ASCII representation of our serial 0123456789. If you don't know exactly how characters are converted to ASCII, you can see a table [here][ascii-table]. Finally there are two characters to represent the checksum byte. This is an 8 bit checksum, where we add up all bytes then take the 2's complement. If you are unfamiliar with 2's complement, you will take the 8 bit value and invert it (this is 1's complement) then add 1 to the sum. This gives us a byte that when added to all the other bytes creates a byte sum of 0. In this example (0x0A + 0x3E + 0xF4 + 0x00 + 0x30 + 0x31 + 0x32 + 0x33 + 0x34 + 0x35 + 0x36 + 0x37 + 0x38 + 0x39 + 0xB7 = 0x400) and we only care about the single byte and nothing that overflows from there (0x400 & 0xFF = 0x00).

While this gives you the basics of reading a simple data line in a hex file there are more complexities to them and if you want to get a broader understanding you can start [here][intelhex-explained].

###### BEGINNING OUR SCRIPT
The main purpose of this script is to locate a specific line in the hex file based on its memory address and then replace it with a generated valid line. This means that we need to be able to open/copy/save files as well as parse the files for text content, convert strings to ascii, and calculate checksums. When I start any script like this, I just focus on base tasks and build from there. How do I open a file and then save it? How do I scan through the file and how do I search the lines for specific text? And many more as I continued to create a script that did what I needed to. Starting at the simplest place and building logically is a great way to build large projects. I don't really need to take in arguments from the command line until I am closer to the end and focusing on it too early could actually cause me to have to rework more code than I should have to later on.

So lets place this [hex file](/assets/files/serial-example.hex) into a folder for us to work in. Then we can create a file serialTest.py in the same folder. Lets put some really simple code in this file to simply open our serial-example.hex and print the first line.
```python
with open("serial-example.hex", 'r') as inputWriter:
    print(inputWriter.readline())
```
these two simple lines will open our hex file with read-only ('r') attribute. This uses pythons 'with' block to have exception catching if we want. This simple script will print the first line of our hex file to the console:
`:10000000480800202901000025010000250100000A`

The next big step is to take our input hex file and copy every line to a new hex file that we will name "output.hex". All this takes is 3 simple lines of code under our previous withblock:
```python
with open("serial-example.hex", 'r') as inputWriter:
    with open("output.hex", 'w') as outputWriter:
        for line in inputWriter:
            outputWriter.write(line)
```
This opens the input file with the readonly flag and then opens the output file with the write only flag (creating the file if it doesn't exist already). Finally the last couple lines say that for every line in the input file, write that line to the output file. Now the next simple task is to locate our memory location 0x3EF4 and, for the time being, we can just replace it with a simple line like "INSERTHERE". Python makes looking for substrings in strings easy using the "in" keyword. So lets replace our "for line" code with the following:
```python
for line in inputWriter:
            if "3EF4" in line:
                outputWriter.write("INSERTHERE\n")
            else:
                outputWriter.write(line)
```
Now if we open output.hex we can scroll to the bottom and clearly see where the `:0A3EF40030313233343536373839B7` line is replaced with "INSERTHERE".

Next we are going to take a hardcoded serial string and turn that into a valid intel hex line to insert instead of our placeholder "INSERTHERE". We will work with a hardcoded `serialString = "0123443210"` and convert it to string of ascii bytes to use. The most complex part of this step is going to be creating the checksum. Remember we need a byte that overflows to a perfect 0x00 when summed with every byte in the string. The easiest way to start is to combine the ascii serial bytes with what needs to exist at the beginning of the string (data length + memory location + record type). Then we will use a library called wrap that we can import from textwrap. This will allow us to wrap this single string every 2 characters giving us an array of bytes described with two hex characters. We can convert each hex byte of the array to integers and add them all together using this sum to calculate the checksum value. We will convert that checksum value to a string of 2 digit hex characters. Finally using the string we originally constructed before wrapping, we will prepend a colon (":") and append the checksum. This whole process will create a new valid line for our hex file that looks like this `:0A3EF40030313233343433323130D0`. We will use this line instead when we locate the serial line in our hex file. Our script at this point should look something like this:
```python
from textwrap import wrap

# Hardcoded serial number string
serialNumber = "0123443210"

# Convert to ascii hex and create new line for hex file
serialString = bytes(serialNumber, 'ascii').hex()
replacementLine = "0A" + "3EF4" + "00" + serialString

# Calculate checksum and finish new line
wrappedBytes = wrap(replacementLine, 2)
sumValue = 0;
for value in wrappedBytes:
    sumValue += int(value,16)
checksumValue = (-(sumValue % 256) & 0xFF)
replacementLine += str.format('{:02X}', checksumValue)
replacementLine = ":" + replacementLine + "\n"

# Copy input file to output file, overwriting serial line
with open("serial-example.hex", 'r') as inputWriter:
    with open("output.hex", 'w') as outputWriter:
        for line in inputWriter:
            if "3EF4" in line:
                outputWriter.write(replacementLine)
            else:
                outputWriter.write(line)
```

So now we really have something going with our script. If you are confused on what anything is doing you can debug the script or just start adding in print() commands in different places to assist with understanding the program flow. The last thing we really have to accomplish is to take in command line arguments because we aren't going to want to use the same hardcoded serial number every time. We are going to import getopt for accepting commandline arguments into our script. For the sake of simplicity I am only going to write code for an argument to accept a serial number passed in. This single argument will create a script that you can change a couple of variables in for each project and then call with a provided serial number and output your program hex file. If you want to expand on that you can visit this projects github and look at the python script there. It has many arguments that let you use the same script on varying projects, such as specifying input hex, output hex, serial number, incrementable serial file, and memory location. We will use "-s --serial" as the short and long form of our serial argument. We must check that the serial argument is passed, verify it's length, then use it with the above code to create a replacement line in our hex file. We are just going to remove our hardcoded serial string and then add a little bit of code to the beginning of our script, in order to get our starting string from user input. The whole script should now look like this:
```python
import sys
import getopt
from textwrap import wrap

# Set up initial variable values
serialNumber = ""

# Set up possible arguments
short_options = "s:"
long_option = ["serial"]
full_cmd_arguments = sys.argv
argument_list = full_cmd_arguments[1:]
arguments, values = getopt.getopt(argument_list, short_options, long_option)

# Parse arguments
for current_argument, current_value in arguments:
    if (current_argument) in ("-s", "--serial"):
        serialNumber = current_value

# Validate arguments
if (serialNumber ==""):
    sys.exit("Must pass serial number argument.")
if (len(serialNumber)!=10):
    sys.exit("Serial must be 10 characters.")

# Convert to ascii hex and create new line for hex file
serialString = bytes(serialNumber, 'ascii').hex()
replacementLine = "0A" + "3EF4" + "00" + serialString

# Calculate checksum and finish new line
wrappedBytes = wrap(replacementLine, 2)
sumValue = 0;
for value in wrappedBytes:
    sumValue += int(value,16)
checksumValue = (-(sumValue % 256) & 0xFF)
replacementLine += str.format('{:02X}', checksumValue)
replacementLine = ":" + replacementLine + "\n"

# Copy input file to output file, overwriting serial line
with open("serial-example.hex", 'r') as inputWriter:
    with open("output.hex", 'w') as outputWriter:
        for line in inputWriter:
            if "3EF4" in line:
                outputWriter.write(replacementLine)
            else:
                outputWriter.write(line)
```
When you run the script with `python3 serialTest.py` you will now get `Must pass serial number argument.` as an output. If we try something like `python3 serialTest.py -s 1243` we will get an output of `Serial must be 10 characters.` on the terminal. Finally if we enter `python3 serialTest.py -s 1233211230` our script will complete without error and when we open our hex file we see our 0x3ef4 memory line changed to `:0A3EF40031323333323131323330D2`.

###### CONCLUSION
This is the simplest form of this script or what I would consider a least viable product (Remember products aren't just things you create to sell, products are anything you create that is utilized by yourself or someone else). You could then call this script from other scripts that track serial numbers. Maybe assembly workers will scan barcodes, qr codes, or manually enter data that is stored in a database when they create this serial and program your device. You could then tie the serial in this database to customers when they purchase a device and shipping sends out an invoiced item. Many different solutions are possible and they can be anything under the sun really. This is why the full blown script even has a mode where it just tracks the last used serial in a file and increments it. Automation scripting allows you to streamline your production process whether you are 1 person at your home or an engineer at a company with many employees. The oppurtunities are endless!

[serialhexedit-codebase]: https://github.com/zlittell/MSF.SerialHEXEdit
[macro9pad-linkerfile]: https://github.com/zlittell/Macro9PadFW/blob/master/src/startup/samd11d14am_flash.ld
[m9p-serialnumberdotc]: https://github.com/zlittell/Macro9PadFW/blob/master/src/serialnumber.c
[ascii-table]: http://www.asciitable.com/
[intelhex-explained]: https://www.kanda.com/blog/microcontrollers/intel-hex-files-explained/