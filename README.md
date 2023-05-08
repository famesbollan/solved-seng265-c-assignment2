Download Link: https://assignmentchef.com/product/solved-seng265-c-assignment2
<br>
Use the C programming language to write your implementation of the

Lempel-Ziv-Welch compression/decompression algorithm, in a file named “LZW.c”

<ul>

 <li>Test your code against the provided test data, and other test data you create, to ensure it’s working as expected. The instructor will also be testing your program against test data that isn’t provided to you. Think creatively about how to “break” and strengthen your code!</li>

</ul>

<h1>This assignment: “LZW.c”</h1>

In this assignment, we will implement Lempel–Ziv–Welch (LZW), a universal, lossless data compression algorithm. LZW is the algorithm of the Unix file compression utility, compress, and is used in the GIF image format. “Lossless” means that the compression operation preserves 100% of the information in the original format: with a correctly implemented algorithm, you can always recover the identical original data given its compressed form, even though the compressed form ​<em>may</em>​ be significantly smaller.

While the (glory) days when you could develop a company based on a compression algorithm are long-gone, there is still lasting practical value in knowing how to manipulate files of all types while implementing algorithms of increasing complexity, not to mention continuing to gain proficiency with array-based implementations — we can achieve all of the above with a deeper dive into compression via LZW.

Broadly speaking, compression is a process of recognizing recurring patterns in some piece of information to be communicated, and replacing multiple duplicate patterns with shorter references to entries in a list of such patterns. For example, imagine you have a list of numbers on a piece of paper, and you need to read it to a friend over the phone. As you’re going through the list, if you see the same number repeated enough times, you’re likely to say something like “48, 3011 ten times, 189, …” and rely on a shared understanding of natural language grammar to distinguish between the in-band data (the numbers) and the out-of-band data (the instructions about patterns). Unfortunately, digital data doesn’t have a “universal grammar” —

general-purpose data compression algorithms that are expected to operate on any type of data must be able to learn the repeated sequences in particular files automatically. As in life, “There ain’t no such thing as a free lunch”. For files that contain random data, or patterns that are too diverse or too sparsely distributed, a general-purpose compression algorithm might even produce “compressed” data that’s larger than the original!

LZW is a method for adaptive / dynamic compression – it starts with an initial model, reads data piece by piece and then updates the model while encoding the data as it proceeds through the input file. LZW can be used for binary and text files – it works on any type of file — that said, it is known for performing well on files or data streams with (any type of) repeated substrings (e.g. text files, which often contain frequently used words like “an”, “the”, “there”, etc.)​.

With English text, the algorithm can be expected to offer 50% compression levels or greater. Binary data files, depending on structuring are capable of decent compression levels as well –  for this assignment, file size is not part of the grading criteria–in fact, the evaluation relies on your code producing exact replicas of the expected compressed files so don’t get ​<em>too</em>​ creative!

As in the previous assignment, we are interested in “round-trip” encoding and decoding, translating from the original format to encoded and back to original. I hope that some of the lessons from the previous assignment will become useful sources for your reuse. While the compression technique may be new, other details remain similar. Implementing a more complex algorithm shows how we might layer on complexity once we establish base skills.

Dictionary Data Structure

A <strong>dictionary​</strong>​     is a general-purpose data structure for storing a group of objects. A dictionary has a set of keys and each key has a single associated value – the elements in this abstract data structure are commonly called,<strong> key-value pairs​</strong>​        . Dictionaries are usually implemented using hash tables, and are often known as “dict”, “hash”, “map”, or sometimes “hashmap” in different programming languages.

<h2>Binary Coding Schemes</h2>

The “<strong>LZW​</strong>​          ” algorithm described by Welch’s 1984 article [1] encodes variable-length sequences of 8-bit bytes as fixed-length 12-bit codes.

As an example of a binary code, this toy coding scheme uses three-bit codes to represent a choice of eight possible symbols (the letters ‘A’ through ‘H’). The choice of symbols is arbitrary–we could have used eight different emoji. As long as the sender and receiver have a shared understanding of what the codes mean, they can communicate using compressed data.

<table width="173">

 <tbody>

  <tr>

   <td width="99"><strong>binary code </strong></td>

   <td width="74"><strong>symbol </strong></td>

  </tr>

  <tr>

   <td width="99">000</td>

   <td width="74">A</td>

  </tr>

  <tr>

   <td width="99">001</td>

   <td width="74">B</td>

  </tr>

  <tr>

   <td width="99">010</td>

   <td width="74">C</td>

  </tr>

  <tr>

   <td width="99">011</td>

   <td width="74">D</td>

  </tr>

  <tr>

   <td width="99">100</td>

   <td width="74">E</td>

  </tr>

  <tr>

   <td width="99">101</td>

   <td width="74">F</td>

  </tr>

  <tr>

   <td width="99">110</td>

   <td width="74">G</td>

  </tr>

  <tr>

   <td width="99">111</td>

   <td width="74">H</td>

  </tr>

 </tbody>

</table>




In a binary code, each additional bit added to the code scheme doubles the number of distinct symbols that can be encoded–for an n-bit code, 2​<sup>n</sup>​ different symbols can be represented. In the classic LZW algorithm, the choice of a 12-bit coding scheme allows for 4096 distinct symbols with binary codes between 000000000000 and 111111111111. Each binary code can also be represented as three hexadecimal digits, e.g. 000 to FFF.




The dictionary data structure to be used for this assignment has a capacity of 4096 entries (2​<sup>12</sup>​), of which:

<ul>

 <li>the first 256 entries, numbered 0 through 255, are reserved for every possible 1-byte value (i.e. entry #0 is for the ASCII NUL character, entry #32 is for the ASCII space character, entry #65 is for the capital letter “A”… look at <strong>man ascii​</strong>​ ! (press ‘q’ to exit the man page reader))</li>

 <li>the entries numbered 256 through 4094 are assigned dynamically by the compression algorithm to represent multi-byte sequences in the input data.</li>

 <li>the last entry, numbered 4095, is reserved for a padding marker</li>

</ul>




The code that refers to an entry in the dictionary is simply the entry’s numeric position, encoded as a 12-bit binary integer.

Additional Notes

Both encoding and decoding programs refer to the dictionary countless times during the algorithm. You may want to reference the hints in the Provided Materials section of this document.

The encoder looks up indexes in the dictionary by using strings. That is, to aid the look-up of indexes, we use a dictionary of strings – i.e. an array of strings. With each string having some maximum number of characters, you might consider using as your data structure, a multidimensional array — as discussed in Unit 3, Slide 29, this has a form similar to: e.g., X[row][column], recall that we access elements of the array through integer indexes.

Why select such an approach? A data structure with 0(1), “Order-1 time complexity”, can be useful in practice.

Algorithm and Example

<table width="684">

 <tbody>

  <tr>

   <td width="684"><strong><em>LZW Encoding</em></strong> 1.    Start with a ​<em>standard initial dictionary</em>​ of 4096 entries (each entry can be identified using a 12-bit number)2.    Set ​<em>w</em>​ to ​”3.    As long as there are more characters to reada.    read a character ​<em>k </em>b.    if ​<em>wk</em>​ is in the dictionary    set ​<em>w</em>​ to ​<em>wk </em>elseoutput the code for ​<em>w </em>add ​<em>wk</em>​ to the dictionary, if it’s not full    set ​<em>w</em>​ to ​<em>k </em>4.    output the code for ​<em>w </em>Note #1: Step 2 is:    Set w to ​” It is not:Set w to ​’ ‘That is, you start off with w set to nothing, not a space character. So if you append ‘t’ to ”, you end up with ‘t’ (not ‘ t’).Note #2: The first 256 entries (0-255) in the LZW dictionary should be the 1-byte codes whose values are 0-255. That is,    dict[42][1] is 42 (and dict[42][0] is 1)    dict[228][1] is 228 (and dict[228][0] is 1)In general, for 0 &lt;= i &lt;= 255: dict[i][1] is i (and dict[i][0] is 1)</td>

  </tr>

 </tbody>

</table>




<table width="684">

 <tbody>

  <tr>

   <td width="684"><strong><em>LZW Decoding</em></strong> 1.    Start with a ​<em>standard initial dictionary</em>​ of 4096 entries (the first 256 are standard ASCII, the rest are empty).2.    Read a code <em>k</em>​ ​ from the encoded file3.    Output dict[​<em>k</em>​]4.    Set ​<em>w</em>​ to dict[​<em>k</em>​]5.    As long as there are more codes to read in the input filea.    read a code <em>k</em>​b.    if ​<em>k</em>​ is in the dictionary    output dict[​<em>k</em>​]add ​<em>w</em>​ + first character of dict[​<em>k</em>​] to the dict elseadd <em>w</em>​ ​ + first char of <em>w</em>​ ​ to the dict <em>and</em>​ ​ output itc.     Set ​<em>w</em>​ to dict[​<em>k</em>​]</td>

  </tr>

 </tbody>

</table>




The programs LZW.c takes as its first two inputs, the name of an input file as the first command line argument, and the name of an output file as the second command line argument.

If the second passed command line argument is an ‘e’, then the program creates a new file with the same name as the file in the command line, but with the extension .LZW. The new file contains the LZW encoding of the original file.

If the second passed command line argument is a ‘d’, then the program takes the name of compressed file as a command line argument (a file ending in extension, .LZW). The program interprets the original file name by dropping the .LZW file extension and then decodes the LZW encoding back to the “original.” (actually a copy).

The program should open both the input file and output file as binary files and make no assumption about what byte values appear in the files. Here’s what their behaviour looks like:




<table width="643">

 <tbody>

  <tr>

   <td width="643">~ /assignment2 &gt; cat tests/encode_short_string/input.txt this_is_his_history~ /assignment2 &gt; ./tools/b2x ./tests/encode_short_string/input.txt746869735F69735F6869735F686973746F72790A~ /assignment2 &gt; ./LZW ./tests/encode_short_string/input.txt e~ /assignment2 &gt; ./tools/b2x ./tests/encode_short_string/input.txt.LZW07406806907305F10205F10110310707406F07207900AFFF# different hex code means the input file and the encoded file are different~ /assignment2 &gt; diff ./tests/encode_short_string/compare.txt.LZW./tests/encode_short_string/input.txt.LZW# no output means the files are the same# rename our file so we don’t overwrite it (harder to determine if program executed correctly) mv ./tests/encode_short_string/input.txt ./tests/encode_short_string/input.old# now let’s decode our compressed file, and make sure it matches the original input ~ /assignment2 &gt; ./LZW ./tests/encode_short_string/input.txt.LZW d~ /assignment2 &gt; cat ./tests/encode_short_string/input.txt this_is_his_history~/assignment2 &gt; diff ./tests/encode_short_string/input.txt ./tests/encode_short_string/input.old # no output means they’re the same!~ /assignment2 &gt;</td>

  </tr>

 </tbody>

</table>




This session shows the contents of the file ​input.txt​. The file contains the string ​this_is_his_history​.

The hexadecimal values of the ASCII codes are also shown. Running the program ​LZW​ (with the encode flag) on ​input.txt produces a file input.ext.LZW​. The file ​input.txt.LZW​ contains 12-bit values for the LZW codes:

074 = t

068 = h

069 = i

073 = s

05F = _

102 = is

05F = _

101 = hi 103 = s_

etc.). The decode version of the program, LZW &lt;filename&gt; d, then correctly recovers the original file name from the given input, dropping .LZW from the input file name, and recovers the original data.

<strong> </strong>Requirements

The requirements for this assignment are as follows:

<ol>

 <li>Use arrays (and not dynamic memory allocation!) as an aggregate data type for file processing (encoding and decoding).</li>

 <li>Create a ​<strong>main </strong>​function that:</li>

 <li>Takes two command line arguments:

  <ol>

   <li>A string indicating the name of the file to operate on (the “input” file);</li>

   <li>A one-character flag indicating whether the program should operate in encoding or decoding mode. Use ‘e’ for encoding mode, or ‘d’ for decoding mode.</li>

  </ol></li>

</ol>

If the second argument is not provided, or is not either ‘e’ or ‘d’, iii.  print “Invalid Usage, expected: LZW {input_file} [e | d]” to the console

<ol start="4">

 <li>terminate the program with exit code 4.</li>

 <li>Opens the input file indicated by the first argument.</li>

</ol>

If there is no filename specified:

<ol>

 <li>print “Error: No input file specified!” to the console</li>

 <li>terminate the program with exit code 1.</li>

</ol>

If the input filename specified doesn’t exist or can’t be read for any reason: iii.      print “Read error: file not found or cannot be read” to the console iv.      terminate the program with exit code 2.

<ol>

 <li>Opens an output file using the naming convention as specified</li>

 <li>Depending on the value of the second command-line argument, passes the string read from the input file to either the ​<strong>encode </strong>or ​ ​<strong>decode </strong>​function, as described below</li>

 <li>Create a second function called ​<strong>encode </strong>with the following characteristics:​

  <ol>

   <li>Takes two C files as function parameters:

    <ol>

     <li>an input file</li>

     <li>an output file</li>

    </ol></li>

  </ol></li>

 <li>Encodes the provided file using the algorithm described above.</li>

 <li>If the file was encoded successfully:

  <ol>

   <li>create the encoded representation to the file system ii. terminate the program with exit code 0, indicating success</li>

  </ol></li>

</ol>

<ol>

 <li>If the file could not be encoded for any reason (e.g. a particular input fails):

  <ol>

   <li>print “Error: File could not be encoded” to the console ii. terminate the program with exit code</li>

  </ol></li>

</ol>

<ol>

 <li>Create a third function called ​<strong>decode </strong>with the following characteristics:​

  <ol>

   <li>Take two C files as function parameters:

    <ol>

     <li>an input file</li>

     <li>an output file</li>

    </ol></li>

  </ol></li>

 <li>Decode the provided file using the algorithm described above</li>

</ol>

If your program encounters invalid, corrupted or badly formatted data that cannot be decoded according to the specified algorithm, the program should:

<ol start="3">

 <li>print “Error: Invalid format” to the console ii. terminate the program with exit code 3.</li>

 <li>If the file was decoded successfully:

  <ol>

   <li>write the resulting decoded representation to a file with the expected name</li>

   <li>terminate the program with exit code 0, indicating success</li>

  </ol></li>

</ol>

<ol>

 <li>If the file could not be decoded for any reason (other than those stated above):

  <ol>

   <li>print “Error: File could not be decoded” to the console</li>

   <li>terminate the program with exit code 5</li>

  </ol></li>

</ol>

By now, you’re developing your own tests to evaluate your software (we don’t get what we ​<em>expect</em>​, we get what we ​<strong><em>inspect</em></strong>​) — consequently, we would like to see at least one test input from YOU – use the testing convention described in this assignment – i.e. if your test input is expected to successfully execute, then it should follow the happy path of the algorithm. If the test input is expected to fail, then it should show a brief error message followed by exiting with error code​<strong><em> 6</em></strong>​.

You may also think of tweaks to the algorithms that will make the encoder/decoder run faster, or make the compression better. If so, implement these ideas, and compare. But, this is not necessary. You are encouraged to devote time to testing your program.

Provided Materials

We have created a git repository for you – it contains a starter file, LZW-starter.c, a tool, b2x, a limited test suite, and these assignment instructions. There are also some hints in this section

<h3>LZW-starter.c</h3>

A “starter kit” for LZW.c for you to reuse in your implementation. Copy it to LZW.c and edit it, or copy and paste parts of it. The starter-file contains a few functions you are invited to use in your solution – these are read12(), write12() and flush12(). It is not your aim to understand the implementation of these functions (although you’re welcome to explore as you see fit) – it IS your aim to understand how you might apply. these functions in your solution – if you choose to apply them, it should have the effect of simplifying the solution. The use of flush12() might have an analogous application to the use of fflush(stdin) – while we’re on the subject: fflush() might have some use in your solution as well.

<h3> b2x Tool</h3>




We created a directory /tools and placed a source file, b2x.c, that reads a binary file and writes the “binhex” representation of a file to the screen — binhex is short for binary-to-hexadecimal – it’s yet another encoding mechanism – you can compile and use this tool if useful (it is not required). You can also use the common unix hexdump utility for similar purposes.

Test Suite

We will supply a limited suite of test cases that you can use to evaluate your own program, and get a feel for the types of inputs you can expect. You will see the tests include both text and binary files. The test suite has the same structure as the previous assignment – a set of directories, with one test case provided per directory. The directory, as previously, is named according to the type of test – e.g. encode_text or decode_image, where the first word denotes the mode the program should be run in while running this test.

Things are not identical to the previous assignment – for example, in the case of encoding:

<ul>

 <li>the filename passed may have ​<em>any</em>​ name and extension, e.g.<strong>ext​</strong>​ (your program is expected to read that file as its input, with ext replaced, e.g. with “txt” or “jpg”) NOTE: Your program ​<strong>does not</strong>​ need to locate the appropriate file to operate on — you can look at the files in the test case directory, and pass the input filename as a command line parameter!</li>

 <li>Continuing the example above, if your file is expected to run successfully given its input, the directory will contain a file whose name is<strong><em>ext.​</em></strong>​ <strong>LZW​</strong> where <strong><em>ext​</em></strong>​ is the original uncompressed filename extension</li>

</ul>

○    NOTE: Why “compare.ext”? — Well, we give you the input file for testing purposes; if properly run, you should get input.ext.LZW as your output – if we provided you with this properly named file as output, then you could perhaps overwrite it accidentally, which would remove your ability to compare – hence, we prefix the outputs with “compare”.

<ul>

 <li>Your program’s output is expected to ​<em>precisely</em>​ (bit-for-bit!) match the contents of this output file.</li>

 <li>If your program is expected to terminate with a non-zero (unsuccessful) error code when given the provided input file, the directory will contain one of the following:

  <ul>

   <li>a file named <strong><em>​</em></strong>​ <strong>err.txt​</strong>, where ​<strong><em>n​</em></strong> is the expected exit code, containing text that your program’s output is expected to precisely match, ​<strong>or; </strong></li>

   <li>a file named <strong><em>​</em></strong>​ <strong>err​</strong>, where <strong><em>n​</em></strong>​ is the expected exit code. In this case your output doesn’t need to match any specific error message, but your program should still exit with error code <strong><em>n</em></strong>​</li>

   <li>NOTE: for YOUR test(s) that you provide, should the test(s) exit with an error code, the error code must be <strong><em>6</em></strong>​ <strong><em>​</em></strong>.</li>

   <li>In the case of decoding, we provide input.ext.LZW and also provide compare.ext – again, you can derive the resulting output filename from the given input, and then compare accordingly.</li>

  </ul></li>

</ul>