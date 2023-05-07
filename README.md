Download Link: https://assignmentchef.com/product/solved-kernel-project-4-proc-file-systems-and-mmap
<br>
The purpose of this project is to practice writing Linux kernel modules to create a new device file and an entry in the proc file system. The objectives of this project is to learn:

<ol>

 <li>How to write a helloworld Linux kernel module.</li>

 <li>How to create a new device in Linux.</li>

 <li>How to add a new entry in the proc file system.</li>

</ol>

<h1>Project</h1>

<h2>Part 1: Create a Helloworld kernel module</h2>

The following code is a complete helloworld module.

#include <em>&lt;</em>linux/module .h<em>&gt;</em>

#include <em>&lt;</em>linux/kernel .h<em>&gt;</em>

int init module ( void ) { printk (KERN INFO ”Hello world !
”); return 0;

}

void cleanup module ( void ){ printk (KERN INFO ”Goodbye world !
”);

}

moduleinit ( init module ); moduleexit ( cleanup module );

MODULE LICENSE(”GPL”);

The module defines two functions. init module is invoked when the module is loaded into the kernel and cleanup module is called when the module is removed from the kernel. module init and module exit are special kernel macros to indicate the role of these two functions. Use the following Makefile to compile the module (you need <strong>root </strong>permission).

obj−m += new module . o all :

sudo make −C / lib /modules/$( shell uname −r )/ build M=$(PWD) modules clean :

sudo make −C / lib /modules/$( shell uname −r )/ build M=$(PWD)                   clean

Do you see any errors? If so, how to fix the error? To insert the module into the Linux kernel:

# sudo insmod new module.ko

Use the following command to verify the module has been loaded:

# lsmod

To remove the module from the kernel:

# sudo rmmod new module

Use dmesg to see the module’s output.

<h2>Part 2: Create an entry in the /proc file system for user level read and write</h2>

Write a kernel module that creates an entry in the /proc file system. Use the following code skeleton to write the module:

#include <em>&lt;</em>linux/module .h<em>&gt;</em>

#include <em>&lt;</em>linux/kernel .h<em>&gt;</em>

#include <em>&lt;</em>linux/ proc fs .h<em>&gt; </em>#include <em>&lt;</em>linux/ string .h<em>&gt;</em>

#include <em>&lt;</em>linux/vmalloc .h<em>&gt;</em>

#include <em>&lt;</em>asm/uaccess .h<em>&gt;</em>

#define MAX LEN 4096 int read info ( char ∗page , char ∗∗ start , off t off , int count , int ∗eof , void ∗data );

ssize t write info ( struct f i l e ∗ filp , const char user ∗buff , unsigned long len , void ∗data );

<table width="0">

 <tbody>

  <tr>

   <td width="30">int</td>

   <td width="349">init module ( void ) { int ret = 0;// allocated memory space              for           the proc entry info = ( char ∗) vmalloc (MAX LEN);memset( info ,           0 , MAX LEN);//implement this : create the proc entry write index = 0;</td>

  </tr>

 </tbody>

</table>

static struct proc dir entry ∗proc entry ; static char ∗ info ; static int write index ; static int read index ;

readindex = 0;

<table width="0">

 <tbody>

  <tr>

   <td width="312">// register the write and read callback proc entry−<em>&gt;</em>read proc = read info ;</td>

   <td width="68">functions</td>

  </tr>

 </tbody>

</table>

<table width="0">

 <tbody>

  <tr>

   <td width="446">             return       ret ;}void cleanup module ( void ){//remove the proc entry and free                 info      space}</td>

   <td width="53"></td>

  </tr>

  <tr>

   <td width="446">ssize t write info ( struct f i l e ∗ filp , const char user unsigned long len , void ∗ data ){</td>

   <td width="53">∗buff ,</td>

  </tr>

  <tr>

   <td width="446">//copy the written data from user space and save it return len ;}</td>

   <td width="53">in       info</td>

  </tr>

 </tbody>

</table>

procentry−<em>&gt;</em>write proc = write info ; printk (KERN INFO ” test proc created .
”);

int read info ( char ∗buff , char ∗∗ start , off t offset , int count , int ∗eof , void ∗data ){

//output the content of info to user ’ s buffer pointed by buff return len ;

}

The callback functions write info and read info will be invoked whenever the proc file is written and read, respectively, e.g., using the cat and echo commands. write info uses the copy from user function to communicate with the user space. To test your results, load the kernel module and there should be a new entry created under /proc. Use cat and echo to verify and change the content of the new entry.

Note: you need to change the previous Makefile’s obj-m += new module.o line to compile the module.

<h2>Part 3: Exchange data between the user and kernel space via mmap</h2>

Write a kernel module that creates an entry in the /proc file system. The new entry can not be directly read from or written to using cat and echo. Instead, map the new entry to a user space memory area so that user level processes can read from and write to the kernel space via mmap. The skeleton of the kernel module is given below:

#include<em>&lt;</em>linux/module .h<em>&gt;</em>

#include<em>&lt;</em>linux/ l i s t .h<em>&gt;</em>

#include<em>&lt;</em>linux/ init .h<em>&gt;</em>

#include<em>&lt;</em>linux/kernel .h<em>&gt;</em>

#include<em>&lt;</em>linux/types .h<em>&gt;</em>

#include<em>&lt;</em>linux/kthread .h<em>&gt;</em>

#include<em>&lt;</em>linux/ proc fs .h<em>&gt; </em>#include<em>&lt;</em>linux/sched .h<em>&gt; </em>#include<em>&lt;</em>linux/mm.h<em>&gt;</em>

#include<em>&lt;</em>linux/ fs .h<em>&gt;</em>

<table width="0">

 <tbody>

  <tr>

   <td width="55">static</td>

   <td width="507">struct                     proc dir entry ∗tempdir , ∗tempinfo ;</td>

  </tr>

  <tr>

   <td width="55">static</td>

   <td width="507">unsigned char ∗ buffer ;</td>

  </tr>

  <tr>

   <td width="55">static</td>

   <td width="507">unsigned char array                                                    [12]={0 , 1 , 2 , 3 , 4 , 5 , 6 , 7 , 8 , 9 , 10 , 11};</td>

  </tr>

  <tr>

   <td width="55">static</td>

   <td width="507">void allocate memory ( void );</td>

  </tr>

 </tbody>

</table>

#include<em>&lt;</em>linux/slab .h<em>&gt; </em>#include <em>&lt;</em>asm/io .h<em>&gt;</em>

static void clear memory ( void ); int my map( struct f i l e ∗ filp , struct vm area struct ∗vma);

static           const       struct           file operations              myproc fops = {

.mmap = my map,

};

static int my map( struct f i l e ∗ flip , struct vm area struct ∗vma){ // map vma of user space to a continuous physical space return 0;

}

static           int         init myproc module ( void ){

// create a directory              in /proc

// create a new entry under the new directory printk (” init myproc module successfully 
”);

allocate memory (); // i n i t i a l i z e the buffer for ( i =0; i <em>&lt;</em>12; i++){ buffer [ i ] = array [ i ] ;

}

return      0;

}

static                void allocate memory ( void ){

// allocation memory

// set        the memory as         reserved

}

static              void clear memory ( void ){

// clear         reserved memory

// free memory

}

static void exit myproc module ( void ){ clear memory (); remove proc entry (”myinfo” , tempdir ); remove proc entry (”mydir” , NULL);

printk (”remove myproc module successfully 
”);

}

moduleinit ( initmyprocmodule ); moduleexit ( exit myproc module );

MODULE LICENSE(”GPL”);

Write a user space program to test the proc file you just created. Use the following skeleton:

#include <em>&lt;</em>unistd .h<em>&gt; </em>#include <em>&lt;</em>stdio .h<em>&gt;</em>

#include <em>&lt;</em>stdlib .h<em>&gt;</em>

#include <em>&lt;</em>string .h<em>&gt;</em>

#include <em>&lt;</em>fcntl .h<em>&gt;</em>

#include <em>&lt;</em>linux/fb .h<em>&gt;</em>

#include <em>&lt;</em>sys/mman.h<em>&gt;</em>

#include <em>&lt;</em>sys/ ioctl .h<em>&gt;</em>

#define PAGE SIZE 4096

int main( int argc , char ∗argv [ ] ) { unsigned char ∗p map ;

// open proc        f i l e

// map p map to the proc             f i l e            and grant read &amp; write                privilege

// design        test                  case to read from and write to p map

// unmap p map from the proc f i l e return 0 ;

}