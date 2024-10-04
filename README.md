# Head-and-Uniq-for-xv6
Implementation of Head and Uniq commands in xv6
Environment :
-------------
Project is done in xv6 which is installed in ubuntu in virtualbox.
New files created for this project are:
->uniq.c
->head.c
->uniqKernal.c
->headKernal.c

Modified and done implementation in below files for Kernal mode:
->syscall.h
->syscall.c
->sysproc.c
->usys.S
->user.h
->defs.h
->proc.c
Note:defs.h and proc.c are used only in uniqKernal.And all other 5 files are used for both uniq and head implementation in kernel mode.

Used text files for testing:
OS611example.txt
uniq_test.txt
test_head.txt

Process/Commands used:
----------------------
Head | Usermode
----------------
1.make clean
2.make
3.make qemu
4.head filename
5.head -n numberoflines filename or  head --lines numberoflines filename
6.head -n numberoflines filename1 filename2  (multiple files when given at once)
7.head (when no filename is given, it takes input from console and prints out in console)

Head | Kernelmode
----------------
1.make clean
2.make
3.make qemu
4.headKernal filename
5.headKernal -n numberoflines filename or  head --lines numberoflines filename
6.headKernal -n numberoflines filename1 filename2  (multiple files when given at once)
7.headKernal (when no filename is given, it takes input from console and prints out in console)


Uniq|Usermode
-------------
1.make clean
2.make
3.make qemu
4.uniq filename
5.uniq (when no filename is given, it takes input from console and prints out in console...double enter to terminate)
6.uniq -c filename
7.uniq -d filename
8.uniq -i filename
9.cat filename | uniq

Uniq|Kernelmode
-------------
1.make clean
2.make
3.make qemu
4.uniqKernal filename
5.uniqKernal (when no filename is given, it takes input from console and prints out in console...double enter to terminate)
6.uniqKernal -c filename
7.uniqKernal -d filename
8.uniqKernal -i filename
9.cat filename | uniqKernal


References:
===========
1.https://medium.com/@mahi12/adding-system-call-in-xv6-a5468ce1b463
2.https://www.cse.iitd.ernet.in/~sbansal/os/previous_years/2011/xv6_html/syscall_8c.html#662eedd65f3e2165093842b80e3bc024
3.https://www.geeksforgeeks.org/uniq-command-in-linux-with-examples/
4.https://www.geeksforgeeks.org/xv6-operating-system-adding-a-new-system-call/
5.https://www.youtube.com/watch?v=-RFjRqXRAGY&list=PLG5M71nRqrauttV-0AGo6dCvUNuMR3MbO&index=6


//Head User mode:
-------------------

Implementation in User mode in xv6 OS
> Features implemented 
	- Printing the first 14 lines by default of the specfied files.
	- If the execution statement has only one file name. Then head is execution on that file alone.
	- Otherwise, the console shows the output of the each file with the heading before printing the contents of the file. 
        - If no filename is given, it takes input from console and prints output on the console.
        - Implemented to print specific number of lines using options -n || --lines for the given input file.(for multiple files also implemented).
        
	
> Code and logic
	- file name : head.c
	- This program consists of a main function and a user defined head function
	- main function reads the input [file name and options] from the command line. 
	- Then checks if there are any arguments provided. If there are no arguments then it will read from the console and write to the console.
	- If there are any options provided it checks if they are valid. If the options option is invalid the error message is displayed.
	- Used strcmp function to compare the options.
	- If we provide only one argument (file name). Then, if the system is able to find that file in the directory head command writes the output on the console.
	- If we provide an option [-n || --lines] and the number of lines then the head write specified number of lines from the each file respectively.
	- Then a message is displayed saying "Head command is getting executed in user mode".
	- A while loop is used to run the head on each file to display the output.
	- An if statement is used to check for the number of arguments/ file names. To write the output for each file individually with the file name before starting.
	- Then we open the file and pass fd as the first argument. 
	- Then the user defined head function is called with the parameters as fd - file descriptor and number lines to write on the console.
	
	=>In the head function
	- Using fd we read the file, which returns the number of bytes in it, that is stored in readInput variable. 
	- We also declared writeLines and byteRead as two integer variables.
	- writeLines to track number of lines are writen on the console.
	- byteRead to check the new line character to calculate the count of lines. Both are initialized to 0
	- while loop breaks if (byteRead < readInput) && (writeLines < num_lines) condition is failed.
	- ie., either number of bytes count is morethan the size of the readInput or if the count of the writeLines is morethan the num_lines [Number of lines to write].
	- Within the while loop each byte is written and compared to new line character to count the number of lines, also byteRead is incremented to keep track of the     	  total numberof bytes written on the console.
	- Finally fd is closed and since the return type is void noting is returned.
	- Controlled now is in the main and finally program is exited. 
	- On the console new shell process is invoked.
	
> In Makefile
	- In UPROGS we add the executed file name with '_' -> _head\
	- In extras we update with the head.c\ file
	
	
Head Kernal Mode:
-----------------
- To implement the head command in Kernal mode using system calls we have to update five files. Three for the system calls and two files for the user program.
- The input is taken from the user program through the console/ terminal and passed to the system program via argint for integer input and argstr for string input. 
- First we add a registery input in syscall.h "#define SYS_get_headK 22"
- Then create a pointer to this registery "[SYS_get_headK] sys_get_headK", is created in syscall.c and also declare system call function prototype as well "extern int sys_get_headK(void);" 
- Then create the function defination to the same system function in sysproc.c with the same return type and arguments. We ususlly create the int to void type function to match the datatypes in the call stack of "static int (*syscalls[])(void){...}"
- We write the head logic in sysproc.c of the function prototype that we declared in syscall.c. 
- Arguments are passed to this sys call through the main function or user function. User function acts as an interface for the system calls as system functions cannot be access directly as they reside in the kernal for the security reasons. 
- This is the reason for using the user function to pass the arguments. 
- Arguments passed are recieved in the same order and with the same datatypes as a pointer in the system function using the xv6 inbuilt argint and argstr for int and str respectively.
- As we are using the user function as a interface we have to update it in the user.h under system calls.
- We usually open the file and read it in the main function as we cannot import the user.h in the sysproc.c because, sysproc.c is only for the system calls. Therefore, we pass the file data as a buffer and size of the buffer and any other related options or parameters from headKernal.c file. 
- We also update the function in the usys.S as this is the assembly meta data for the user functions to let the OS know that this is a legal user function.
- Finally we update the Makefile in two places UPROGS and EXTRAS. In UPROGS we add the object code/ byte code file name and in EXTRAS executable code or .c file.

> In Makefile
	- In UPROGS we add the executed file name with '_' -> _headKernal\
	- In extras we update with the headKernal.c\ file




//uniq User mode
-----------------
Code and logic
	- file name : uniq.c
	- This program consists of a main function and a user defined tolower and len functions.
	- main function reads the input [file name and options] from the command line. 
	- Then checks if there are any arguments provided. If there are no arguments then it will read from the console and write to the console.
	- Iterating 'for' loop with length argc to store all the arguments in param character array.We are checking if any options such as c,d or i given and storing in      	  option[0].
	- If there are any options provided it checks if they are valid. If the options option is invalid the error message is displayed.
        - Opens a file using 'open' and it stores its returning value in fd variable(file descriptor) . 
        - If the file is not there it will throw error "Error: could not open the file".
        - If file is there, it will read file using 'read' inbuilt function .
        - Then a message is displayed saying "Uniq command is getting executed in user mode,".
        - Used 2 while loops.One is basically to read character by character till the '\n' (end of line) and other outer while loop is used to perform uniq operations  	  based on input/arguments given in command line.
	- Used strcmp function to compare the options.
	- If we provide only one argument (file name). Then, if the system is able to find that file in the directory uniq command writes the output on the console
          (prints all lines removing adjacent duplicate lines).
	- If we provide an option -c then the count of each repeating line and its lines will be printed on the console.
        - If we provide an option -d then the duplicate lines one per group will be printed on the console.
        - If we provide an option -i then the case is ignored and prints all the lines removing adjacent duplicate lines on the console.Written 'tolower' function to 		  change all string characters to lowercase.And used 'len' function to find length of the string.We are first checking current line and previous line length and 	  then changing to lowercase then comparing currentline and previous line with 'strcmp' function.

	
> In Makefile
	- In UPROGS we add the executed file name with '_' -> _uniq\
	- In extras we update with the uniq.c\ file

> Features implemented 
	- If the execution statement has only file name. Then uniq will print all lines removing duplicate lines.
	- Implemented -c to get count of adjacent duplicate lines along with its  line.
        - Implemented -d to print duplicate lines one per each group.
        - Implemented -i to print all lines removing adjacent duplicate lines and ignoring case.
        - Works cat filename | uniq.


// uniq Kernal mode
--------------------
- To implement uniq in Kernal mode I use system calls in xv6.
- For this implementation I modify 7 files. 
- 2 User files : user.h - user-level function prototypes are defined and usys.S - for assembly level process and meta data. 
- 3 system files: 1 registry file syscall.h, 1 creating system process prototype function and creating the pointer to the registry - syscall.c, 1 system process related file sysproc.c : In xv6, proc.c is the file responsible for managing processes. It contains code for creating, scheduling, and controlling processes, as well as handling process-related system calls and context switching.
- 2 proccess [For other processes apart from the system processes] but user functions cannot be called here. 1 proc.c and other is 
	- In uniq command implementation I implemented logic in this file as I cannot call the other processes except sys processes in syscall.c.
	- In proc.c I created a uniq_internal function and called functions otherthan, sys processes and user functions in the process of implementation. 
	- I needed strcpy - str copy, 
			   strcmp - str comparision, 
			   len - to find the len of the str,
			   inttoStr - converting the int to str as I am using the filewrite function to print the output to the console.
	- These function prototypes are declared in defs.h.
		defs.h in xv6 is a header file that defines various data structures, constants, and function prototypes used throughout the operating system. It serves as a central repository for essential definitions and declarations needed by multiple parts of the kernel and user-level programs.

- Finally Make file to make a system call through the user functions/ user space as we cannot directly access the kernel functions/ kernal space.

> Function flow:
	- Arguments from the console are read and processed through the main file to the syscall via argint, argstr for int type and str type respectively as a pointer
	- In this implementation we used get_uniq(char, char, int) as a user level function to sys_get_uniq(void) as system level fun. 
	- Since we cannot call user processes in syscall.c we pass the same parameters to uniq_internal(char* , char*, int) in proc.c file. Where we have liberty to create new processes and make them isolated. 
	- This function doesn't return any this as it is used to write the output in the console. But, sys_get_uniqK(void) returns 1 for a successful execution and get_userK(char, char, int) also returns 1 for the successful execution as the same return value of the syscall.

> In Makefile
	- In UPROGS we add the executed file name with '_' -> _uniqKernal\
	- In extras we update with the uniqKernal.c\ file
