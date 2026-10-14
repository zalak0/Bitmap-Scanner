Overview: Printed Circuit Board Quality Control
Many consumer electronic devices contain one or many Printed Circuit Boards (PCBs) which are used to hold and connect together the various electronic components in a circuit. PCBs are typically manufactured on large, automated production lines, which require quality control processes to detect any faults or defects that occur during the manufacturing process. Quality control on manufactured circuits is performed using a variety of means: one technique is to capture a digital photograph (or micrograph) image of the circuit during production, and to use image processing methods in software to assess the physical layout of the board, to determine if the board has been manufactured correctly.


In this assignment, you will play the role of a software developer working at a large PCB manufacturing company tasked with developing a basic suite of image processing software for automated PCB quality control. Your software will be provided with images of circuit boards as they come off the production line, and using a database of known components, you will map the layout of components you detect in the images and the various electrical connections between them using the visible PCB trace lines.

Overview of the Task
The program you develop for this assignment will read in two files: (a) a BMP format file that contains an image of the target/manufactured PCB and (b) a custom binary data file which contains image template data for a library of possible components that may appear on the PCB. Your program will implement three key capabilities:

Output/display template data: Your program will print to the screen information for a selected template component

List/Map components: Your program will identify the location in the image of electronic components using the template library and report these to the user.

Map electrical connections: Your program will build a list of which components are electrically connected to one another on the PCB by analysing the traces in the image.

Background: Images and Colour
Colour images of manufactured PCBs will be captured on the production line and passed into your software.

Standard colour images contain a two-dimensional array of pixel data, where the colour of each pixel is represented by the component brightness of Red (R), Green (G) and Blue (B) light (similar to the tri-chromatic receptiveness to light by the human eye). In many image formats, each of the Red-Green-Blue values is stored as a number between 0-255, and the combinations of the strength of these different RGB values gives the gamut of colours visible by the human eye, for example:

(RGB) = (255,0,0) indicates pure red

(RGB) = (0,255,0) indicates pure green

(RGB) = (0,0,255) indicates pure blue

(RGB) = (255,255,0) indicates yellow (i.e. a mix of green and red)

(RGB) = (255,0,255) indicate purple/magenta (i.e. a mix of red and blue)

(RGB) = (255,128,0) indicate orange (mostly red, with a bit of green)

(RGB) = (0,0,0) is black and (RGB) = (255,255,255) is white

The combination of (RGB) gives 256 x 256 x 256 = 16,777,216 different possible colours. 

Background: Image Coordinates
Image data is fundamentally a two-dimensional array of pixels. Image coordinates (x, y) represent the index in both the horizontal and vertical directions of the image of a corresponding pixel, and are measured from the bottom left-hand corner of the image:


For example the coordinate (0,0) is the bottom left pixel of an image, and one pixel up from this in the vertical direction is the coordinate (0,1). Pixel data (in this assignment) is stored such that the rows (y) are in the first dimension of the data and the columns (x) are in the second dimension. A third index of the array represents the colour channel (i.e. Red: 0, Green: 1 and Blue: 2).

Background: the BMP File Format
In this assignment, you will work with images stored in the BMP (Bitmap) format. You will be provided with scaffold code that can be used to read in a Bitmap image file and also save image data to a Bitmap file (see "Bitmap.h", "Bitmap.c" in the scaffold code, and further details of the provided functions in the below sub-section "Pre-provided code in the scaffold").

Bitmap files (*.bmp) have a relatively simple file format. A bitmap file consists of a 14-byte general header followed immediately by a DIB (Device-Independant Bitmap) header containing metadata about the pixel format and image size. The pixel data follows immediately after the DIB header. The size of the pixel data can be calculated from the information in the header. Several example images are provided in the scaffold for the assignment. 

Running the hex dump utility 
 xxd <file_name.bmp> 
in the terminal window is a useful way of displaying the encoded BMP information. For more information see https://en.wikipedia.org/wiki/Hex_dump

Bitmap File Header
A bitmap file always begins with a 14-byte header that describes the contents of the file. The bitmap file header always starts with the 2 bytes 0x42 0x4d (i.e. BM). The next 4 bytes give the size of the file in bytes. The next 4 bytes are reserved. The next 4 bytes indicate the offset from the start of the file to where the pixel data start.

DIB Header
The bitmap file header is always immediately followed by the DIB header. The first 4 bytes of the DIB header indicate the size of the DIB header. The next 4 bytes indicate the number of pixels in the horizontal direction. The next 4 bytes indicate the number of pixels in the vertical direction. The remainder of the DIB header contains additional information about the image, such as colour data and compression, that is not relevant to this assignment. 

If you are interested in learning more about the BMP file format see https://en.wikipedia.org/wiki/BMP_file_format

Image Pixel Data
Each pixel in the image is represented by 3 bytes that give the values of the red, green and blue (R, G, B) content of the pixel. Each of the 3 bytes can have a value between 0 and 255 that indicates the amount of R, G or B in a pixel.

Image template-based matching to find components
"Template Matching" is an image processing method used for detecting and localising an object in an image when one possesses an example "template" for how the object should appear. To perform template matching, images are often pre-processed by thresholding, where each pixel of the image data is assigned one of two possible values: either a 1/true or 0/false, usually based on whether or not the pixel brightness value is above a specified threshold value. The template image data (which is usually of a smaller region than the target image) is then used to compare to every possible location in the target image, by scanning across the image rows and columns, looking for a match. A match occurs at a specific location when the thresholded values of each and every corresponding pair of pixels from the target image and template image are equal, by overlaying the template image at the specific location in the image.


The process for template matching can be summarised in the following steps:

Load the target image and threshold the target image, so that every pixel is labelled as either 1 or 0. For thresholding, if the average (R,G,B) value of the pixel is greater than 127, then the pixel is labelled 1, and 0 otherwise. The image data is represented as a two dimensional array with dimensions HxW, where W is the image width and H the image height.

Load a template image (pre-thresholded) for a particular object to search for. The template data is represented as a two dimensional array with dimensions MxM, where M is the width/height of the template image.

Loop through (using two nested loops) each possible (row,column) position in the image to test for a match. Coordinates will vary from 0 to (W-M) horizontally and 0 to (H-M) vertically.

At each candidate location (row,column) loop through the MxM pixels at this image location (row to (row+M) and column to (column+M) ) and compare each against it's correspond pixel in the template. If all pixels have matching true/false values, then a positive match is found. If any one of the pixel pairs is not equal, then this location is not a match.

The reported location of the match is the image coordinate (row,column) that corresponds to where the bottom left pixel (first pixel) of the template is overlaid.

This proceedure is then repeated with multiple different templates corresponding to different objects to detect.

*Note on processing real/more complex images outside of this assignment: For the purposes of this assignment, you can follow the above proceedure. As an aside, in practice and other applications, sometimes image data doesn't match precisely pixel-for-pixel: the above proceedure is typically modified to calculate the correlation coefficient (or other performance metrics) as a means of detecting if a target has been found (you can assume in this assignment that the images are captured with sufficient precision such that a true object will match precisely with it's corresponding template).

Image Template Library/Database File
Image templates used in the proceedure above for each of the electronic components you will need to detect are provided to you in a custom binary file. The file encodes the template data (pre-thresholded into 0/1 per pixel) for a number of N different components. The file is organised as shown below:


The first byte of the binary file corresponds to N (between 0-255) the number of templates in the file. Template data is then stored immediately after this in each subsequent 128 bytes. Each bit of the data represents the 1/true or 0/false pixel value on the 32x32 template images, starting from the bottom left pixel of the template and moving row-by-row through the image data (one byte contains 8 pixels, 4 bytes per image row x 32 rows = 128 bytes).

Identifying connections between components
Once components have been identified on a target PCB, the PCB image data can also be analysed to determine the connections between the various components by analysing the traces visible on a thresholded image of the PCB. Shown below is an example of thresholded PCB image and an indication of which components are connected to one another: two components are considered connected if and only if there exists a visible trace line that runs directly between each of the outer edges of the 32x32 pixel regions for each component. 


It is possible to identify these connections by searching for the existence of a "path" that runs from one outer edge to another in the thresholded pixel data. The thresholded image pixels can be treated as a graph in which 1/true pixels are graph nodes, and edges exist between each 1/true pixel and any of it's direct neighbours (up/down/left/right, but NOT diagonals) which also have a 1/true value. If a path can be found between one component's outer edge to another (for example be performing a depth-first or breadth first search), then these components are considered "connected".

Note that the space occupied in the 32x32 pixel regions itself of each component is not considered a valid region of this graph: so looking at the example above, component 1 and 4 are not considered connected to one another as they cannot be considered directly connected via their common connection with component 2.

Program Input Format
The program will be run from the command line in the following format:

./pcb_check <mode t|l|c> <template_filename> <index or board_image_filename>

The first argument passed to the program is the program mode (can be 't', 'l' or 'c'). The second argument is the file path to the image template library file. The third argument is either an index integer of one of the components in the library file (for mode 't') or the file path to the PCB image file (for modes 'l' and 'c').

The program mode can be one of three possible values:

Mode 't': template mode: in this mode, the program reads the template file and prints to the screen a representation of the component template corresponding to the index provided as an integer in the third argument.

Mode 'l': component list mode: in this mode, the program uses the template library file (argument 2) and the PCB image file (argument 3) to identify any components on the PCB (using template matching) and list their positions and types according to the library.

Mode 'c': check connections mode: in this mode, the program detects and lists components (as for mode 'l') and also lists the electrical connections between each of the components identified.

If the user provides the incorrect number of arguments to the program, then you rpogram should return the message:

Invalid arguements!

and exit. If an invalid mode is selected, your program should return the message:

Invalid mode selected!

and exit. If your program cannot find the file specified for the template library (second arguement) or there is an error opening the file, then your program should return the message:

Can't load template file

and exit. If a valid template files is opened, but the specified template index (arguement 3 for mode 't') is invalid or out of range, then your program should return the message:

Template index out of range

before exiting.

Mode 't' (template mode)
When run using mode 't' your program should display a text-based representation of the template data for the specified index in the template file. The program should print Template data: on the first line, and each subsequent line should contain each row of the template image data represented using characters, starting from the top left pixel/row. A '1' should be printed for each 1/true pixel and a space ' ' for each 0/false pixel in the template (also see "Examples" below).

Mode 'l' (list components mode)
When called in mode 'l', your program should produce a list of the identified components and their image coordinates using the template matching procedure described above. The first line of output should indicate the number of components found, and each subsequent line provide details of the component types (index in the template library file) and image position (row,col) of the bottom-left pixel of the object. The order in which components are listed should be first by their row (components towards bottom of the image first), and for components on the same row, they should be ordered by column (components towards the left of the image first).

For example the following output:

Found 3 components:
type: 4, row: 61, column: 99
type: 0, row: 206, column: 78
type: 2, row: 227, column: 189

would indicate that three components were found: types 4, 0 and 2. The first component found (type 4) was located at (row,column) = (61,99) in the image. The second component found (type 0) was located at (row,column) = (206,78) in the image. The third component found (type 2) was located at (row,column) = (227,189) in the image.

Mode 'c' (check connections mode)
When called in mode 'c', your program should begin by returning the list of components as shown above for mode 'l'. Your program should then return a list of connections that exist between each component. Each component's connections should be specified on a separate line, in order of component number (from 0 to L, for L components found in the image) where the component number corresponds to the order in which they were found (see mode 'l' above).

For example, for the circuit displayed in section "Identifying connections between components" above, the output provided would be:

Component 0 connected to 1 2
Component 1 connected to 0 2
Component 2 connected to 0 1 4
Component 3 connected to nothing
Component 4 connected to 2

Suggested approach to the assignment
You should consider breaking up this assignment into several progressive functionalities and capabilities. A suggested road map to follow in your development process is:

Start by developing functions to read in and decode binary template library file and handle 't' mode use cases.

Next develop functionality to perform template matching on an image that is loaded in from a BMP file and associated use case of the 'l' mode.

Finally develop functions checking connections between identified components by treating the thresholded image as graph and searching for a valid path between components.

The test cases for the assignment will move through scenarios that progressively test these different functionalities.

Examples
A number of examples template library files and PCB image files are provided in the scaffold/code challenge for the assignment.

Input:

./pcb_check t templates/temp001.bin 2

Output:

Template data:
11111111111111111111111111111111
1                              1
1                              1
1                              1
1  11111111111111111111111111  1
1  1                        1  1
1  1                        1  1
1  1  11111  111  1    11   1  1
1  1    1   1     1     1   1  1
1  1    1   1     1 1   1   1  1
1  1    1   1     11111 1   1  1
1  1    1   1       1   1   1  1
1  1  11111  111    1   1   1  1
1  1                        1  1
1  1                        1  1
1  1                        1  1
1  1  111111111             1  1
1  1  1       1             1  1
1  1  1       1             1  1
1  1  1       1             1  1
1  1  1       1             1  1
1  1  1       1             1  1
1  1  1       1             1  1
1  1  1       1             1  1
1  1  111111111             1  1
1  1                        1  1
1  1                        1  1
1  11111111111111111111111111  1
1                              1
1                              1
1                              1
11111111111111111111111111111111

Expand

Input:

./pcb_check l templates/temp001.bin images/pcb001.bmp

Image of PCB001:


Output:

Found 3 components:
type: 4, row: 61, column: 99
type: 0, row: 206, column: 78
type: 2, row: 227, column: 189

Input:

./pcb_check c templates/temp001.bin images/pcb002.bmp

Image of PCB002:


Output:

Found 9 components:
type: 1, row: 39, column: 286
type: 1, row: 68, column: 63
type: 0, row: 86, column: 137
type: 1, row: 113, column: 63
type: 4, row: 135, column: 323
type: 3, row: 172, column: 228
type: 7, row: 200, column: 404
type: 7, row: 222, column: 350
type: 5, row: 238, column: 63
Component 0 connected to 4 
Component 1 connected to 2 3 
Component 2 connected to 1 3 4 6 8 
Component 3 connected to 1 2 
Component 4 connected to 0 2 6 7 8 
Component 5 connected to nothing
Component 6 connected to 2 4 7 8 
Component 7 connected to 4 6 
Component 8 connected to 2 4 6 

Marking
Your final mark for this assignment is made up of the following component marks:

Code Challenge Automated Test Cases (70%)

Code quality and Written explanation of all of your code (30%)

The mark for point 2 is composed of:

Code description (1-page limit) worth 10% - this is mandatory, and you will receive zero marks for the assignment without this.

Code quality worth 20%: This includes use of best practice in variable and function names, formatting, style, use of modular design principles and appropriate/effective use of C constructs for the task at hand (see additional notes below).

Automated test case marking is done by pressing 'Mark' in the assessment editor. You will see immediate feedback on each test case (excluding a number of hidden test cases to prevent hard-coding). Make sure you attempt to compile and test the program first using the terminal!

Code description is a short (maximum 1-page) description of your code and of the design process followed to develop your code.

code_description.txt/code_description.pdf: create and edit this text file in Ed like you would write code, or upload a pdf containing your document.

Once you have uploaded or written your description in the code challenge workspace, make sure to click the "mark" button one more time so that it will be visible to the tutors when marking.

Design principles used when creating/developing the code.

Accuracy of description to actual code.

Succinct and concise description. Extra words won't mean more marks and may detract!

Code quality is an assessment by the marker of:

Appropriate and consistent formatting and style, self-documenting code, including appropriate commenting

Appropriate use modular design principles, including functions and modular source design.

Appropriate use of C language constructs which are suitable for the task.

Use of Libraries
You may use any functions in the C Standard Library for the code challenges. You may not use any external sources of code, regardless of licensing. The following headers are likely to be most useful:

Header <stdio.h>

Header<string.h>

Header <stdlib.h>

Look at cppreference for details about what functions each header contains.

Academic Honesty
This assignment is to represent individual work. You are not to collaborate with your peers - this is an individual assignment. You may discuss your approach to the problem with other students, but you may not share code, look at another student's code, or allow another student to look at your code. Similar code will be treated with suspicion. All submissions will be checked for similarity.

All code you submit for this assignment must be your own code, and written by yourself. It is not acceptable to use code written by others, including the use of any code produced using generative AI tools.

Although part of the marking of the assignment is based on the number of automated test cases you pass, we will be manually checking solutions for situations in which the solutions has been "hard-coded". By hard-coding, we mean that the code is specifically designed to directly solve the provided test cases only, and would definitely not work in anyway with more general cases if provided. An example of hard-coding would be a program that uses a switch-case (or other decision-making construct) to directly identify a specific input test case, and just print back the known desired response, without trying to address the problem at hand.

Evidence of hard-coding in your code solutions will result in zero marks for this assignment.

Late Submission
Late submissions will be penalised at a rate of 5 marks per day or part thereof. Submissions that are more than 10 calendar days late will not be accepted. The last submission is the one that will be assessed, regardless of whether an earlier one has a greater value in test cases solved.
