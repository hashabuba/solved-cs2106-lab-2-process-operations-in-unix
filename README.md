Download Link: https://assignmentchef.com/product/solved-cs2106-lab-2-process-operations-in-unix
<br>
The purpose of this lab is to learn about process operations in Unix-based OS. There are a total of <strong>three exercises </strong>in this lab. The first exercise, which is the demo exercise, requires you to try out two system calls covered in lecture 2b. The remaining two exercises will see you building a simple shell interpreter. Since exercises 2 and 3 are quite involved, we will describe them in a separate section (section 2).




<strong>1.1 Exercise 1 </strong><strong><u>[Lab Demo Exercise]</u></strong>




In this exercise, you will use a combination of fork() and waitpid() to perform simple process creation and synchronization. The waitpid()library call is a variant in the wait() family which enables us to wait on a specific pid. Don’t forget to check the manual page (“man –s2 waitpid”). The requirement is best understood through a sample session.




<table width="516">

 <tbody>

  <tr>

   <td width="142"><strong>Sample Input: </strong></td>

   <td width="374"> </td>

  </tr>

  <tr>

   <td width="142"><strong>3</strong></td>

   <td width="374">//3 child processes</td>

  </tr>

 </tbody>

</table>




<table width="516">

 <tbody>

  <tr>

   <td width="516"><strong>Sample Output: </strong></td>

  </tr>

  <tr>

   <td width="516"><strong>Child 1[818]: Hello!   </strong>//818 is the process id of child<strong> Parent: Child 1 [818] done. </strong><strong>Child 3[820]: Hello! </strong><strong>Child 2[819]: Hello!    </strong>//Order between child processes is not fixed<strong> Parent: Child 2[819] done. </strong><strong>Parent: Child 3[820] done. </strong><strong>Parent: Exiting.</strong></td>

  </tr>

 </tbody>

</table>

The program will read in an integer <strong>N</strong> between 1 to 9 (inclusive) which indicates the number of child processes to spawn. Each of the child processes print out a “Child X[PID]: Hello!” message, where X is the index number and PID is the process id of the child,. The parent process will then check for the termination for each of the child and print out the corresponding message, “Parent: Child X[PID] is done.”. The parent process should spawn all child processes before checking on them. This check is performed in the spawning order, i.e. Child <strong>1, 2, … N</strong>.




Note that the order in which the child process print out message is not fixed, so your output may not exactly match the sample session. However, certain order must always be respected, e.g. Child X must print the “Hello” message before the parent can print out “Child X is done” message. Due to the nature of the output, there is <strong>no sample test cases provided. [</strong>Testing hint: use the “<strong>sleep()</strong>” command to introduce random small delays for the child processes. Pay attention to the order of messages.<strong>]</strong>




<h1>Section 2. Command Line Interpreter (Exercise 2 and 3)</h1>




As shown during the lecture, a command line interpreter (aka command prompt or shell) involves most of the major process operations. So, you are going to implement an interpreter with various features in the remaining exercises. In exercise 2, only a few simple functionalities are required. You should take the opportunity to plan and design your code since the same code can be reused in exercise 3.




Note that when you unpacked the directories for these two exercises, you’ll find several C files and a file named “<strong>makefile</strong>” in each exercise directories in additional to the usual skeleton files <strong>ex2.c</strong> and <strong>ex3.c</strong>. These extra C files are tiny programs with various runtime behavior to aid your testing later. To build these programs, simply enter “<strong>make</strong>” command under the directories. The “<strong>make</strong>” utility invokes the <strong>gcc</strong> compiler automatically to prepare the executables for you.




Summary of the extra programs:

<table width="520">

 <tbody>

  <tr>

   <td width="138"><strong>clock</strong></td>

   <td width="382">Prints out a message after <strong>X</strong> seconds and repeat for <strong>Y</strong> times. <strong>X</strong> and <strong>Y</strong> can be specified as command line arguments. See given code for more details</td>

  </tr>

  <tr>

   <td width="138"><strong>alarmClock </strong></td>

   <td width="382">Prints out a message after <strong>X</strong> seconds and terminates. <strong>X</strong> can be specified as command line argument. See given code for more details.</td>

  </tr>

  <tr>

   <td width="138"><strong>infinite </strong></td>

   <td width="382">Goes into infinite loop, never terminates.</td>

  </tr>

  <tr>

   <td width="138"><strong>return </strong></td>

   <td width="382">Return the user specified number <strong>X</strong> to the shell interpreter.</td>

  </tr>

  <tr>

   <td width="138"><strong>showCmdArg </strong></td>

   <td width="382">Shows the command line arguments passed in by user.</td>

  </tr>

  <tr>

   <td width="138"><strong>stringTokenizer </strong></td>

   <td width="382">Shows how to split user input into subparts (token).</td>

  </tr>

 </tbody>

</table>

Due to the nature of the interpreter, it is hard to test using sample input/output. So, instead of sample test cases, a sample usage session is included in the later sections of this document.




<strong>2.1 Exercise 2 (Basic Interpreter) </strong>




Let us implement a simple working interpreter with limited features in this exercise.




<table width="383">

 <tbody>

  <tr>

   <td width="383"><strong>General flow of basic interpreter </strong></td>

  </tr>

  <tr>

   <td width="383">1.     Prompt for user request.2.     Carry out the user request.3.     Unless user terminates the program, go to step 1.</td>

  </tr>

 </tbody>

</table>







<table width="507">

 <tbody>

  <tr>

   <td width="507"><strong>TWO possible user requests: </strong></td>

  </tr>

  <tr>

   <td width="507">a. Quit the interpreter. Format: <strong>quit</strong> <strong>Behavior: </strong><sup>•</sup> Print message “Goodbye!” and terminates the interpreter.</td>

  </tr>

  <tr>

   <td width="507">b. Run a command. Format: <strong>command_path </strong>//Read the path of the command//The <strong>command_path</strong> is assumed to be less than 20 characters <strong>Behavior: </strong>a.   If command exist, run the command in a child process<sup>•</sup> Wait until the child process is doneb.   Else print error message “<strong>XXXX not found</strong>”, where <strong>XXXX</strong> is the user command.</td>

  </tr>

 </tbody>

</table>




For (b), you need to check whether the <strong><em>command_path</em></strong> exists (i.e. valid). This can be achieved by various methods, one simple way is to make use of a library call:




<strong>stat() </strong>




Find out more about this function by “<strong>man –s2 stat</strong>”. This library function has various usages, and some are quite involved. However, you don’t need to have the full knowledge to use this call effectively. [Hint: look carefully at the return type of this function.]




Make use of the <strong>fork() </strong>and <strong>execl()</strong> combo to run valid commands. The <strong>execl()</strong> function call discussed in the lecture is sufficient for this exercise. When using <strong>execl()</strong>, you can assume that the path of the program is the same as command line argument zero (i.e. name of the program).




Just like a real shell interpreter, your program will wait until the command finishes before asking for another user request.




<table width="507">

 <tbody>

  <tr>

   <td width="507"><strong>Assumptions: </strong></td>

  </tr>

  <tr>

   <td width="507"><strong>a.     </strong><strong>Non-terminating command will NOT be tested on your interpreter </strong><strong>b.    </strong><strong>No </strong><strong>ctrl-z or </strong><strong>ctrl-c will be used during testing. As we have not covered signal handling in Unix, it is hard for you to take care of these at the moment. </strong><strong>c.     </strong><strong>You can assume user requests are “syntactically correct”. </strong><strong>You can assume the </strong><strong>command_path has less than 20 characters. <em> </em></strong></td>

  </tr>

 </tbody>

</table>







Suggestions on approach:




<ul>

 <li>Modularize your code! Try to find the correct code for each of the functionality independently. Test the function thoroughly then work on another part of the program. This allows your code to be reused in exercise 3.</li>

</ul>




<strong>Sample Session: </strong>




User Input is shown as <strong>bold</strong>. The prompt is “YWIMC”, short for “Your whim is my command” J, just a way to show your shell interpreter is a devoted servant. Note that additional empty lines are added in between of user inputs to improve readability.




YWIMC &gt; <strong>/bin/ls               </strong>//See note*

a.out     ex2.c     …               //output from the “ls” command




<h2>YWIMC &gt; <strong>./alarmClock</strong><strong>                      </strong>//Use the provided program</h2>

Time’s up. 3 seconds elapsed.           //after ~3 second delay




YWIMC &gt; <strong>./anything</strong>

./anything not found                      // “./anything” does not exist

<strong> </strong>

YWIMC &gt; <strong>quit </strong>

<h2>Goodbye!                      // Interpreter exits</h2>

<strong><sup> </sup></strong><strong>Note*:</strong> The “<strong>ls</strong>” command may be located at a different location on your system. Use <strong> </strong>“<strong>whereis ls</strong>” to find out the correct path for your system. <strong> </strong>

<strong>2.2 Exercise 3 (Advanced Interpreter) </strong>

<strong> </strong>

This exercise extends the capabilities of the interpreter from exercise 2. Please note the enhanced or new requests below:




<table width="507">

 <tbody>

  <tr>

   <td width="7"> </td>

   <td width="494"><strong>SIX possible user requests: </strong></td>

   <td width="7"> </td>

  </tr>

  <tr>

   <td colspan="3" width="507">a. [No change] Quit the interpreter. Format: <strong>quit</strong>                  //No change from ex2.</td>

  </tr>

  <tr>

   <td colspan="3" width="507">b. <strong>[Enhanced]</strong> Run a command. Format:<strong>command_path <u>[arg1 to arg4]</u> </strong><u>//the user can supply <strong>up to 4 command line arguments</strong></u><strong> </strong><strong>Behavior: </strong>a.    If the specified command exists, run the command in a child process with the supplied command line arguments.<sup>•    </sup>Wait until the child process is done.<sup>•    </sup>Capture the child process’ return value (see “<strong>result</strong>” command).b.   Else print error message “<strong>XXXX not found</strong>”, where <strong>XXXX</strong> is the user entered command.</td>

  </tr>

  <tr>

   <td colspan="3" width="507">c. <strong>[New]</strong> Run a command in the background. Format: <strong>command_path [arg1 to arg4] <u>&amp;</u> </strong>//Note the “<strong>&amp;</strong>” symbol at the end of the command. The user can supply <strong>up to 4 command line arguments, same as (b). </strong><strong> </strong><strong>Behavior: </strong>a.    If the specified command exists, run the command in a child process with the supplied command line arguments.<sup>•    </sup>Print “<strong>Child XXXX in background</strong>”, <strong>XXXX</strong> should be the PID of the child process.<sup>•    </sup><u>Continue to accept user request.</u>b.   Else print error message “<strong>XXXX not found</strong>”, where <strong>XXXX</strong> is the user entered command.</td>

  </tr>

  <tr>

   <td colspan="3" width="507">d. <strong>[New]</strong> Wait for a background process. Format:<strong>wait job_pid </strong>//”wait” followed by an integer <strong>job_pid</strong> <strong>Behavior: </strong>a.      If the <strong>job_pid</strong> is a valid child pid (generated by the request in (c)) and <u>has not been waited before:</u><sup>•    </sup>Wait on this child process until it is done, i.e. the interpreter will stop accepting request.<sup>•    </sup>Capture the child process’ return value (see “<strong>result</strong>” command).b.     Else print error message “<strong>XXXX not a valid child pid</strong>”, where</td>

  </tr>

  <tr>

   <td colspan="3" width="507"><strong>XXXX</strong> is <strong>job_pid</strong> entered by user.</td>

  </tr>

  <tr>

   <td colspan="3" width="507">e. <strong>[New]</strong> Print the pids of all unwaited background child process. Format: <strong>printchild  </strong>Output format:<strong>Unwaited Child Processes:      </strong>//Just an output header<strong>&lt;Pid of Unwaited Child 1&gt;      </strong>//May be empty if there is no<strong>&lt;Pid of Unwaited Child 2&gt;      </strong>// unwaited child<strong>  </strong><strong>…………………………… </strong><strong>Behavior:                   </strong>a. PID of all “unwaited” background child processes (including terminated ones) are printed. <strong> </strong></td>

  </tr>

  <tr>

   <td colspan="3" width="507">f. <strong>[New]</strong> Print the return result (i.e. exit status) of a child process. Format: <strong>result</strong>Output format:<strong>&lt;return result&gt;      </strong>//A single integer number<strong>Behavior: </strong>a. This command is <strong>only valid after a successful (b) or (d)</strong>.</td>

  </tr>

 </tbody>

</table>




<table width="507">

 <tbody>

  <tr>

   <td width="507"><strong>Assumptions: </strong></td>

  </tr>

  <tr>

   <td width="507">a.     Non-terminating command will NOT be tested on your interpreter.b.     No ctrl-z or ctrl-c will be used during testing. As we have not covered signal handling in Unix, it is hard for you to take care of these at the moment.c.     <strong>Each</strong> command line argument for (b) and (c) has less than 20 characters.d.     There are <strong>at most 10 background jobs</strong> in a single test session.e.     <strong>You can assume all user requests are “syntactically correct”.</strong></td>

  </tr>

 </tbody>

</table>




<strong>Notes: </strong><sup>o</sup> You need to learn a few C library calls by exploring the manual pages. Most of them are variants of what we discussed in lecture.

<ul>

 <li>Instead of <strong>execl()</strong>, it is easier to make use of <strong>execv()</strong> in this case. Read up on <strong>execv()</strong> by “<strong>man –s2 execv</strong>”.</li>

 <li>Remember to use the <strong>waitpid()</strong> call in exercise 1.</li>

 <li>You only need a <strong>simple way </strong>to keep track of the background job PIDs. (hint: a simple array will do….).</li>

</ul>




<strong>             </strong>

<strong>Suggestions on approach: </strong>

<ul>

 <li>This exercise is fairly involved. Again, implement the required functionalities incrementally is the best approach.</li>

 <li>You will need to manipulate string to some extent. Try to write a program just to test out your code first. [Warning: String manipulation is pretty frustrating in C. Don’t be demoralized. J] [Hint: take a look at the sample programs…]</li>

</ul>




<strong>Sample Session: </strong>

<strong> </strong>

User Input is shown as <strong>bold</strong>.




<table width="514">

 <tbody>

  <tr>

   <td width="514">YWIMC &gt; <strong>/bin/ls        </strong>//Path of <strong>ls </strong>may be different on your system a.out      ex3.c <strong>……</strong>       //output from the “ls” command YWIMC &gt; <strong>/bin/ls –l</strong><strong>  </strong>//same as executing “ls –l” total 144-rwx——   1 sooyj    compsc      8548 Aug 13 12:06 a.out………………..          //other files not shown YWIMC &gt; <strong>result          </strong>//successful “ls” returns 0 to shell0 YWIMC &gt; <strong>./alarmClock 20 &amp; </strong>//Background job. <strong>See Note*</strong> Child 12345 in background //PID 12345 is just an example<strong> </strong>YWIMC &gt; <strong>./showCmdArg one 2 3 four</strong>Arg 0: ./showCmdArgArg 1: oneArg 2: 2Arg 3: 3Arg 4: four<strong> </strong>YWIMC &gt; <strong>wait 12340</strong><strong>            </strong>//Try to wait for  PID1234012340 not a valid child<strong> </strong>YWIMC &gt; <strong>printchild </strong>Unwaited Child Processes:12345<strong> </strong>YWIMC &gt; <strong>wait 12345  </strong>// Will wait until the previous alarmClock// process terminates. Note: It is possible that                                                 // alarmClock has already terminated by this time.// In that case, this request will return immediately. YWIMC &gt; <strong>printchild </strong>Unwaited Child Processes:     // empty in this case YWIMC &gt; <strong>./anything 1 2 3</strong>./anything not found                     // “./anything” does not exist</td>

  </tr>

 </tbody>

</table>

<strong>Note*:</strong> The background job printing will intermix with your interpreter input / output. There is no need (and no way  J) to make background job print “nicely”.




<table width="514">

 <tbody>

  <tr>

   <td width="514"> YWIMC &gt; <strong>./return 188 &amp;        </strong>// simply return “188” to shell<strong>   </strong>Child 12366 in background   //PID 12366 is just an example <strong>   </strong>YWIMC &gt; <strong>wait 12366            </strong> YWIMC &gt; <strong>result          </strong>//show the return result of the waited process188 YWIMC&gt; <strong>quit </strong>Goodbye!                      // Interpreter exits</td>

  </tr>

 </tbody>

</table>