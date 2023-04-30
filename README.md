Download Link: https://assignmentchef.com/product/solved-csc415-assignment-3-the-file-system
<br>
File system designs vary between systems and even between use cases for the file system. System can hold information within blocks, which are the smallest readable unit that address information, and these blocks can be between 512 bytes to even 4 megabytes.Because there are many variations of file systems, designing our own was a challenge. We started off with a design that would link blocks to one another. Doing so would eliminate fragmentation in the volume. We even thought about implementing clusters that can hold a fixed size of blocks. Adding clusters would help keep blocks that belong to one set of information from being scattered around the file system. We didn’t want our design to go off the rails, so we did take inspiration from other systems like having block sizes that are 512 bytes.




Implementing our system did not go as planned. While we did strive to implement the system we planned out beforehand, like many designs changes had to be made. The biggest change we had was dropping our design of a link allocation for blocks. This was due to the fact that when we first started the assignment, we were unsure of how these blocks can easily be read back. Another issue was that if we were to assign a struct data to blocks, blocks would have extra data that is not needed. The design was then changed very briefly to an index allocation for data. This allowed data to take up a full space of a block if needed without any extra information tied to it. The only downside was that an extra block would be needed to store the metadata about a data like its size, filename, etc. But the issue doesn’t end there.

After working through the difficulties of figuring out which allocation method we can work with, we settled on a hybrid of an index/contiguous allocation. This is far different from the original plan of having only a link allocation. To keep reading from the volume simple, data is written to it in contiguous blocks. That means contiguous free blocks just need to be found. After the blocks are found, we just need the start of the free block and we keep writing until we reach the end of the contiguous blocks. This also means that reading is the same way. We just need to find the starting block and since we know the size of the file, the end can be calculated from the file size and LBA size. The hybrid with the index allocation comes into play when it comes to files. Directories can be stored in as little as one block since the only difference between an actual file and a directory is a member variable difference. Files have an extra block associated with it that holds the files metadata and the starting block location of the data.

Issues Along the Way:

Several bugs cropped up along the way, each offering its own problems and requiring its own solutions. An initial issue cropped up in the allocation of the bitmap, which was initially dynamically sized, but this proved problematic as it could potentially grow past the size of its allocated space and begin overwriting or being overwritten. To go along with this, blocks were not being properly allocated at first. This also leads to some blocks being overwritten.

Another issue came from the initial attempts at the cd command. Initial variations simply accepted a single variable to either go up or down one directory level. Going up a level was also initially under the presumption of a linear directory structure, decrementing the current directory and potentially accessing random directories. To fix this, logic was added to search for the parent directory for moving up a level.

This logic for searching got used for other functions, such as copying and moving files within the directories. This then brought about the creation of a searching function, which had some initial issues with how it read in the needed blocks for directories (accidentally just reading random blocks based on the wrong variable).A simple fix and the function could be used for increased readability of the code as searching the file system was abstracted. This function also allowed for reading paths rather than single level variables.

Other issues arose with the size of the bitmap, which was initially too small for even the most minimal of file size, though this was another easy fix. To simplify the allocation of blocks with a bitmap, we chose to just store an array of integers that indicate if a block is free or not. For initial testing of the program, a small bitmap array size was used to go along with a small volume size. This helped to ensure easy readability when the volume file was hexdumped. When the volume size was increased, the bitmap wasn’t able to hold all of the blocks so a segmentation fault would occur. This was easily fixed by increasing the size of the bitmap array to work with the minimum size of the created volume.

Copying in files from the Linux system had initial issues as well. When the function to perform this task was written, the function fget was used. This function works great for text files, but not for files like images or PDFs. When using fget with these types of files, it would either read in nothing or read in some garbage data and shift the actual file in memory. This was confirmed by checking the hexdump of the original file and the copied file. To solve this, fread was used as it can read in memory by memory exactly. This kept the file intact within our file system. And when it was copied back out, that file was not corrupted and worked as it should.

Nested files within directories also offered issues for copying and deleting directories, as this requires digging into the directory and finding all files nested within it and deleting them as well as the requested directory.

Driver and Functions:

The main driver of the program acts akin to the cmd shell, giving a prompt for an input from the command line and acting accordingly. Within the main is this simple command selection, offering twelve possible functions that are described below. To run each command, the input is taken and scrubbed of its newline character before being tokenized and checked against the list of supported commands.

The driver contains a loop with menu items that continues to execute until “exit” is typed in. The first block of code in the loop displays a prompt on the console. Logic was added so that the program will also print out the current working directory like the Linux terminal. When the loop is done, the driver writes the current state of the file system to the volume, frees up allocated memory, and closes the partition system.

The first function to be coded was the blockInit function. This function takes in a pointer to a struct called Volume that contains the information of the volume and the argv array. This function uses the passed in command line arguments to initialize the struct for the volume. initBlock will also take the blocks for the file system and it will clear and initialize each one. After initializing all values for the volume struct and the free blocks, the function writes these to the volume so that the information is stored. Another important part of  this function is that it checks if a volume already exists and it will read back the information so that it can be used.

The function, makeDirectory, was the first function to be coded that operates on the volume. This is similar to the terminal command mkdir and is also used in our project to invoke the function. This function, like many others, takes in an char array and a pointer to a struct called Volume that contains the information of the volume. The function is simple enough with its design. It allocates space for a directory the size of the struct in relation to how many blocks it will take up. It finds contiguous blocks using a function that will be described later and if there’s not enough free blocks it returns. Finally, the function sets the blocks as used in the volume and writes the directory to the volume.

Nano was a little difficult at first to code. This function takes a filename and removes the extension if there is one. From here, getline is used so a user can enter text that can be saved to the volume with the entered filename. Just like makeDirectory, contiguous blocks are searched for for the text and for the block that will hold the metadata for the file. The function then indicates that the found contiguous blocks are not free anymore and then writes the text and metadata block to the volume.

Like nano, the removeFile function had its difficulties as well. It tokenizes a string to get the filename to remove, and then it looks through the use blocks to see if a file makes with that name and is in the current directory. If that file is not found, it returns to the menu. If the file is found however, a struct that indicates that a function is free is written onto the blocks to be removed and zeroes out the block. If the file to be removed also has other blocks associated to it like how the nano function creates a block for metadata and blocks for the file, those associated blocks will be zeroed out also.




ChangeDirectory was pretty simple to code. It takes the name of the directory to change to and sets the current directory variable to that new directory. If a user wants to go back a directory, the function will set the current directory to the parent of the current directory. There is also an option for a user to go all the way back to root like the Linux system. And finally at the end of the function, if none of these change directory options work or no directory was found, it will print out as such.

Printing out files in the current directory was probably the simplest function to make. It loops through the blocks used and if a used block has the current directory as a parent, it will print out. The only extra logic added was that for files, since the extensions were scrube off and saved into a separate char array, the filename is printed out with a dot and then the extension is printed out.

ReadFile functions just like a cat in the Linux terminal. It takes the filename to look for and removes the extension since it’s not really needed. Then it loops through the used blocks and checks if a file with the same name and resides in the current directory exists. If it does not exist the function returns to the menu. If it does exist, it reads the block containing the text into a buffer and then prints it to the console.

The function to move files was more involved than we thought even though the idea of moving a file is simple. We decided that it wasn’t necessary to read the file off the volume to move it, but instead keep the file as it is in the volume and just change the parent of the file. The function first checks if a file exists and if it is doesn’t it returns from the function. If the file does exist, then the function continues  and checks if the directory to move to exists and the same condition happens here as well. Once the file and directory are found, the files parent is set to the found directory. The part that makes this a little more complex is that the mv command on terminal also allows files to be renamed. So extra logic was added to check if the second parameter passed in was a filename. If it wasn’t a directory that was passed in, then it was a new filename passed in. so instead of changing the parent of the file, the function will now change the filename and the extension

Copying files was probably the most complex and difficult function to make. While it did share many similarities to the move file function and nano function, there was still mental juggling on how to actually write this new but copied file. The bulk of the copyFile function is similar to the moveFile function where a file and directory are checked to see if both exist. Where the complexity starts is finding a new location for the copied file. The function looks for contiguous blocks of appropriate size for both the metadata block and the file block. If contiguous blocks cannot be found then the function returns stating that there is no more free space. With the found contiguous free blocks, the function then reads the blocks to be copied into a buffer. It then writes these blocks back to the volume, but it is written to the found location of contiguous blocks. So now the file exists in two locations in the volume.

Finding contiguous blocks was very difficult at first. But after discussing what algorithms can be used to find the blocks, the function became very simple. In our implementation, finding contiguous blocks required a nested for loop. The first loop would go through individual blocks while the inner loop would search a set distance from the first block. For example, if 5 blocks are needed, the outer loop will start at block 0, the inner loop will then go from block 0 to block 4 checking if each of those blocks are free. If not. Then the next iteration will check blocks 1 through 5 and so on. If blocks are found, then the function returns the start location of the contiguous blocks. If nothing is found, a sentinel value of -1 returned indicating that no free blocks were found.

The special function to copy files from Linux was actually simple to code. The difficulty came from debugging the issue where files were read in incorrectly. This function took similar code from nano and even code from the very from the threading assignment for reading files in. the function reads the file into memory and gets the size of the file. It then looks for contiguous blocks for the file and the metadata block for the file. This part was very similar to the nano function. Metadata was set and written to the block and the file was written to a buffer in memory. After that, the file was written to its blocks. The difficulty came from figuring out which function to use for reading. Using fget works perfectly for text documents, but not for other files sucks as audio or images. Using memcpy to copy the file to the buffer was producing garbage or nothing at all. Finally, using fread provided the results we were looking for. It allowed the file to be read exactly into memory and then written to the block.

The second special function to copy files to Linux allowed for us to test how the function to copy from Linux performed. The functions were written one right after the other so that we could test both functions. This function first declares an output file with the name passed in. next the file with the passed in name is checked to see if it exists. If the file does exist, then the file is read into a buffer and then written out to the file declared at the beginning of the function.

Functions (and screenshots):

<strong>mkdir</strong>:​ Make Directory creates a directory within the current directory, named with an additional argument.

<strong>ls</strong>:​ List Directory outputs all the files and nested directories within the current directory.

<strong>nano</strong>:​ Nano allows for the creation of text files within the current directory. These files are named with an additional argument. Following the call for nano, the user is prompted to input text to fill the file.

<strong>cd</strong>:​ Change Directory allows for the movement up and down throughout the directories, using .. to go up a level or a directory name to go down a layer.

<strong>cat</strong>:​ Read File outputs the text of the file given through the additional argument to the command window..

<strong>rm</strong>:​ Remove File allows for the deletion of a file or a directory from the current working directory.

<strong>mv</strong>:​ Move File moves a specified file from the current directory to the specified directory.

<strong>cp</strong>:​ Copy File makes a copy of the specified file within the current directory into the directory specified as the second argument.

<strong>cpfl</strong>:​ Copy from Linux copies files from the current work directory on the Linux machine into the current directory of the file system.

(Above) The current directory (matthew) without linuxfile.txt before cpfl is run

(Below) The current directory (matthew) with linuxfile.txt and reading it out, after cpfl is run

<strong>cptl</strong>:​ Copy to Linux will copy the selected file from the filesystem into the working directory of the Linux system it’s being run on.

(Above) The current directory and command line before cptl is run.

(Below) The current directory, now with testfile.txt within it, after cptl is run.

<strong>help</strong>:​ Help simply calls for a help menu to be printed to the command console. This lists the other commands and the arguments that they take.

<strong>df</strong>:​ Prints out the volume information of the file system

<strong>exit</strong>:​ exit terminal/file

















