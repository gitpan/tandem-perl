***********************************************************************
*                         Perl for GUARDIAN                           *
***********************************************************************



FORWARD
-------


Welcome to Perl (Version 5.004) for Guardian.  This is version 2 of the
Guardian Port.

This file contains information you will need to:

     * Run Perl on your Guardian NSK system
     * Compile Perl on your Guardian NSK system (should that be necessary)
     * Install Perl on your Guardian NSK system (jump to end of document)

Take a look at the file whatsnew.txt for a description of the
changes since the last upload.

This port has only been running on a D35 version of Guardian and has been
tested on G06 (using the D44 C compiler & the G06 NMC compiler).  If the
executable file doesn't run on your system, try recompiling (using the
supplied instructions).  If it still doesn't run, give me a call and maybe
I can help you figure out what to do next.

This document contains the following sections:

     * INTRODUCTION

       Some remarks on why I tried to do this port (_why_ indeed?)

     * FILE ORGANIZATION

       How the Perl subdirectories got mapped into Guardian file names

     * LIBRARIES

       How libraries work under Guardian

     * HOW TO USE PERL ON GUARDIAN

       An explanation of the command line

     * STUFF THAT APPEARS TO WORK

       A discussion of what works

     * STUFF THAT WON'T WORK

       A discussion of what doesn't (or can't ever) work.

     * PERFORMANCE

       A brief remark on performance.

     * EXTENDING PERL FOR GUARDIAN WITH C ROUTINES

       A pointer to another text file to read

     * TEST SUITE RESULTS

       A discussion of what happened when I ran the validation suite

     * COMPILATION INSTRUCTIONS

       Some tools for recompiling and re-binding Perl for Guardian

     * TACL SUPPORT

       An explanation of the TACL macros

     * LICENSING

       This describes the LICENSE (it points you at the LICENSE file).

     * INSTALLATION INSTRUCTIONS

       How to install this beast.

     * README DOCUMENT CHANGE HISTORY

       A list of changes made to this document and the software over time.
       If you make any changes to the software please update this document
       to reflect them.


First, a caveat:  I'm not a UNIX person and only a moderately experienced C
programmer (most of my work has been in TAL).  So the choices I made about
what to get working and the approach I took for getting it working may seem
odd or idiosyncratic or even just plain wrong.  So, I am very open to
suggestions about how to improve this port.  Please feel free to call or
write with any ideas or thoughts you might have.

Enjoy!

Carl Adler
IDX, Systems Corp.
Seattle, Washington
(206) 689-1479
carl_adler@idx.com

(206) 542-6291
carla@universalmediation.com




INTRODUCTION
------------

This first bit of verbiage is a discussion of why we are trying
to do this port and some of the peculiarities associated with
implementing Perl on Guardian.

Perl was built to help people do interesting work on UNIX
platforms, primarily related to system managment.  It has been
successfully ported to a number of other platforms which resemble
UNIX in several important aspects:

     *  The file system is tree structured

     *  Process creation is not a capital crime

     *  Most of the UNIX utilities or something like them have
        already been ported.

Gurdian does not resemble UNIX in any of these aspects.  So what
did we hope to gain by trying to do this port?

     *  TACL doesn't do regular expressions; Perl does.

     *  Many people know Perl; not many know TACL.

     *  There are books you can buy to help you learn Perl.  There
        are no books on TACL that I'm aware of.

     *  For people who know C, Perl is probably easier to learn
        than TACL.

     *  With a little luck, any Perl scripts generated for our use
        on Guardian, will be useful on other platforms.  It's a
        cinch that our TACL macros won't be useful.

     *  There are a ton of Perl scripts available in the world that might
        be usable to people on a Tandem (obviously, those that are used
        to help manage UNIX systems won't be).



FILE ORGANIZATION
-----------------

On tree structured file systems, Perl is usually in something
like:

      /usr/perl/...

and there is a tree under this with "bin", "lib", etc.

Here's what we did:

     PERL is currently installed on volume $SYS2 and resides in
     a set of subvols, each of whose names is prefixed with "PERL":


     PERL       PERLO executeable, TACL library, PERLLOCL (config &
                init) PERL bootstrap macro and this file.

     PERLMNT    Perl Maintanence macros

     PERLSRC    Source code (program and header files)

     PERLOBJ    Object code (one per source compile)

     PERLL01  - PERLL09 Library directories (see the AAREADME file in
                each subvol for it's original UNIX name).

     PERLT01  - PERLT07 /t directories (see the README file in each
                subvol for it's orignal UNIX name).

     PERLLIB  - Libraries added by implementer and site

     PERLEXT  - This is where we've stashed code to implement our
                extensions (at the time of this writing there are two).

     PERLEXTT - PERL scripts we used to test our extensions



LIBRARIES
---------

Starting with the 10Sep1998 version of this port, the way we
handle libraries has changed.

You no longer need to specifically have a define for each library
file.  You only need defines for sub-volumes.  In particular:

    =PERL_LIBRARY,        CLASS SEARCH, SUBVOL0(...)
    =PERL_LIBRARY_FILE,   CLASS SEARCH, SUBVOL1(...)
    =PERL_LIBRARY_TEXT,   CLASS SEARCH, SUBVOL2(...)
    =PERL_LIBRARY_MATH,   CLASS SEARCH, SUBVOL3(...)
    =PERL_LIBRARY_GETOPT, CLASS SEARCH, SUBVOL4(...)
    =PERL_LIBRARY_SYS,    CLASS SEARCH, SUBVOL5(...)
    =PERL_LIBRARY_SEARCH, CLASS SEARCH, SUBVOL6(...)
    =PERL_LIBRARY_TIME,   CLASS SEARCH, SUBVOL7(...)
    =PERL_LIBRARY_TIE,    CLASS SEARCH, SUBVOL8(...)
    =PERL_LIBRARY_LOCAL,  CLASS SEARCH, SUBVOL9(...)
    =PERL_LIBRARY_CSTM,   CLASS SEARCH, SUBVOL10(...)
    =PERL_LIBRARY_EXT,    CLASS SEARCH, SUBVOL11(...)

where "..." is a list of subvols that hold the library files.


** Library Sub-volumes.

In the distribution, there are 9 sub-volumes:  PERLL01 thru PERLL09
which map to various library sub-directories in the original CPAN
distribution.  The PERLL01 sub-volume maps to the top level "LIB"
dictory.  The rest map to lower level directies.

We have added a sub-volume called PERLLIB for IDX or Guardian
specific libraries.

=PERL_LIBRARY_LOCAL always maps to PERLLIB.



** Search DEFINES.

Each one of these defines maps to one of these sub-volumes (via
the contents of the "...".  You can put home-grown libraries that
you want to make available to everyone on the system in
PERLLIB (and map that subvol to "..._LOCAL".  Individual user
libraries can be mapped to any subvolume desired via the
"..._CSTM" define (these work more or less like TACLLOCL and
TACLCSTM).



** How it Works.

In your PERL script you could say:

    use strict;

and internally, PERL for Guardian will use the =PERL_LIBRARY
search define to find the file:  $vol.PERLL01.STRICT.

For portability reasons, we have to be able to support, for example:

    use File::Find;

In the UNIX world: "File::Find" maps to "./File/Find.pm"
which is a relative path from the current directory (note that
through the @INC mechanism, UNIX and DOS versions of perl
accomplish what we're doing here via SEARCH defines.  When
PERL for Guardian sees this usage, it will strip the "file"
and append it to the main LIBRARY define, creating the define:

    =PERL_LIBRARY_FILE

PERL for Guardian then uses the =PERL_LIBRARY_FILE search define
to locate $vol.PERLL02.FIND.  ONLY two levels are supported
in PERL for Guardian.  That is:

    use foo::bar::zot;

is equivalent to:

    use bar::zot;

Internally, PERL converts:

    "strict" and "File::Find"

to:

    "strict%" and "File/Find%"

which acts as a trigger for invoking these mechansims.

In the stock PERL for Guardian distribution the define
=PERL_LIBRARY lists every library subvol as search subvols.  Thus
You never have to qualifify a library name unless it is duplicated
in which case you will want to qualify it.  For example, suppose
you wanted to create an alternate version of "find" for use at
your site.  You would put in in the PERLLIB sub-volume and in
your script you would say:

    use local::find

and PERL for Guardian would use the library $vol.PERLLIB.FIND
instead of $vol.PERLL01.FIND.

NOTE:  The Perl error message: "Can't locate <use file> in @INC"
has been changed to:  "Unable to resolve <use file> to a file".
When you get this error it typically means the DEFINE that
was selected by PERL (based on <use file>) does not reference
as sub-volume that contains the file you are trying to USE.

ANOTHER NOTE:  The Perl error message (or something like it):

    Undefined subroutine &main::find called at foo line 6

occurs when you have a "use" statement that references a library
file that did exist but references package names that don't.  Notice
in the previous example where we did (first letter of each is in caps):

    use File::Find;

This is because the package name associated with File Finder Library
is:

    package File::Find;

While Guardian file names are not case sensitive, package names are.
That is:

    use file::find;

will compile and execute correctly, but the subroutine call

    find (...);

will not.


** When do the DEFINES get created?

When you execute the PERL macro in PERLMAC, the first thing that
happens is that a macro called PERL.PERLLOCL is executed.  This
macro contains all of the defines for the standard PERL library
directories (or at least many of them).  PERLMAC maintains a
timestamp which tells it not run the PERLLOCL macro again unless
it has been modified.

The PERL macro will also execute a macro called PERLCSTM if it
finds it in your default sub-volume.  This PERLCSTM is a macro
which should contain, among other things:

    ADD DEFINE =PERL_LIBRARY_<suffix>, CLASS SEARCH SUBVOL10 <mylib>

where:

    <mylib> is a subvolume name where you want to put your own
    personal library files.

To access your own library files you would say:

    use <suffix>::<file>

where:

    <suffix> is some string you invent for the higher level
    directory

and:

    <file> is the name of a file in the subvolume <mylib>




HOW TO USE PERL ON GUARDIAN
---------------------------


1.  Use normal PERL run line syntax.  For example:

       PERL -w PSCRIPT ARG1 ARG2


Assuming <vol>.PERL has been added to your PMSEARCHLIST, "PERL"
in this case is really a macro that will execute the PERL
interpreter.  The first time you execute PERL in any particular
TACL session, the PERL macro will initialize your TACL session
by creating a bunch of variables and defines that make PERL
work correctly on GUARDIAN.

Other macros are available for debugging PERL.  Look inside
<vol>.PERL.PERLMAC for details.



STUFF THAT APPEARS TO WORK
--------------------------

In general anything that operates on variables and does no I/O
or system calls appears to work.  Some I/O and system calls work:

1.  Basic "line oriented" file I/O AND ENSCRIBE I/O works.
    Line oriented files are EDIT files and UNSTRUCTURED files.
    You must include the "\n" character to terminate a line oriented
    record.  ENSCRIBE is different.  See 1.3 below for details.

    1.1  Perl can read and write UNSTRUCTURED files.  This means
         for example, that, a file of binary data that has been
         FTP'd to the Tandem can be processed.

         PAY ATTENTION TO THIS:

               UNSTRUCTURED files must have ODDUNSTR flag set
               in order for Perl to be able to write to them.
               Otherwise you will get the following error:

                     "Not a C file"

         You can use either <> or "read" to read UNSTRUCTURED files.
         However, suppose $D1.FOO.BAR is a an UNSTRUCTURED file
         containing binary (i.e. non-displayable) information.  The
         following script:

              $foo = "\$D1.FOO.BAR";
              open F,$foo;
              $d = <F>;

         will read all of the information from the file $D1.FOO.BAR
         into the variable $d until either EOF or until a "\n"
         character is found.  That is, for "line oriented" files the
         <FILE> will read one line and a line is alway terminated with
         the "\n" character.  NOTE: the "\n" character will be
         included in the information in $d.  If there is no "\n"
         character in a file, <FILE> will read the entire file into
         memory.  So far we have not seen a limit on how large the file
         can be in this case.

    1.2  Perl scripts can know what kind of file you are using.
         The following script demonstrates this:

            stat($ARGV[0]);                   # get "status" of file
            if (!defined -e _) {              # does not exist if undefinded
               print "$ARGV[0] doesn't exist\n";
               exit 0;
            }

            print "Directory\n" if -d _;      # if file was <template>
            print "Block Special\n" if -b _;  # if file was ENSCRIBE
            print "File\n" if -f _;           # if file was line oriented
            print "Text\n" if -T _;           # if content is displayable
            print "Binary\n" if -B _;         # if content isn't

         That is:

            -d returns true if the filename is actually a template
            -b returns true if the file is an ENSCRIBE file
            -f returns true if the file is a line oriented file
            -T returns true if the first 512 bytes or the first
               ENSCRIBE record contains only displayable data.
            -B returns true if the first 512 bytes or the first
               ENSCRIBE record contains non-displayable data.

         and, if the result of any of these tests is !defined then
         the file you are testing doesn't exist.

    1.3  Perl can read and write ENSCRIBE files but only through the use
         of the following commands:

            - sysopen
            - sysread
            - syswrite

         If you try to use "open" on an ENSCRIBE file you'll get an
         error (in $!):

               "Not a C file"

         If you are reading ENSCRIBE files you must specify a length
         that is >= the record length of the file or you will truncate
         your data.  For example:

               $file = "\$CUST15.HHDATA.ACCT";
               sysopen (ACCT, $file, 0);
               $status = sysread (ACCT, $rec, 4000);

         The "sysread" will return up to 4000 bytes from an ACCT record.

         Also, $rec will contain RAW data.  That is, if the record
         had binary data in it, then $rec will have it as well.
         You must write your scripts knowing the record layout.

         To write to ENSCRIBE files use "syswrite".  The length
         of the string you writing must make sense in terms of the
         record length of the file.  For example, if the record length
         of the ENSCRIBE record was 127, then the variable that you
         write should be 127 bytes long.

         When writing to ENSCRIBE files, DO NOT include "\n" character
         at the end of the string.  ENSCRIBE files are record oriented
         (1 syswrite = 1 record) and the "\n" will be included in the
         data which you probably don't want.

         When writing to KEY SEQUENCED files you must make sure that
         the primary key is unique.  For example, suppose you were
         writing to a KEY SEQUENCED file with KEYOFF = 0, KEYLEN = 10.
         The first 10 bytes of every variable you write will be the
         primary key.

         When writing to RELATIVE RECORD files, if you write out a zero
         length string, there will not be a record written.  That is
         if you have say:  $foo = "" and then do:

              syswrite (OUTFILE, $foo, length($foo));

         No record will be writen if OUTFILE is RELATIVE RECORD.

         EOF doesn't work on ENSCRIBE files (it always returnes TRUE).
         For sysread you can always use "defined" to test the result
         of the read (I've doctored up sysread so that even Perl
         files return !defined at end of file).  Read on for more.

         And, if the result of sysread or syswrite is !defined then
         an error has occured.  $! in a numeric context returns the
         error code.  $! in a string context returns an error message.
         The message is usually somethine like:  "Guardian or user
         define error <num>".  In particular, you should quit your
         read/write loops when the result of system is !defined.  For
         example:

              while (defined sysread (FILE, $var, $rdlen)) {
                  ...process $var...
              }

              if ($! != 1) {
                 print "Error on FILE: $!\n";
              }

         Writing is similar.

         Errors are the usual Guardian suspects:

              21   number of bytes is longer than record
              45   file full
              43   disk file
               1   end of file
              etc.

2.  OPENDIR/CLOSEDIR/READDIR works but it has some funny Guardian
    quirks.  Here are some examples that work (that is, READDIR
    will actually give you a file name)

         a.    opendir DIR, $CUST20.F5OBEY

               Each readdir will return a file in this directory.

         b.    opendir DIR, $CUST20.F5OBEY.A*

               The mask is expanded and readdir will return an
               item from this list.

         c.    opendir DIR, $CUST20.FAOBEY.A

               same as b.

     The rule is this:  OPENDIR will take whatever you pass it and
     try to turn it into a template which is then passed to the
     FILE_FINDFIRST_ Guardian procedure call.  If you pass something
     that doesn't have any wild cards (as in example "a", above),
     OPENDIR will append a ".*" to the end of it.  OPENDIR will
     take "b" verbatum because it has wild cards in it.  Example "c"
     may look like a file name but OPENDIR needs to pass a template
     so it converts "c" into "b".  if "A" were really a file name
     OPENDIR will still work, but READDIR will return all of the
     files in FAOBEY that begin with the letter "A".

     NOTE:  the PERL run command in the TACL library will expand
     templates before running the Perl executable unless the
     template is surrounded by quotes.  For exmample, suppose
     the subvolume $D1.FOO had three files in it:  BAR, DRAT, & ZOT.
     And, further suppose you entered the following command (REMEMBER,
     YOU MUST RUN PERL VIA THE PERLMAC MACRO):

               PERL -W SCRIPT FOO.*

     The command that actually gets executed is:

               PERL -W SCRIPT BAR DRAT ZOT

     In particular, note that SCRIPT will never see the reference
     to "FOO.*".

     You can get a template passed to a PERL script by surrounding it
     with double quotes.  For example, if you wanted FOO.* to be
     seen by SCRIPT, you would enter the command like:

               PERL -W SCRIPT "FOO.*"

     SCRIPT can access this template as $ARGV[0].


3.   SYSTEM works.  What you do with it is execute TACL commands.
     The C Runtime starts up a TACL the first time you use it and
     this TACL is yours and yours only.  This means you can do
     things like:

               system ('#push #out');
               system ('#set #out $s.#junk');
               system ('fileinfo $d1.junk.*');

     The fileinfo will generate a list of the files in $d1.junk to
     the spooler location mentioned.  Note the use of single quotes
     to prevent PERL from treating $s and $d1 as variables.

4.   CHDIR seems to work (see PERLL02.FIND).

5.   While(<>) seems to work as long as you don't try to pass it
     any templates (this is why PERL command line expands templates
     before running the PERL program).

6.   Command switches that seem to work:

         -w    will display warnings
         -h    will display help
         -V    will display configuration (not that it tells you much)
         -e    but be careful because the command line parsing isn't
               intutively obvious.  Here's an example, this doesn't work
               as you'd expect:

                   perl -e"$a=~1"; -e"print ""$a\n""";

               but this version of the same thing does:

                   perl -e"$a = ~1"; -e"print ""$a\n""";


7.   Requester/Server scripts.  Perl scripts can act as either servers
     or requesters.

     A Perl script acting as a server accepts messages on $RECEIVE,
     does something, and the replies.  A perl script acting as a
     requester, "prompts" a server for some work and accepts its reply
     all in a single read request (Guardian WRITEREAD).

     To "enable" reqeuster/server behavior, a script must open $RECEIVE
     or a process using "sysopen" with "MODE=3".

     The following is a simple server:

        sysopen (R, "\$RECEIVE", 3);
        while (defined sysread (R, $s, 200)){
             print "Request:  $s\n";
             $s = "Ok";
             syswrite (R, $s, length($s));
        }
        print "$!\n";

     It opens $RECEIVE with mode = 3.  Then until an error occurs
     (which isn't likely) the script reads $RECEIVE, prints what it
     received, and then responds with "Ok".  This script will run
     forever until you stop the PERL processor.

     IMPORTANT NOTE:  When you want Perl to execute a server script
     Perl must itself be running as a named process.  That is:

        TACL > perl/name $ptzzz/smplsvr

     This is so that requesters know where to send their messages.

     The following is a simple requester:

        sysopen (S, "\$PTZZZ", 3);
        $s = ">";
        sysread (S, $s, 200);
        print $s;

     This script opens a process named $PTZZZ (e.g. the Perl processor
     running the server script above).  It then sends a ">" as a prompt
     to the process, gets back it's response, prints the response and
     then terminates.  Note that when "mode = 3" "sysread" works like
     the COBOL verb:  READ WITH PROMPT with the contents of buffer
     containing the prompt and being replaced with the response.

     Perl scripts can also do "simplex" (i.e. one-way) output to
     processes and can do "simplex" (i.e. one-way) reads on $RECEIVE.
     In this case, you use SYSOPEN with mode = 0 (for read) or 1
     (for write).

     [ed note:  someday, when we're bored, we might try to implement
     PATHSENDS, but don't hold your breath].

8.   Perl can directly access the following Guardian system calls:

           Perl Call                    System Call
           ---------                    -----------
           GUARDEXT::aborttmf           ABORTTRANSACTION
           GUARDEXT::begintmf           BEGINTRANSACTION
           GUARDEXT::endtmf             ENDTRANSACTION
           GUARDEXT::control            CONTROL
           GUARDEXT::setmode            SETMODE
           GUARDEXT::position           POSITITION
           GUARDEXT::keypos             KEYPOSITIONX

      In every case the parameters and return values are as
      described in the Guardian Procedure Calls Manual.  The following
      script illustrates how to use the keypos routine to position
      within a KEYSEQUENCED file before reading.

        1       sysopen  (HTLOG, "\$CUST15.HHDATA.HTLOG", 0)
                         or die "Couldn't open HTLOG: $!\n";
        2       $fnum = fileno(HTLOG);
        3       $keyval = "9802230800";
        4       $keyspec = "\x00\x00";
        5       $len = length $keyval;
        6       $len = ($len << 8) | $len;
        7       $pmode = 0;
        8       $status = GUARDEXT::keypos($fnum, $keyval, $keyspec, $len,
                         $pmode);
        9       print "$status\n";
       10       sysread (HTLOG, $logrec, 300);
       11       print "$logrec\n";

      Line 1 opens an HTLOG file using sysopen.
      line 2 gets the file number for that file
      line 3 sets the date/time to start at
      line 4 sets the key to "primary key"
      line 5 & 6 sets up the length word (left half = righ half) as
        described in the Procedure calls manual.
      line 7 sets the positioning mode to approximate.
      line 8 does the keypos
      line 9 prints the status (1 = Okay, anything else is failure)
      line 10 reads the first record at or after the one that has the
        the date/time specified
      line 11 prints out the record.


9.   Libraries and packages seem to work.  But there is an anomoly
     you need to be careful about.  Suppose you were to say:

        use file::find;

     thinking that you would be able to then say, somewhere in your
     script:

        find (\&callback, ...);

     Because Guardian file names are case INSENSITIVE, PERL will be
     able to locate the "find" library in PERLL02 via the
     =PERL_LIBRARY_FILE define.  So far so good.

     However, since the package name is:

        File:Find

     (note that the first character is upshifted), the use
     command never actually found anything to import into the main::
     namespace.

     So the moral of this story is when you want to "use" packages,
     make sure you ask for them EXACTLY like they were declared
     keeping case sensitivity in mind even though in terms of
     finding the file, Guardian won't care about the case.

10.  The MD5 package has been added.  Here are the notes from the README
     file.  See PERLEXTT.MD5T2 for an example of how to use it.

     Digest::MD5
     ---------------------------

     This is a Perl extension interface to the RSA Data Security Inc. MD5
     and MD2 Message Digest algorithms.  This package also provide modules
     which calculate HMAC digests.

     <notes on installation, removed>

     Copyright 1998 Gisle Aas.
     Copyright 1995-1996 Neil Winton.
     Copyright 1990-1992 RSA Data Security, Inc.

     This library is free software; you can redistribute it and/or
     modify it under the same terms as Perl itself.

     The Digest::MD5 and Digest::MD2 modules are written by Gisle Aas
     <gisle@aas.no>.  They are based on MD5-1.7 made by Neil Winton.  The
     Digest::HMAC module is written by Graham Barr and Gisle Aas.

11.  `cmd` doesn't work, but we've implemented a module called BACKTICK
     which provides almost the same functionality:

        SYNTAX: backtick ("<TACL command>")

        Where:

                <TACL command>

        This argument is Required.  It can be any valid command that can
        be issued from the TACL prompt.

        Description:

        Backtick is a Perl function that allows a Perl program to issue
        TACL commands.  It sends a command, and returns the output from that
        command.  The output is returned as a reference to an array.

        Usage:

        The statement:

            use backtick;

        must be present in the Perl code using backtick, prior to any
        backtick commands.

        Output and Errors:

        If there is an error in the TACL command, let's say the command
        used was "foop info myfile", then backtick will return the error
        message produced by TACL.

        If there was no output produced from TACL, let's say the command
        used was "volume $d1.myvol", then the returned output will be defined,
        but blank.

        If there was an error in backtick itself, let's say it can not open
        TACL, then the returned output will be undefined.

        Limitations:

        Backtick will not work if Perl is being used as a server (see
        the stuff previously about sysopen mode 3 I/O).

        Efficiency concerns:

        The first time backtick is used in a Perl program, it needs to
        create and open a TACL session, which is significant overhead.
        However, only the first backtick command in a Perl program has
        this overhead, all other backtick commands do not need to start
        TACL.

        Example:

            # Perl program that uses backtick
            use backtick;
            # print out the results of a FUP INFO command
            print backtick ("fup info $data1.foo.*");

            # capture the results of a TACL #output command
            my $array_ref = backtick ("#output foobar");

            # and print out the captured results
            foreach (@$array_ref)  {print $_, "\n";}



STUFF THAT WON'T WORK
---------------------

This list isn't fully enumerated yet.  Please tell me if you try
something that you thought might work and doesn't.

In general, anything that appears to be a UNIX kind of command
like fork(), exec(), mkdir(), etc., won't work.

seek() Doesn't work.

PIPES don't work.

TCP/IP stuff doesn't work (although we could, in principle, make
them work).

File name globbing doesn't work (e.g. while(<FOO.*>)

File tests that ask questions about ownership, executability and
other OS dependent stuff don't work (see the discussion above
about file I/O for a list of the file tests that do work).

The @{<expression>} construct causes perl to crash.  I don't know why
yet.

You can redirect STDOUT and STDERR to a disk file inside a perl
perl program but once you do, you can't switch back.

Printing arrays directly does not give you the results that you'd expect.
For example if @a contains 3 elements: "EL1", "EL2", and "EL3".  Then
print @a generates "EL1EL2EL3" with maybe some garbage characters
interspersed.  In general when you print arrays, you should do it from
within a foreach loop.

Command line switches that don't work:

     -d  A bunch of errors suggesting that perl is doing very
         UNIXy kind of stuff

See below for details on the results from the suite of tests.


PERFORMANCE
-----------

Input seems acceptable.  I was able to load a 13,300 line file into a
variable in about 4 + or - 1 seconds using:

      undef $/;
      $d = <INFILE>;
      $/ = '\n';

I haven't done anything to optimize Output so it's a little slow.

Regular expression evaluation looks like it's about 25% to 50%
slower than on DOS.

If you notice anything else, let me know.


EXTENDING PERL FOR GUARDIAN WITH C ROUTINES
-------------------------------------------

Please read the file HOWTOEXT which is part of this release.



TEST SUITE RESULTS
------------------

The following summarizes the results of running the programs in
the /perl/t/... directories.

In general, they support my supposition that anything that perl does
internally works and only a few things that it does with the
outside world work.  It turns out, that Perl is crashing either
during compilation or execution of 17 of the many dozens of test.
It also turns out that many of the tests fail because of hard coded
UNIX syntax file names.  Sometimes (I only tried a few) by modifying
the names to GUARDIAN syntax I could get the scripts to run.  If
I had to modify hard coded file names and, after doing so, the script
ran, I called it successful.

The other thing the test suite does, a lot, is add a hard coded
library reference to @INC.  I had to eliminate those assignments
to get many of the scripts to run.  This suggests that any scripts
that depend on being able to look in multiple places for a file
will fail.

The following is the "directory" listing for all of the test scripts.
You'll notice some of the names have been munged to 8 characters:


perl5.004/t/README
perl5.004/t/TEST
perl5.004/t/base/cond.t
perl5.004/t/base/if.t
perl5.004/t/base/lex.t
perl5.004/t/base/pat.t
perl5.004/t/base/term.t
perl5.004/t/cmd/elsif.t
perl5.004/t/cmd/for.t
perl5.004/t/cmd/mod.t
perl5.004/t/cmd/subval.t
perl5.004/t/cmd/switch.t
perl5.004/t/cmd/while.t
perl5.004/t/comp/cmdopt.t
perl5.004/t/comp/colon.t
perl5.004/t/comp/cpp.aux
perl5.004/t/comp/cpp.t
perl5.004/t/comp/decl.t
perl5.004/t/comp/multiline.t
perl5.004/t/comp/package.t
perl5.004/t/comp/proto.t
perl5.004/t/comp/redef.t
perl5.004/t/comp/script.t
perl5.004/t/comp/term.t
perl5.004/t/comp/use.t
perl5.004/t/harness
perl5.004/t/io/argv.t
perl5.004/t/io/dup.t
perl5.004/t/io/fs.t
perl5.004/t/io/inplace.t
perl5.004/t/io/pipe.t
perl5.004/t/io/print.t
perl5.004/t/io/read.t
perl5.004/t/io/tell.t
perl5.004/t/lib/abbrev.t
perl5.004/t/lib/anydbm.t
perl5.004/t/lib/autoloader.t
perl5.004/t/lib/basename.t
perl5.004/t/lib/bigint.t
perl5.004/t/lib/bigintpm.t
perl5.004/t/lib/checktree.t
perl5.004/t/lib/complex.t
perl5.004/t/lib/db-btree.t
perl5.004/t/lib/db-hash.t
perl5.004/t/lib/db-recno.t
perl5.004/t/lib/dirhand.t
perl5.004/t/lib/english.t
perl5.004/t/lib/env.t
perl5.004/t/lib/filecache.t
perl5.004/t/lib/filecopy.t
perl5.004/t/lib/filefind.t
perl5.004/t/lib/filehand.t
perl5.004/t/lib/filepath.t
perl5.004/t/lib/findbin.t
perl5.004/t/lib/gdbm.t
perl5.004/t/lib/getopt.t
perl5.004/t/lib/hostname.t
perl5.004/t/lib/io_dup.t
perl5.004/t/lib/io_pipe.t
perl5.004/t/lib/io_sel.t
perl5.004/t/lib/io_sock.t
perl5.004/t/lib/io_taint.t
perl5.004/t/lib/io_tell.t
perl5.004/t/lib/io_udp.t
perl5.004/t/lib/io_xs.t
perl5.004/t/lib/ndbm.t
perl5.004/t/lib/odbm.t
perl5.004/t/lib/opcode.t
perl5.004/t/lib/open2.t
perl5.004/t/lib/open3.t
perl5.004/t/lib/ops.t
perl5.004/t/lib/parsewords.t
perl5.004/t/lib/posix.t
perl5.004/t/lib/safe1.t
perl5.004/t/lib/safe2.t
perl5.004/t/lib/sdbm.t
perl5.004/t/lib/searchdict.t
perl5.004/t/lib/selectsaver.t
perl5.004/t/lib/socket.t
perl5.004/t/lib/soundex.t
perl5.004/t/lib/symbol.t
perl5.004/t/lib/texttabs.t
perl5.004/t/lib/textwrap.t
perl5.004/t/lib/timelocal.t
perl5.004/t/lib/trig.t
perl5.004/t/op/append.t
perl5.004/t/op/arith.t
perl5.004/t/op/array.t
perl5.004/t/op/assignwarn.t
perl5.004/t/op/auto.t
perl5.004/t/op/bop.t
perl5.004/t/op/chop.t
perl5.004/t/op/closure.t
perl5.004/t/op/cmp.t
perl5.004/t/op/cond.t
perl5.004/t/op/delete.t
perl5.004/t/op/do.t
perl5.004/t/op/each.t
perl5.004/t/op/eval.t
perl5.004/t/op/exec.t
perl5.004/t/op/exp.t
perl5.004/t/op/flip.t
perl5.004/t/op/fork.t
perl5.004/t/op/glob.t
perl5.004/t/op/goto.t
perl5.004/t/op/groups.t
perl5.004/t/op/gv.t
perl5.004/t/op/inc.t
perl5.004/t/op/index.t
perl5.004/t/op/int.t
perl5.004/t/op/join.t
perl5.004/t/op/list.t
perl5.004/t/op/local.t
perl5.004/t/op/magic.t
perl5.004/t/op/method.t
perl5.004/t/op/misc.t
perl5.004/t/op/mkdir.t
perl5.004/t/op/my.t
perl5.004/t/op/oct.t
perl5.004/t/op/ord.t
perl5.004/t/op/pack.t
perl5.004/t/op/pat.t
perl5.004/t/op/push.t
perl5.004/t/op/quotemeta.t
perl5.004/t/op/rand.t
perl5.004/t/op/range.t
perl5.004/t/op/re_tests
perl5.004/t/op/read.t
perl5.004/t/op/readdir.t
perl5.004/t/op/recurse.t
perl5.004/t/op/ref.t
perl5.004/t/op/regexp.t
perl5.004/t/op/repeat.t
perl5.004/t/op/runlevel.t
perl5.004/t/op/sleep.t
perl5.004/t/op/sort.t
perl5.004/t/op/split.t
perl5.004/t/op/sprintf.t
perl5.004/t/op/stat.t
perl5.004/t/op/study.t
perl5.004/t/op/subst.t
perl5.004/t/op/substr.t
perl5.004/t/op/sysio.t
perl5.004/t/op/taint.t
perl5.004/t/op/tie.t
perl5.004/t/op/time.t
perl5.004/t/op/undef.t
perl5.004/t/op/universal.t
perl5.004/t/op/unshift.t
perl5.004/t/op/vec.t
perl5.004/t/op/write.t
perl5.004/t/pragma/constant.t
perl5.004/t/pragma/locale.t
perl5.004/t/pragma/overload.t
perl5.004/t/pragma/strict-refs
perl5.004/t/pragma/strict-subs
perl5.004/t/pragma/strict-vars
perl5.004/t/pragma/strict.t
perl5.004/t/pragma/subs.t
perl5.004/t/pragma/warn-1global
perl5.004/t/pragma/warning.t


The following is a summary of the results of each of the above tests
An "*" means there is additional information available following the
summary.


/perl/t/base - PERLT01

     cond.t      passed
     if.t        passed
    *lex.t       passed (Fixed 15SEP1997 - see note #4)
     pat.t       passed
     term.t      Failed (too much dependence on UNIX)

                                                                               .
/perl/t/cmd  - PERLT02

     elsif.t     passed
     for.t       passed
     mod.t       passed
     subval.t    passed
     while.t     passed


/perl/t/comp - PERLT03

     cmdopt.t    passed
     colon.t     passed
     cppaux.t    Failed (Can't emulate -P on #! line at perlt03.cppaux line 1)
     decl.t      passed
     multili.t   Failed (backtick)
     package.t   passed
     proto.t     passed
     redef.t     passed
     script.t    Failed (backtick)
     term.t      passed
     use.t       Failed (eval "use lib 1.01" & similar statements)

/perl/t/io   -  PERLT04

     argv.t      Failed (backtick)
     dup.t       Failed (open '>&xxx')
     fs.t        Failed (backtick, link, chmod, stat, rmdir, mkdir)
     inplace.t   Failed (backtick, inplace edits)
     pipe.t      Failed (fork not implemented)
     read.t      Failed (open '+>xxx' and seek)
     tell.t      Failed (seek, maybe tell)

/perl/t/lib   - PERLT05

     abbrev.t    passed
     anydbm.t    Failed (DBM package not installed)
     autoloa.t   Failed (mkdir required, might work otherwise)
     basenam.t   passed
     bigintp.t   Failed (40 of 287 tests failed)
     checktr.t   Failed (depends on UNIX or UNIX like file system)
    *complex.t   passed (fixed 19SEP1997, see note #6)
     dbbtree.t   Failed (ext/ not installed)
     dbhash.t    Failed (ext/ not installed)
     dbrecno.t   Failed (ext/ not installed)
     dirhand.t   Failed (object creation)
     filepat.t   Failed (mkdir not supported)
     findbin.t   Failed (Cannot chdir to $D1.ADLERPT/ -- file name spec)
     gdbm.t      Failed (ext/ not installed)
    *getopt.t    passed (fixed 24SEP1997, see note #7)
     hostnam.t   Failed (hostname not implemented)
     iodup.t     Failed (/ext not installed)
     iopipe.t    Failed (/ext not installed)
     iosel.t     Failed (/ext not installed)
     iosock.t    Failed (/ext not installed)
     iotaint.t   Failed (/ext not installed)
     iotell.t    Failed (/ext not installed)
     ioudp.t     Failed (/ext not installed)
     ioxs.t      Failed (/ext not installed)
     ndbm.t      Failed (/ext not installed)
     odbm.t      Failed (/ext not installed)
     opcode.t    Failed (/ext not installed)
     open2.t     Failed (fork not implemented, /ext not installed)
     open3.t     Failed (fork not implemented, /ext not installed)
     ops.t       Failed (/ext not installed)
     parsewo.t   Failed (thinks subroutine name is a script name)
     posix.t     Failed (/ext not installed)
     safe1.t     Failed (/ext not installed)
     safe2.t     Failed (/ext not installed)
     sdbm.t      Failed (/ext not installed)
     searchd.t   Failed (tries to create file w/illegal Guardian name)
     selects.t   Failed (tries to create file w/illegal Guardian name)
     socket.t    Failed (TCP/IP not implemented)
     soundex.t   passed
     symbol.t    passed
     texttab.t   passed
     textwra.t   passed
     timeloc.t   passed (sort of:  there's a wierd library deal going on)
    *trig.t      passed (Fixed 19SEP1997 see note #6)

/perl/t/op/*.* - PERLT06

     append.t    passed
     arith.t     passed
     array.t     passed (Fixed 19SEP1997 - see note #6)
     assignw.t   passed
     auto.t      passed
     bop.t       passed (Fixed 19SEP1997 - see note #6)
     chop.t      passed
     closure.t   Failed (closeure.t looks like it goes into a loop)
     cmp.t       passed
     cond.t      passed
     delete.t    passed
     do.t        passed
     each.t      passed
     eval.t      passed
     exec.t      Failed (exec not supported)
     exp.t       passed
     flip.t      passed
     fork.t      Failed (fork not supported)
     glob.t      Failed (file globbing not supported)
     goto.t      Failed (backtick not supported)
     groups.t    Failed (Security stuff not implemented)
     gv.t        passed
     inc.t       passed (Fixed 19SEP1997 - see note #6)
     index.t     passed
     int.t       passed
     join.t      passed
     list.t      passed
     local.t     passed (Fixed 19SEP1997 - see note #6)
     magic.t     Failed (Use of uninitialized value at perlt06.magict line 28)
     method.t    passed (this is odd: OO stuff had a problem above)
     misc.t      Failed (probably, but I'm not sure what real result is)
     mkdir.t     Failed (mkdir not supported)
     my.t        passed (Fixed 19SEP1997 - see note #6)
     oct.t       passed (fixed 18SEP1997 - was crashing during compilation)
     ord.t       passed
     pack.t      Failed (some tests failed)
     pat.t       passed
     push.t      passed (Fixed 19SEP1997 - see note #6)
     quotme.t    passed
     rand.t      passed (it seems, anyway)
     range.t     passed
     readdir.t   Failed (19SEP1997 - 1 passed, 2 failed, 3 no output)
     read.t      Failed (1 test of 4 failed)
     recurse.t   Failed (some functions never terminate in my lifetime)
     ref.t       passed
     regexp.t    Failed (Reg exp work, so I don't know what's going on here)
     repeat.t    passed
     runleve.t   Failed (tests failed)
     sleep.t     Failed (sleep returned incorrect value)
     sort.t      passed (Fixed 19SEP1997 - see note #6)
     split.t     passed
     sprintf.t   passed
     stat.t      Failed (stat not fully supported)
     study.t     passed
     substr.t    passed
     subst.t     Failed (3 out of 60 tests failed)
     sysio.t     Failed (50% of the tests failed)
     taint.t     Failed (Too late for "-T" option at perlt06.taintt line 1.)
     tie.t       Failed (tests failed)
     time.t      Failed (times not implemented at perlt06.timet line 8.)
     undef.t     passed
     univers.t   passed
     unshift.t   passed
     vec.t       passed
     write.t     Failed (backtick not implemented)

/perl/t/pragma/*.* - PERLT07

     constan.t   passed (Fixed 19SEP1997 - see note #6)
     locale.t    Failed (Too late for "-T" option at perlt07.localet line 1.)
     overloa.t   Failed (4 out of 115 tests failed)
     strict.t    Failed (no output)
     subs.t      Failed (Tests failed)
     warning.t   Failed (no output)



NOTES on tests:

1.  /perl/t/base/lex.t                            (01SEP1997)

    This failed when PERL tried parse a 10 digit number.  The crash
    resulted from an arithmetic overflow.  Evidently, other C runtimes
    will allow this to occur without any disasters.

    So, if you get an arithmetic overflow with any of your scripts, it's
    probably because a value has exceeded the largest value possible for
    a 32 bit integer.  For example, this will fail:

            $foo = int ($bar);      # where $bar is a float > 2147483647.0


2.  /perl/t/lib/complex.t                               (03SEP1997)
    /perl/t/lib/trig.t

    Perl crashes during compilation when it tries to compile one or another
    of the following:

            use constant pi => 4 * atan2(1, 1);
            use constant log10inv => 1 / log(10);

    The failure is an arithmetic overflow in sv_upgrade.  The double

            4294967293.

    is being cast into a 32 bit integer which C can't do because the
    maximum integer value is 2147483647.  We don't know where this
    number comes from.

    This means that you can't use the trig or complex number functions
    from the library.

3.  /perl/t/lib/getopt.t                              (03SEP1997)

    This is similar to #2 above.  The crash is an arithmetic overflow
    during a cast from double to int.  The location in the code is the
    exact same place as #2 and the number that is being dealt with is
    almost the same:

            4294965757.


    instead of:

            4294967293.

    Both of these numbers, when cast as (long long) end up looking like
    negative 32 bit integers.  In the first case it's FFFFF9FD or -1539
    and in the second it looks like FFFFFFFD or -3.

    This suggests that some arithmetic is going wrong somewhere or maybe
    an ABS function.  Too soon to speculate.


4.  More on #2 and #3 above:                           (15SEP1997)

    Here's a reliable case that causes the exact same crash

          # t23 - complement test
          #
          #
          $b = 17;
          $b &= ~1;
          print "$a, $b\n";

    Evidently, Perl is treating ~1 as ULONG_MAX - 1 instead of -2.  And
    because ULONG_MAX is > LONG_MAX Perl converts this thing to a double.
    Well that's okay until it has to convert it back to a long.  That's
    when we get into trouble.  We can't convert directly from double to
    long.  We have to convert from double to unsigned long to long.

    This has been fixed by changing the macros:  I_32 and I_V as follows:

         #ifdef CASTI32
         #   ifndef _GUARDIAN_TARGET
         #      define I_32(what) ((I32)(what))
         #      define I_V(what) ((IV)(what))
         #   else
         #      define I_32(what)   ((I32)(UV)(what))
         #      define I_V(what) ((IV)(UV)(what))
         #   endif
         #else
         #   etc. etc.

    That is, instead of going directly from double to long we cast
    to unsigned long first.

5.  /perl/t/lib/complex.t                         (18SEP1997)
    /perl/t/lib/trig.t
    /perl/t/lib/getopt.t

    See note #4 before reading this one.

    After fixing the problem with "~" (complement), Perl is now crashing
    during execution with the following error in both tests.

      \PHAMIS.$:5:242:411332716 - Invalid function parameter
      \PHAMIS.$:5:242:411332716 - From Perl_pp_padav + %246, UC.01


6.  /perl/t/op/array.t                            (18SEP1997)

    See note #3 before reading this one.

    After fixing the arithmetic overflow problem, we are now able to
    start executing the test.  But now, Perl crashes during execution.

            *** Run-time Error 040 ***
            Invalid function parameter
            From Perl_pp_rv2av + %1052, UC.02
            Perl_runops + %143, UC.02
            perl_run + %546, UC.00
            main + %57, UC.00
            _MAIN + %33, UC.00

    There are two things to pay attention to here:  first, this error is
    occured in the following bit of script:

            @foo = ();
            $r = join(',', $#foo, @foo);

    And went away when I added:

            @foo = (1, 2, 3);

    as the first statement.  Then the test passed completely.  Then,
    secondly, when I removed the first line from the script, it worked
    perfectly.

    Go figure.

    In the debugger the problem was occuring in the following
    C statement:

             Copy(AvARRAY(av), SP+1, maxarg, SV*);

    which invokes the macro:

             (void)memcpy((char*)(d),(char*)(s), (n) * sizeof(t))

    AvARRAY(sv) is equal to 0, which is why memcpy is complaining.

    When I tried this same code in the WIN32 version, there wasn't any
    problem.

    So, here's what I suspect:

    Perl doesn't know that the array it's moving is empty and it doesn't
    know that the destination doesn't really exist yet.  So it happily
    goes off to move 0 bytes into outerspace.  Evidently in the
    Guardian C compiler if outerspace is NULL then we get a run-time
    error.

    I think the fix is to change the Copy MACRO so that if the length
    is zero, it doesn't try to execute memcpy.

    Postscript to #6:  I implmented the change to the copy MACRO and
    that eliminated every single one of the crashes.  A new symptom
    showed up that is related to how I tortered the GETS routine
    into performing.


7.  /perl/t/lib/getopt.t                          (19SEP1997)

    The test ran to completion, but when the script tried to close a
    file:

              *** Run-time Error 034 ***
              Released space not allocated
              From GUARDIAN_FCLOSE + %21, UC.03
              PerlIO_close + %5, UC.01
              Perl_io_close + %175, UC.00
              Perl_do_close + %176, UC.00
              Perl_pp_close + %63, UC.02
              Perl_runops + %143, UC.02
              perl_run + %546, UC.00
              main + %57, UC.00
              _MAIN + %33, UC.00

    Perl is trying to close STDERR twice.  It's the second time through
    that is causing the error.  Oddly, the the first time STDERR is
    closed is during execution of pp_open.

    The way we redirect STDERR in GUARDIAN C is to use an ASSIGN.  I
    think what's going on might have something to do with the special
    status of STDERR in UNIX.

    (21SEP1997)

    The runtime library routine, fileno, returns 0, 1, 2 for stdin, stdout
    and stderr respectively.  these aren't good numbers.  So, when perl
    starts messing around with STDERR, it tries to do some stuff with
    file number 2 and gets into trouble.

    MAXSYSFD is a variable that controls (I think) which numbers are
    defined as system files (i.e. stdin, stdout and sterr).

    So, if we are to support the following construct:

         open STDERR, ">foo" or die "stderr: $!\n";
         print STDERR "this is an error?\n";
         close STDERR;

    and have something actually show up in foo, we're going to have to
    allow open, close, etc. to work on these wierd files.

    (24SEP1997)

    Turns out the WIN32 version of perl doesn't quite work here either.
    We'll make the GUARDIAN version work like WIN32.

    Once you open STDERR (or STDOUT) with a different file, you can't
    switch back to where STDERR was originally pointing.

    So the above script will work but if you then try to print to
    STDERR after the close, you'll get a "write on unopened file"
    warning.


COMPILATION INSTRUCTIONS
------------------------

Unless you start messing the source code or if the executable provided
in this distribution doesn't work on your system, you shouldn't have
to re-compile the source.  But that's the programmer's kiss of death so
here is what you do if you have to recompile:

A) Complete Recompilation

If you want to do a complete recompilation, purge all of the files in
PERLOBJ (this is where all of the non-executables end up).

There is a cheesy little MAKE macro that will recompile and rebind Perl.
To use it:

     1.  VOLUME PERLMNT
     2.  RUN MAKE MKPERL
     3.  GO TO COFFEE (I live in Seattle so that's obligatory)

It will do what you'd expect a cheesy little MAKE macro to do and either
generates an executable or quits when something goes wrong.

If you are compiling on a system that has NMC it will use it and NLD to
to build the PERLO executable (ignore all of the warnings that are
printed -- you can't turn them off in NMC).

If your system does not have NMC the executable that gets generated is
called PERLUX (meaning PERL unaccellerated).  You should accellerate
the code using:

     > AXCEL PERLUX PERLO
     > BIND STRIP PERLO

On a system with NMC, you can force the use of Guardian C by including:

     "COMPILER CISC"

at the beginning of the MKPERL file.


B)  Partial Recompilation

In some cases, after you know how the pieces fit together, you can recompile
individual files and just rebind (even though MAKE would recompile more
or even the whole thing).

There is a macro called PERLCOMP that compiles one source file.

     > VOLUME PERLMNT
     > RUN PERLCOMP <file>

where <file> is the name of the source code file without the C suffix.
PERLCOMP always uses NMC if it's present (you'll have to edit the file
to force it to use Guardian C).

Then when your ready to rebind everything you can do:

     > RUN PERLBIND

This does the same bind that the MAKE does.  PERLBIND always uses NLD if
it is present.  You'll have to edit the file if you want to force the
use of Guardian C.

Again, partial recompilations are not recommended unless you kind of
know what you're doing.  Particularly if you're making header file
changes.



TACL SUPPORT
------------

As alluded to above, in many places, we've added a bunch of TACL to
make the use of Perl on Guardian a little more natural than it would
otherwise be.

Aside from the source code maintanence macros mentioned in the previous
section the following TACL macros and libraries are distributed:

     PERLLOCL

     This is a file which creates all of the defines necessary for Perl
     to find its libraries.  It uses the TACL builtins ("#" commands)
     instead of the ADD DEFINE commands and, as a result, it is very
     fast (you can't tell that it exeutes).

     PERLLOCL is executed under two different circumstances:

     1)  when the timestamp changes (i.e. when its been modified)
     2)  when a "signal" define is missing (i.e. when the defines have
         never been loaded).


     PERLMAC

     This is a TACL library containing, among other things, the PERL
     command that runs the PERLO executable.  It is responsible for
     keeping tabs on the current state of PERLLOCL and, if present
     PERLCSTM (which is your own personal PERL wrapper).

     When PERLMAC has been loaded, and you enter, for example:

          > perl foo bar > outfile

     the routines in PERLMAC build a new set of commands:

          > SINK [#PURGE OUTFILE]
          > PERL/OUT OUTFILE/FOO BAR

     As we mentioned previously, these routines will also expand file name
     template parameters into a space separated list.


     PERL

     This is a "bootstrap" macro.  It's only job in life is to load PERLMAC
     and then re-run the command you used.  So, if your #PMSEARCHLIST
     inclues the volume and subvolume that contains PERLMAC (and PERLO) the
     PERL macro will first load PERLMAC and then re-execute exactly the
     same thing you typed.  Once the PERLMAC library has been loaded, the
     only way you can execute this macro again is by fully qualifying it
     because "PERL" is now a variable and variables take precendence over
     file names.


     BTPROMPT

     As you may recall from reading about the stuff that doesn't work,
     `cmd` doesn't work on Guardian.  I thought about trying to make it
     work, but in the end, the method I came up could be done using
     Perl itself (along with the "sysio" functions I implemented).  So
     instead of making `cmd` work, we implemented a module called BACKTICK
     (in PERLLIB).  One of the components of this TACL macro which changes
     the TACL prompt to something very recognizable.  See "Stuff that works"
     item 11.


As we began to use Guardian PERL for more serious work, we found the
need to add some more TACL level support (It would be a good idea to
review the material in the previous section labeled "LIBRARY" before
trying to read this stuff):

      PARAM PERL-DEV

      One of the problems that arises because we didn't port @INC
      functionality very well is the need for a "private" development
      libraries.  These are places where developers could put alpha
      or beta copies of ".pm" type scripts until they were ready for
      prime time.

      While the preferred method of handling this problem is using
      PERLCSTM, described previously, sometimes you want something
      even more "temporary".  To that end we added support for the
      PERL-DEV parameter.  It allows you to specify a temporary
      library subvolume where PERL can find test versions of libraries.

      For example when you say in your script:

          use FOOBAR;

      PERL normally expects to find FOOBAR in one of the supported
      library sub-volumes.  However, when you are just starting out
      to develop this module, you don't want anyone else to use it.
      So you say:

          TACL> PARAM PERL-DEV "<subvol>"

      prior to running your PERL script.  <subvol> is where FOOBAR
      is located.  This will add <subvol> to the list of sub-volumes
      where PERL looks for library routines.

      NOTES:

      1.  When PERL-DEV is in effect the search list order is:

               - distributed modules (for example, strict)
               - the PERL-DEV sub-volume
               - PERLLIB

          So if you are writing a new version of a module that was
          in the original distribution, be sure to name it different.

      2.  the use of PERL-DEV will cause the re-execution of PERLLOCL
          upon each execution of PERL.  So, you should move your code
          to PERLLIB after testing, and then do away with the PERL-DEV
          parameter (CLEAR PARAM PERL-DEV).



      RUNNING A PERL SCRIPT FROM A MACRO

      When you want to execute a PERL script from within a TACL macro
      you have to be careful to preserve global TACL variables that
      the PERL support macros create and use.

      The naive approach is:

          ?TACL MACRO
          #FRAME
             do some set-up work
                  :

             perl <your script>

                  :
             do some finish work
          #UNFRAME

      There is a minor performance hit if you do it this way.  To avoid
      this there is a routine you can run prior to the #FRAME.  The
      new macro would look like this:

          ?TACL MACRO
          PERLINIT
          #FRAME
             do some set-up work
                  :

             perl <your script>

                  :
             do some finish work
          #UNFRAME

      Notice the invocation of PERLINIT prior to the #FRAME.  What's
      going on here is PERLINIT is invoking all of the TACL code
      that would normally get invoked the first time you run PERL
      from a particular TACL session.  The TACL variables the PERL
      support macros require are created outside the frame your
      application macro creates.

      Another macro is available that you can run after the #UNFRAME
      or in your TACL exception handler code:

          PERLKILL

      This routine will delete all the global variables and
      the =perl* definitions.  This causes your TACL session to forget
      that PERL ever existed.

      As a rule, you don't need to use PERLINIT and PERLKILL if the
      macro you are writing is run only once per TACL session (let's
      say, as part of a NETBATCH job).  If, on the other hand, you are
      creating a tool that many people will be using throughout the day
      then the use of PERLINIT, in particular, could save lots of
      I/O and cycles.  As always you've got to think about the
      application and how it's likely to be used.




NNC NOTES
---------

The native mode compiler/linker assumes the OSS system calls
and the NMC runtime library both of which are different then
the Guardian C runtime.  In particular, the "open" used by NMC
returns a different kind of number than the "open" in the C
runtime.

The original Guardian extension software (GUARD.C) made the
assumption that both "open" and "FILE_OPEN_" returned Guardian
file numbers.  This was true enough for the code to work.
Under NMC, "open" returns file numbers starting at 0 as does
"FILE_OPEN_" so it was no longer possible to mix "open" files
with "FILE_OPEN_" files.

The modification we made to support both NMC and Guardian C
was to conceptually divide the file number space into OSS
numbers and GUARDIAN numbers.  Whenever a file was opened using
"FILE_OPEN_" we returned the file number with MIN_GUARDIAN_FD
added to it.  Whenever we processed any request for data using
a file number we tested the file number against this value.
If the number was less than MIN_GUARDIAN_FD then we used the
OSS system calls (close, read, write).  If the number is greater
than or equal to MIN_GUARDIAN_FD then we used Guardian I/O
(READX, WRITEX, REPLYX, etc).

This scheme works regardless of the compiler used to build Perl.

Also, FASTGETS doesn't work with NMC so we disable it if the NMC
compiler is used.  This means we always use the runtime GETC which
is a little slower but hopefully, NMC has optimized this a little.

The "CC" function in the PERLMNT.MAKE sub-volume is also modified.
If NMC is present then use it instead of Guardian C.  An important
difference between NMC and Guardian C is the error checking level.
In Guardian C, we get away with all kinds of loose constructs
without any complaints from the compiler.  However in NMC every
little cotten picking anomolie is picked up and complained about
with WARNING messages.  Consequently, we have to ignore
completion codes = 1 for NMC and hope that the warning wasn't
anything we introduced.

Over time, as we work on this code, we should endeavor to fix
the things that NMC complains about.  There are a lot of them
so this is sort of a life's work.



LICENSING
---------

Please read the LICENSE file in the PERL subvol.  I have attempted to
be complient with all provisions.  If I did not it was through my
inability to deal with the legalese and not through any malice.

1.  All of the source code that makes up this port is contained in
    the PERLSRC sub-volume.

2.  If you want a copy of this source code, just ask via the phone numbers
    or email addresses listed above.

3.  NO WARRENTY:  QUOTED FROM THE LICENSE.

  11. BECAUSE THE PROGRAM IS LICENSED FREE OF CHARGE, THERE IS NO WARRANTY
FOR THE PROGRAM, TO THE EXTENT PERMITTED BY APPLICABLE LAW.  EXCEPT WHEN
OTHERWISE STATED IN WRITING THE COPYRIGHT HOLDERS AND/OR OTHER PARTIES
PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED
OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.  THE ENTIRE RISK AS
TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU.  SHOULD THE
PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING,
REPAIR OR CORRECTION.

  12. IN NO EVENT UNLESS REQUIRED BY APPLICABLE LAW OR AGREED TO IN WRITING
WILL ANY COPYRIGHT HOLDER, OR ANY OTHER PARTY WHO MAY MODIFY AND/OR
REDISTRIBUTE THE PROGRAM AS PERMITTED ABOVE, BE LIABLE TO YOU FOR DAMAGES,
INCLUDING ANY GENERAL, SPECIAL, INCIDENTAL OR CONSEQUENTIAL DAMAGES ARISING
OUT OF THE USE OR INABILITY TO USE THE PROGRAM (INCLUDING BUT NOT LIMITED
TO LOSS OF DATA OR DATA BEING RENDERED INACCURATE OR LOSSES SUSTAINED BY
YOU OR THIRD PARTIES OR A FAILURE OF THE PROGRAM TO OPERATE WITH ANY OTHER
PROGRAMS), EVEN IF SUCH HOLDER OR OTHER PARTY HAS BEEN ADVISED OF THE
POSSIBILITY OF SUCH DAMAGES.



INSTALL INSTRUCTIONS
--------------------

****************************************************************************
NOTE:  If you already have an earlier version of this port of Perl
for Guardian installed, you must remove it from you system before
trying to install this version.  If you placed your own Perl scripts
in any of the pre-defined Perl sub-volumes, please set them aside while
doing this this install.
****************************************************************************

The following install instructions assume you are installing from the
ZIP file and it is on your PC.  Also note that I don't talk about security
here at all.  Once the Perl distribution is on the Tandem you'll need
to secure the files so that only authorized people can access them.

The archiver/de-archiver referenced in the following discussion is a set
of programs called PAK/UNPAK available from the ITUG library.  These
programs use the Guardian programs BACKUP and RESTORE (respectively).


Step 1:  Expand the ZIP file (using PKUNZIP or WINZIP or whatever).

     PKUNZIP tandem-perl-v2.zip

What you'll end up with are the following Tandem files:

     File      Code      What
     --------- ----      -------------------------------------
     GPERL     100       A self-extracting archive file

And the following ASCII files:

     README.TXT          This file
     FILELIST.TXT        The output from the PAK
     WHATSNEW.TXT        A short description of new stuff in this release



Step 2:  Upload to Tandem.

Upload GPERL a single subvolume.  Make sure your FTP is in Binary Mode.
The volume probably should NOT be $vol.subvol.PERL


Step 3:  FUP ALTER file codes for GPERL.


     FUP ALTER GPERL,CODE 100


Step 4:  De-archive the perl files.

     RUN GPERL/out <spooler>/ ,*.*.*,VOL <disk volume>,MYID, LISTALL

     where:  <spooler> is where you want the output to go
             <disk volume> is the disk drive where you want PERL to end up

     MYID is the RESTORE command to cause files to be created using the
     currently logged on user.

     LISTALL is the RESTORE command to generate a listing of all of the files
     extracted from the archive.

     *NOTE*:  The comma in front of "*.*.*" is required because the command
     tail is passed verbatum to RESTORE following the "<tape>" device
     parameter.

Step 5:  Run the PLINSTAL macro.

Once you get all of the files that belong to PERL onto the disk, you
need to run the PLINSTAL macro.

The volume and prefix are hard coded in a couple of places.  The
PLINSTAL macro figures out what the volume and prefix is and fixes up
those places where it's been hard coded.

Don't freak-out.  The hard coding is not any of the actual PERL software.
We hard code the volume and prefix in the PERLMAC TACL library which has
to know where everything is and can't figure it out on the fly (because
it's a library) and we also store the volume and prefix in a file called

    PERLPFIX

This file is read by PERLLOCL to figure out where all of the PERL library
scripts are stored (see the section above on libraries).

To run the PLINSTAL macro, make sure you're logged on as someone who can
modify the files in the main PERL subvolume (where the executable is).
Then from whereever you happen to be enter at the TACL prompt:

    RUN <vol>.<prefix>.PLINSTAL

Answer the question (Continue?  Y/N) and that's it.


ADDITIONAL INSTALLATION NOTES:
------------------------------

PERL must physically reside on a single VOLUME (you can always fuss with
the files if you don't want this to be true, but we recommend you don't
mess around with anything until you've had some experience with this port).

The subvolume names PERL uses are all prefixed with a string 5 chracters
or less.  For purposes of this explanation we will assume you are installing
PERL on $SYSTEM and you are using the suggested prefix of "PERL".

This PERL distribution consists of the following subvolumes:

    $SYSTEM.PERL
    $SYSTEM.PERLEXT
    $SYSTEM.PERLEXTT
    $SYSTEM.PERLL01
    $SYSTEM.PERLL02
    $SYSTEM.PERLL03
    $SYSTEM.PERLL04
    $SYSTEM.PERLL05
    $SYSTEM.PERLL06
    $SYSTEM.PERLL07
    $SYSTEM.PERLL08
    $SYSTEM.PERLL09
    $SYSTEM.PERLLIB
    $SYSTEM.PERLMD5
    $SYSTEM.PERLMNT
    $SYSTEM.PERLOBJ
    $SYSTEM.PERLSRC
    $SYSTEM.PERLT01
    $SYSTEM.PERLT02
    $SYSTEM.PERLT03
    $SYSTEM.PERLT04
    $SYSTEM.PERLT05
    $SYSTEM.PERLT06
    $SYSTEM.PERLT07


Again, the volume you choose may be different as might be the prefix.

An aditional sub-volume is distributed:  $SYSTEM.PERLPL which contains
application PERL scripts and is not technically part of PERL itself.
These scripts can be anywhere you want.  These scripts were developed
locally for a variety of uses.

Since the original sub-volume names are prefixed with "PERL", it is likely
that you won't want to go through the trouble of changing this.  However,
if you already have a sub-volume on the target disk with same name as
one in the above list, you'll need to either move it, or move perl to
a temporary disk and then use the RENAME command to change sub-volume names
before moving all of the files to their final home.

The volume and prefix are hard coded in a couple of places.  The
PLINSTAL macro figures out what the volume and prefix is and fixes up
those places where it's been hard coded.

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

               DOCUMENT/SOFTWARE CHANGE HISTORY
               --------------------------------

Date         Who    What
---------    ---    ---------------------------------------------------
27Jan1998    CDA    Added info about ARRAYS not printing directly via
                    print @array;
30Jan1998    CDA    It is possible to read Type U files files and to
                    ask Perl whether a file is an edit file or not
                    <see below for a change>
08Feb1998    CDA    It is no longer possible for a script to tell
                    whether a file is an EDIT file or an UNSTRUCTURED
                    file.  It doesn't need to because both kinds of files
                    are treated the same way:  as "line oriented".
                    It is now possible to read and write ENSCRIBE files.
                    A script can ask weather the file is an Enscribe
                    file or not.  Perl can read any kind of
                    unstructured file but can write to only ODDUNSTR
                    files.
24Feb1998    CDA    Perl scripts can be written as servers or requesters.
                    Perl scripts have access to the following Guardian
                    routines:  ABORTTRANSACTION, BEGINTRANSACTION,
                    ENDTRANSACTION, KEYPOSITION, POSITION, CONTROL AND
                    SETMODE.
24Mar1998    FLC    No longer need to load perl.perlmac prior to
                    running perl.  Perlmac will be loaded via the PERL
                    bootstrap macro if it had not already been loaded
                    via a previous PERL command in the session.  The
                    PERL.PERL 100 file has been renamed to
                    PERL.PERL0.
10Apr1998    CDA    Fixed it up for the upload to CPAN.
10Sep1998    CDA    No longer need defines for individual library
                    files.  We now use FILENAME_RESOLVE_ and "SEARCH"
                    defines.
11Nov1998    CDA    Added MD5 extensions and a documentation file
                    called HOWTOEXT which explains how to extend
                    PERL for guardian.  Added PERLEXT library
                    subvolume.  Fixed BYTEORDER in CONFIGH (turns
                    out it was ending up 0x1234 and it should be
                    0x4321 and evidently only MD5 cared).
20Jan1999    CDA    Made Perl compilable by both Guardian C (CISC) and
                    NMC (RISC).  See "NMC Notes" for details.  Also,
                    fixed PERLMNT.MAKE and PERL.PERL TACL macros so they
                    will work at non-IDX sites (references were made to
                    routines/macros that are locally defined).  Combined
                    README and README2 files into a single file.

