______________________________________________________________________________  
                                                                                
Title: Rexx with Unix commands or executing another rexx given as parameter.    
       and miscellaneous rexx.                                                  
Author: Marius Lewin.                                                           
______________________________________________________________________________  
                                                                                
This summary is divided in three parts:                                         
1) rexx executing z/OS Unix commands.                                           
2) rexx executing another rexx given as parameter.                              
3) Miscellaneous rexx.                                                          
______________________________________________________________________________  
                                                                                
rexx executing z/OS Unix commands                                               
                                                                                
   unix                                                                         
        Rexx and edit macro.                                                    
        Executes BPXWUNIX.                                                      
                                                                                
        If used as an edit macro,                                               
           1) all the parameters constitute the Unix command, and               
           2) the edited member or file is used as STDIN.                       
                                                                                
        If used as a rexx,                                                      
        if the last parameter is a valid dsn or dsn(mbr) then                   
           1) the last parameter is used for STDIN and                          
           2) only the previous parameters constitute the Unix command.         
        otherwise all parameters constitute the Unix command.                   
                                                                                
        In View, edit macro unix invokation:                                    
        Command ===> unix grep -E 'ab.c'                                        
        will obtain the dsname and member name and have bpxwunix                
        execute with command                                                    
        grep -E 'ab.c'                                                          
        and dsname(member) as standard input.                                   
        Command ===> unix cal                                                   
                                                                                
        Examples of rexx use:                                                   
        Command ===> cmde      to preserve lower case options and regex         
        Enter TSO commands below:                                               
        ===> unix cal                                                           
        ===> unix grep -E  'ar *g\(|upp[ef]' userid.TEST(grep05)                
        ===> viewit unix grep -E  'ar *g\(|upp[ef]' userid.TEST(grep05)         
        ===> viewit forallm unix grep -E  'ar *g\(|upp[ef]' userid.TEST(*)      
         viewit forallm unix grep -E -n 'cmd' noview userid.TEST(*)             
         viewit foralld forallm unix grep -E -n 'cmd' noview userid.T*S%(*)     
                                                                                
         Example of batch use:                                                  
                                                                                
          //TSOSH    EXEC PGM=IKJEFT1B                                          
          //SYSTSPRT DD SYSOUT=*                                                
          //SYSTSIN  DD *                                                       
          unix cd /tmp; dd:tsosh arg1 arg2                                      
          //* dd:tsoh replaced by a temporary file in /tmp, thereafter removed  
          //SYSEXEC  DD DISP=SHR,DSN=XXXXXX.TEST  Contains UNIX rexx            
          //TSOSH    DD *,DLM=EOF                                               
          /*    rexx  Needed: / in column 1 and * in column 2 on first line */  
          /*                  rexx is first word in comment on first line.  */  
           trace r                                                              
           parse arg therest                                                    
           say "therest = "therest                                              
           say "abc"                                                            
           'cal' /* implicit address sh under z/OS Unix */                      
           exit                                                                 
          EOF                                                                   
                                                                                
   UNIXEDIT                                                                     
        UNIXEDIT is a JCL procedure.                                            
        Edition of members of a library with z/OS Unix commands,                
        or execution of z/OS Unix commands.                                     
         Description of the two parameters DSNIN and DSNOUT:                    
                                                                                
          DSNIN is the input library.                                           
          DSNOUT is the output library.                                         
          If DSNOUT=DUMMY or DSNOUT=NULLFILE, we do not write                   
             to an output LIBRARY.                                              
          Standard output is copied to the SYSTSPRT file.                       
          If (DSNOUT= with nothing at right of the equal sign                   
             or is not present)                                                 
          and DSNIN was specified with a library name,                          
             then an output library with name, the input library                
             name suffixed by '.OUT' is created, or, if it exists,              
             its members will be replaced.                                      
          If DSNOUT is specified with a dataset name,                           
             (e.g. DSNOUT=BOZO.PDSE.OUTPUT)                                     
             the output dataset, if it does not exist, is created,              
             otherwise, its members will be replaced.                           
                                                                                
          //    SET DSNIN=BOZO.CNTL                                             
          //    SET DSNOUT=BOZO.CNTLOUT                                         
          //    EXEC UNIXEDIT                                                   
             is the same as                                                     
          //    EXEC UNIXEDIT,DSNIN=BOZO.CNTL,DSNOUT=BOZO.CNTLOUT               
                                                                                
          Structure of the rexx executed in this procedure:                     
                                                                                
          1) We copy the input library into a temporary z/OS Unix               
             directory.                                                         
          2) With commands in the //BEGIN file, the user may process            
             the copied members or issue other z/OS Unix commands.              
             (//SYSIN may be used instead of //BEGIN).                          
          3) We set up a   for mbr in *; do ... ; done   loop.                  
             THe user may include z/OS Unix commands in the                     
             file //THIS$MBR. These commands replace the three dots             
             in   "for mbr in *; do ... ; done"                                 
             The name of the member operated upon is $mbr.                      
             Example of commands replacing the three dots:                      
             echo "Member $mbr"                                                 
             head -n5 $mbr                                                      
          4) With commands in the //END file, the user may process              
             the members in the temporary directory or issue                    
             other z/OS Unix commands.                                          
          5) We copy with replacement all members still present                 
             in the temprorary directory to the output library.                 
                                                                                
          Examples:                                                             
          _______________________________________________________               
           Create or replace BOZO.CNTL.OUT                                      
           (No backout copy of BOZO.CNTL.OUT, if it exists.                     
           May be used first to check out)                                      
           //    EXEC UNIXEDIT,DSIN=BOZO.CNTL                                   
           followed by control cards                                            
                                                                                
           Create or replace BOZO.CNTL2                                         
           (No backout copy of BOZO.CNTL2, if it exists)                        
           //    EXEC UNIXEDIT,DSIN=BOZO.CNTL,DSNOUT=BOZO.CNTL2                 
           followed by control cards                                            
                                                                                
           Create or replace members in BOZO.CNTL                               
           (No backout copy)                                                    
           //    EXEC UNIXEDIT,DSIN=BOZO.CNTL,DSNOUT=BOZO.CNTL                  
           followed by control cards                                            
          _______________________________________________________               
           Replace in all members of a library,                                 
           in all lines containing 'HERE', 'BEFORE' by 'AFTER '                 
           //    EXEC UNIXEDIT,DSIN=BOZO.CNTL,DSNOUT=BOZO.CNTL                  
           //BEGIN DD *                                                         
           ls -al                                                               
           //THIS$MBR DD *                                                      
           sed '/HERE/s/BEFORE/AFTER /g' $mbr > $mbr".txt"                      
           mv $mbr".txt" $mbr                                                   
           //END DD *                                                           
           pwd                                                                  
          _______________________________________________________               
           Convert all members of a library to UTF-8 before                     
           unloading them in binary to Windows.                                 
                                                                                
           //    EXEC UNIXEDIT,DSNIN=BOZO.PDSE,DSNOUT=NULLFILE                  
           //BEGIN DD *                                                         
           mydir="/u/bozo/mydir"                                                
           rm -r $mydir                                                         
           mkdir $mydir                                                         
           //THIS$MBR DD *                                                      
           awk 'sub("$", "çr")'   $mbr       > $mbr".txt"                       
           iconv -f 1140 -t utf-8 $mbr".txt" > $mydir"/"$mbr                    
           //*                                                                  
           //* awk substitute to the null character before end of line,         
           //* a carriage return CR. End of line for Windows is CRLF (0x0D0A).  
           //* 1140 for IBM-1140 (US EBCDIC codepage)                           
          _______________________________________________________               
           Make a sequential file with each member preceded by a header         
           ./ADD NAME which may be used as input to the IEBUPDTE utility.       
           //    EXPORT SYMLIST=*                                               
           //    SET DSNIN=BOZO.CNTL         input library                      
           //    SET FLAT=&DSNIN..FLAT  Add .FLAT at end of name, output        
           //    EXEC DELDEF,DSN=&FLAT  Allocate output sequential file         
           //*                                                                  
           //    EXEC UNIXEDIT,DSNIN=&DSNIN,DSNOUT=NULLFILE                     
           //BEGIN DD *,SYMBOLS=EXECSYS                                         
           awk 'FNR==1éprint "./ADD NAME="FILENAMEè;1' * > flat                 
           cp flat "//'&FLAT'"                                                  
           //*                                                                  
           //* * all members                                                    
           //* FNR==1 at first record of each member                            
           //* 1 true, print current record                                     
           //* Only output is &FLAT sequential MVS file                         
                                                                                
           Alternative:                                                         
           //    EXEC UNIXEDIT,DSNIN=&DSNIN,DSNOUT=NULLFILE                     
           //THIS$MBR DD *                                                      
           (echo "./ADD NAME="$mbr; cat $mbr) >> flat                           
           //END DD *,SYMBOLS=EXECSYS                                           
           cp flat "//'&FLAT'"                                                  
          _______________________________________________________               
           Create or replace member DIR in default output                       
           library &DSNIN..OUT (as DSNOUT=).                                    
           If DSNOUT specifies a library which does not exist,                  
           it will be allocated.                                                
           Submit all members with name beginning by JOB.                       
           Copy calendar content of missing days september 1752 month           
           to allocated DDOUT file.                                             
           //    SET DSNIN=BOZO.CNTL         input library                      
           //*                                                                  
           //    EXEC UNIXEDIT,DSNIN=&DSNIN,DSNOUT=                             
           ls -al > DIR                                                         
           submit JOB*                                                          
           £ cp or mv cannot be piped into                                      
           cal sep 1752 > CAL      &&                                           
           cp CAL "//dd:ddout"     &&                                           
           rm CAL                                                               
           //DDOUT DD SYSOUT=*                                                  
          _______________________________________________________               
           With no imput or outyput library.                                    
           Present working directory is a temporary directory                   
           which will be removed at end of processing.                          
           Implicit //SYSIN DD card same as //BEGIN DD card.                    
           //    EXEC UNIXEDIT                                                  
           cal sep 1752 > CAL      &&                                           
           mv CAL "//dd:ddout"                                                  
           //DDOUT DD SYSOUT=*                                                  
                                                                                
           //    EXEC UNIXEDIT                                                  
           //IN    DD DISP=SHR,DSN=BOZO.SEQ                                     
           //OUT   DD SYSOUT=*                                                  
           cp "//dd:in" in                                                      
           wc in > out                                                          
           cp out "//dd:out"                                                    
                                                                                
           //    EXEC UNIXEDIT                                                  
           £ Obtain machine model with uname. E.g.: 8561 (z15)                  
           echo "Model: $(uname -m)"                                            
           £                                                                    
           £ Alternative: obtain machine model with MVS display command D M=CPU 
           £ Use _BPX_SHAREAS=YES with tso command                              
           £ Using a rexx in the SYSEXEC allocation:                            
           export SYSEXEC="alloc da('BOZO.EXEC') shr reuse"                     
           £ export SYSEXEC=BOZO.EXEC:BOZO.EXEC2     Ok                         
           £ c is a rexx in SYSEXEC allocation which executes a MVS command.    
           £ D M=CPU is the executed command. Confer line with CPC SI for model.
           tso c d m=cpu                                                        
           £                                                                    
           £ Other example of using the tso command:                            
           export _BPX_SHAREAS=YES                                              
           tso    time                                                          
           tso    "ex 'BOZO.EXEC(ABC)'"                                         
         ________________________________________________________               
         Other example:                                                         
           //* Copy from PDS RECFM=FB to PDSE RECFM=VB with replacement         
           // EXEC UNIXEDIT,DSNIN=BOZO.PDSFB,DSNOUT=BOZO.PDSEVB                 
           No control cards                                                     
                                                                                
   u                                                                            
        u is an edit macro executing a z/OS Unix command.                       
                                                                                
        Input to the z/OS Unix command is the                                   
        presently edited file.                                                  
        The result of the z/OS Unix command, if not empty,                      
        replaces the presently edited file,                                     
        unless a supplementary keyword 'info' is added .                        
        If 'info' is added to the Unix command (anywhere)                       
        the result of the Unix command is placed into infolines.                
                                                                                
        The user may cancel the result with:                                    
        Command ===> can                                                        
        or restore the previous content of a library member with:               
        Command ===> u r  (or u re, u res, u rest etc... or u u)                
        or save the result.                                                     
                                                                                
        If the word 'info', in mixed case, does not appear at right             
        of u, then edit macro u by default, saves the previous content          
        of a non empty edited library member in another member                  
        called 'SAVEU' before replacing this member                             
        by the result of an Unix command.                                       
                                                                                
        If keyword 'info', in mixed case, is placed anywhere after u,           
        then the result of an Unix command                                      
        does not replace the content but is added                               
        at top of member or sequential file, as infolines.                      
        These infolines may be deleted with RES or made datalines with          
        line commands MD and MDD.                                               
        The previous content is unchanged and saved in member 'SAVEU'.          
                                                                                
        Examples of use in View or Edit:                                        
                                                                                
        Command ===> u           (or u help)        <- shows help               
        Command ===> u u         (or u r or u re)   <- restore                  
        Command ===> u cut -c 1-10                  <- first 10 char            
        Command ===> u cut -c 1-10,12-              <- col 11 disappear         
        Command ===> u sed 's/A.*b/DEF/g'           <- case sensitive           
        Command ===> u iconv -f 1047 -t 1147        <- Unix to French           
        Command ===> u awk '{print $1}'             <- Keep 1st field           
        Command ===> u cal 2020                     <- all year                 
        Command ===> u cal sep 1752                 <- missing days             
        Command ===> u man sed                                                  
        Command ===> u man sed | cut -c 1-80                                    
        Command ===> zexpand                                                    
            u cat "//'BOZO.CNTL(HAYSTACK)'" | grep -E 'N.*dle'                  
        Command ===> u cut -c 1-10 info   <- infolines, no replacement          
        Command ===> u info cut -c 1-10   <- infolines, no replacement          
        Command ===> u man woman info     <- infolines, no replacement          
        Command ===> u ls -al oedit       <- oedit temporary result             
        Command ===> u oedit man awk      <- oedit temporary result             
        oedit is used for long output lines and u may be reissued.              
        Non empty temporary result replaces member or sequential file.          
        Command ===> can                  <- not to SAVE u's result             
        Command ===> u s (or u sa, u sav) <- explicit save in 'SAVEU'           
                                                                                
        edit macro u used to process z/OS Unix commands in member:              
                   BOZO.CNTL(A)                                                 
        Command ===> u sh                  or ===> u sh info                    
        ****** *********************                                            
        000001 whoami                                                           
        000002 pwd                                                              
        000003 printenv                                                         
        ****** *********************                                            
        Command ===> u cal jun ., cal jul        <- '.,' replaced by '; '       
        Command ===> u cd /u/bozo ,. cat abc.txt <- ',.' replaced by '; '       
                                                                                
   to                                                                           
        Edit macro, convert from codepage to codepage.                          
        Executes edit macro unix with iconv.                                    
        Output: Temporary file with converted code                              
                (may be copied on a permanent file).                            
        Examples: Command ===> TO IBM-1047 FROM IBM-1147                        
                  Command ===> TO               uses default                    
                  Command ===> TO LIST    list possible codepages               
                                                                                
   from                                                                         
        Edit macro, convert from codepage to codepage.                          
        Code is an exact copy of the code of the 'to' edit macro.               
        Examples: Command ===> FROM IBM-1047 TO IBM-1147                        
                  Command ===> FROM             uses default                    
                  Command ===> FROM LIST   list possible codepages              
                                                                                
   grepit                                                                       
        Rexx to search a member or a library with a regular expression.         
        It uses the following Unix command with bpxwunix:                       
        cmd = "  strings -z -n 8 | fold -w 72 | grep -Ei '"regex"'"             
        Examples:                                                               
        Command ===> tso grepit 'regex' dsname                                  
        Command ===> tso grepit 'regex' 'dsname'                                
        Command ===> tso grepit 'regex' dsname(member)                          
        Command ===> tso grepit 'regex' 'dsname(member)'                        
        Command ===> tso cmde                                                   
                     grepit 'regex' dsname                                      
        The regular expression regex is in local codepage                       
        (or is |) and must be between '                                         
        The dsname may or may not be surrounded by '                            
                                                                                
   strings                                                                      
        Execute the Unix commands strings and fold.                             
        Display printable strings folded to 72 columns.                         
        There is an optional parameter which indicates                          
        the minimum printable string length. It may be                          
        obtained from the Prompt field.                                         
                                                                                
        Example:                                                                
                 VIEW      XX.YYYY.USER01.LOADLIB                               
         Command ===>                                                           
                    Name     Prompt        Alias-of     Size                    
         _________ ABCLDCN0                           000019D8                  
         strings__ DEFGB790 1   <-- mimimum length  1 000035C0                  
         strings__ DEFGB80D     default min length 20 00007F10                  
                                                                                
   ux                                                                           
        Rexx, alternative to Command ===> epdf /                                
        Example:  Command ===> tso ux                                           
                                                                                
   man                                                                          
                                                                                
        rexx which execute man Unix command in MVS or z/OS Unix.                
        man ouput is translated to local ccsid then displayed                   
        with ISPF VIEW.                                                         
                                                                                
        rexx is placed in a MVS library concatenated to SYSPROC                 
        or SYSEXEC.                                                             
                                                                                
        Examples:                                                               
        Command ===> tso man Grep                                               
                                                                                
        Command ===> cmde                                                       
        Enter TSO commands below:                                               
        ===> man -k compare                                                     
                                                                                
        In z/OS Unix:                                                           
                                                                                
        Command ===> tso omvs                                                   
        $ tso man ls                                                            
              versus                                                            
        $ man ls                                                                
        $ exit                                                                  
                                                                                
   history                                                                      
                                                                                
        rexx which edit z/OS Unix user command history file                     
        from TSO or z/OS Unix.                                                  
                                                                                
        rexx is placed in a MVS library concatenated to SYSPROC                 
        or SYSEXEC.                                                             
                                                                                
        Examples:                                                               
        Command ===> tso history                                                
                                                                                
        In z/OS Unix:                                                           
                                                                                
        Command ===> tso omvs                                                   
        $ tso history                                                           
        $ exit                                                                  
                                                                                
                                                                                
   split                                                                        
        rexx which executes z/OS Unix split command                             
        on a MVS sequential file or a library member,                           
        creating a MVS library.                                                 
                                                                                
        Examples:                                                               
                 VIEW      BOZO.CNTL                                            
         Command ===>                                                           
                    Name     Prompt                                             
         split____ MEMBER01 100    <-- split every 100 lines                    
         split____ MEMBER02 -100   <-- split every 100 lines                    
         split____ MEMBER03 -l 100 <-- split every 100 lines                    
         split____ MEMBER04 500 Z  <-- split every 500 lines                    
                            and prefix created members with Z                   
                                                                                
         Created library:                                                       
         BOZO.CNTL.MEMBER01.SPLIT with members                                  
         XAA, XAB, XAC, ... Each one has 100 lines,                             
         if parameter in Prompt field, is 100.                                  
                                                                                
         split____ MEMBER05     default split every 1000 lines                  
         It start a new file every time it has copied                           
         1000 lines.                                                            
                                                                                
         ISPF 3.4                                                               
                  Data Sets Matching BOZO                                       
         Command ===>                                                           
                                                                                
         Command - Enter "/" to select action                                   
         ----------------------------------------------                         
                  BOZO.SEQ01                                                    
         split / 100 O.SEQ02                                                    
         Created library:                                                       
         BOZO.SEQ02.SPLIT with members                                          
         XAA, XAB, XAC, ... Each one has 100 lines.                             
                                                                                
         Command ===> tso split 'BOZO.CNTL(MEMBER1)' 100                        
         Command ===> cmde      <-- to preserve lower case                      
         Enter TSO commands below:                                              
         ===> split 'BOZO.CNTL(MEMBER2)' -l 100 MBR                             
          Created MVS library 'BOZO.CNTL.MEMBER2.SPLIT will                     
          contain membres MBRAA, MBRAB, ...                                     
                                                                                
                                                                                
   od                                                                           
         rexx                                                                   
         Execute od (octal dump) Unix command.                                  
         If no option provided, the rexx generates option                       
         for an hexadecimal dump (BSD option -Xhc).                             
                                                                                
         rexx is in SYSEXEC or SYSPROC concatenation,                           
         not in OMVS PATH.                                                      
                                                                                
         rexx can be called from either MVS or OMVS.                            
                                                                                
         Examples:                                                              
         Calling from MVS:                                                      
                  VIEW      BOZO.LOAD                                           
          Command ===>                                                          
                     Name     Prompt                                            
          od_______ MEMBER01        <-- hex dump                                
                                                                                
          ISPF 3.4                                                              
                   Data Sets Matching BOZO                                      
          Command ===>                                                          
                                                                                
          Command - Enter "/" to select action                                  
          ----------------------------------------------                        
          od______ BOZO.SEQ01       <-- hex dump                                
                                                                                
          Command ===> tso od 'BOZO.LOAD(MEMBER1)'                              
          Command ===> cmde         <-- to preserve lower case                  
          Enter TSO commands below:                                             
          ===> od 'BOZO.LOAD(MEMBER2)' -hc                                      
          ===> od /u/bozo/test.txt  <-- full path needed                        
                                                                                
         Calling from OMVS:                                                     
          Command ===> tso omvs                                                 
          $ tso od /u/bozo/test.txt                                             
          $ tso od BOZO.SEQ01                                                   
          $ exit                 (from OMVS)                                    
                                                                                
______________________________________________________________________________  
                                                                                
                                                                                
batch job GREPLIB Job executes grep with BPXBATCH on a HFS copy of a library.   
                                                                                
   GREP01  Job executes grep with BPXWUNIX in a procedure                       
                     on a HFS copy of a library                                 
                                                                                
   BPXWUNIX                                                                     
        JCL procedure.                                                          
        Execute z/OS Unix commands in batch                                     
        from CMD keyword then from //STDPARM file                               
                                                                                
        May replace BPXBATCH in a batch job.                                    
        If replacing a BPXBATCH batch job,                                      
           if there is a //STDPARM, replace only                                
              EXEC PGM=BPXBATCH          by                                     
              EXEC BPXWUNIX                                                     
              If ever parameter PARM if present, please suppress it             
              (it is ignored by BPXBATCH when //STDPARM is present)             
           if there no //STDPARM and there is a PARM, replace only              
                 EXEC PGM=BPXBATCH,PARM=    by                                  
                 EXEC BPXWUNIX,CMD=                                             
        Then run.                                                               
                                                                                
        Examples:                                                               
        //CUT     EXEC BPXWUNIX,CMD='cut -c 1-8,10-14',                         
        //             IN=&IN,OUT=&OUT                                          
                                                                                
        //         SET IN=BOZO.SEQ01.FB.LRECL80                                 
        //         SET OUT=BOZO.SEQ01.VB.LRECL84.BZ2                            
        //BZIP2    EXEC BPXWUNIX,CMD='bzip2'                                    
                                                                                
        //   EXEC BPXWUNIX,CMD='set -x; cal;'                                   
        //STDIN  DD DUMMY                                                       
        //STDOUT DD SYSOUT=*                                                    
        //CMD DD *                                                              
        pwd; cat "//'BOZO.SEQ01'" ; echo $A                                     
        //DDENV  DD *                                                           
        A="Bonjour"                                                             
                                                                                
   BZIP2                                                                        
        Batch job.                                                              
        Uses JCL procedure BPXWUNIX to compress a MVS file                      
        with bzip2.                                                             
        Once the file is transfered in binary on another z/OS,                  
        it might be decompressed with bunzip2.                                  
        7zip supports, on Windows and Linux, the .bz2 format.                   
                                                                                
   JAVA                                                                         
        Batch job.                                                              
        Uses JCL procedure BPXWUNIX to copy, compile and execute                
        a basic Java program.                                                   
                                                                                
______________________________________________________________________________  
                                                                                
rexx executing another rexx given as parameter                                  
                                                                                
   viewit                                                                       
        ISPF rexx, ISPF edit macro and z/OS Unix rexx.                          
        May be used to trace another rexx given as a parameter                  
        or to display the result of a command.                                  
        Examples: Used as a rexx:                                               
                      Command ===> tso viewit rexx01 parm1                      
                      instead of   tso rexx01 parm1                             
                      Command ===> TSO VIEWIT RLIST GCICSTRN *                  
                      Command ===> TSO VIEWIT LU                                
                      Command ===> tso viewit help lu                           
                      Command ===> TSO VIEWIT EX 'DSNAME(CLIST1)'               
                                                                                
                  Used as an edit macro:                                        
                    Execute present rexx (rexx being edited)                    
                    without parameters:                                         
                      Command ===> viewit                                       
                    Execute last saved rexx without parameters:                 
                      Command ===> viewit *                                     
                    Execute present rexx with parameters:                       
                      Command ===> viewit parm1 parm2                           
                    Execute last saved rexx with parameters:                    
                      Command ===> viewit * parm1 parm2                         
                                                                                
                  Used in a member list:                                        
                                                                                
                    Command ===>                                                
                               Name     Prompt                                  
                    viewit___ REXX01   PARM1                                    
                                                                                
                  Under z/OS Unix (rexx in SYSPROC or SYSEXEC                   
                  concatenation, vsualization with ISPF VIEW):                  
                    Command ===> tso omvs                                       
                    $ tso viewit time                                           
                    $ exit                                                      
                                                                                
   forallm                                                                      
        ISPF rexx. forallm means for all members.                               
        Command contains a member pattern with at least one '*' or '%'.         
        We execute the command for each member in pattern.                      
        Examples:                                                               
        Command ===> tso forallm lma XXXXXX.LOAD(A*)                            
                     We execute the rexx lma for each member                    
                     of library XXXXXX.LOAD whose name begins by A.             
        Command ===> tso viewit forallm lma XXXXXX.LOAD(A*)                     
                                                                                
   foralld                                                                      
        ISPF rexx. foralld means for all datasets.                              
        Command contains a level with at least one '*' or '%'.                  
        We execute the command for each dsname contained in level.              
        forallm can follow foralld.                                             
        Examples:                                                               
        Command ===> tso foralld whohas XX.YY%G.**.LOADA*                       
        Command ===> tso viewit foralld whohas XX.YY%G.**.LOADA*                
                                                                                
   mac                                                                          
        ISPF rexx. Rexx to invoke macro given as a parameter                    
        It uses three other rexx: viewit, forallm and macedit.                  
        Example of use: (ISPF 3.4, on same line as dsname)                      
         DSLIST - Data Sets Matching XX.YYYY.USER01                             
         Command ===>                                                           
         mac mac01 XX.YYYY.USER01.CNTL                                          
         mac mac01 /        <-- Execute macro mac01 on all members              
         mac mac01 /(*COMP*)                                                    
         mac mac01 /(COMPCOB)                                                   
                                                                                
   bg                                                                           
        Rexx.                                                                   
        bg for background.                                                      
        Example of a rexx which will call another rexx asynchronously.          
        We insert bg before the rexx name.                                      
                                                                                
        Example:                                                                
        Synchronous execution of REXX01 (foreground) would be:                  
        Command ===> tso REXX01 parm1 parm2 ...                                 
                                                                                
        Asynchronous execution of REXX01 (background):                          
        Command ===> tso bg REXX01 parm1 parm2 ...                              
        To preserve lowercase characters in parm1 parm2 ... :                   
        Command ===> cmde                                                       
        Enter TSO commands below:                                               
        ===> bg REXX01 parm1 parm2 ...                                          
                                                                                
   invoked                                                                      
        rexx requests another rexx to call it.                                  
                                                                                
______________________________________________________________________________  
                                                                                
Miscellaneous rexx:                                                             
                                                                                
   lma                                                                          
        Rexx executes Load Module Analyzer and AMBLIST                          
        on a load module/program object or on a small library.                  
        Example of use in ISPF 3.4:                                             
        LMA      XX.YYYY.USER01.LOAD                                            
        LMA      before dsname or member name                                   
                                                                                
   l2                                                                           
        Convert a pds or a pdse to a pdse version 2                             
        with the implied or specified maxgens parameter.                        
        Examples of use in ISPF 3.4:                                            
        //L2     USER01.LOAD1      <-- Group convert                            
                 USER01.LOAD2          to DSNTYPE(LIBRARY,2)                    
        //       USER01.LOAD3          with default MAXGENS                     
                 USER01.LOAD4                                                   
        L2       USER01.LOAD5      <-- Default convert                          
                 USER01.LOAD6                                                   
        L2 / 0   USER01.LOAD7      <-- Convert with MAXGENS=0                   
        L2 / 2   USER01.LOAD8      <-- Convert with MAXGENS=2                   
                                                                                
   v2                                                                           
        Convert a pds or a pdse to a pdse version 2,                            
        or a pdse version 1 or a pds (extension of rexx l2).                    
        Examples of use in ISPF 3.4:                                            
        Convert to DSNTYPE(LIBRARY,2):                                          
        V2       USER01.LOAD1      <-- Default MAXGENS                          
        //V2     USER01.LOAD2      <-- Group convert                            
                 USER01.LOAD3          with default MAXGENS                     
        //       USER01.LOAD4                                                   
        V2 / 0   USER01.LOAD5      <-- Convert with MAXGENS=0                   
        V2 / 2   USER01.LOAD6      <-- Convert with MAXGENS=2                   
        Convert to DSNTYPE(LIBRARY,1):                                          
        V2 / DSNTYPE(LIBRARY,1)                                                 
        Convert to PDS:                                                         
        V2 / DSNTYPE(PDS) DIR(500)                                              
        Examples of use on Command line:                                        
        Command ===> TSO V2 AB.CDEF.CNTL  <-- Default MAXGENS                   
        Command ===> TSO V2 AB.*.C%EF.**.*ABC* SIMULATE                         
        Command ===> TSO V2 AB.*.C%EF.**.*ABC*                                  
        Command ===> TSO V2 AB.CDEF.** 3  <-- MAXGENS=3                         
                                                                                
   ds                                                                           
        Rexx.                                                                   
        Alternative to DSLIST                                                   
                                                                                
        Syntax: Command ===> tso ds parameter(s)                                
        There could be zero, one or two parameters.                             
                                                                                
        1) With zero parameter:                                                 
        Dataset list using a default pattern (to change in the rexx)            
        Command ===> tso ds                                                     
                                                                                
        2) With one parameter, one of the two:                                  
        a) Dataset list using a specific pattern.                               
        pattern may have the % and * wild characters                            
        Examples:                                                               
        Command ===> tso ds bozo.**.*jcl*                                       
        Command ===> cmde     <-- to preserve lower case                        
        Enter TSO commands below:                                               
        ===> tso ds /u/boz*                                                     
        b) Permanent screen name for the present logical screen.                
        Example:                                                                
        Command ===> tso ds screen1                                             
                                                                                
        3) With two parameters:                                                 
        Dataset list using a specific pattern                                   
        in a new logical screen (no parameter order)                            
        Examples:                                                               
        Command ===> tso ds bozo.**.jcl screen1                                 
        or                                                                      
        Command ===> tso ds screen1 bozo.**.jcl                                 
        screen1 is the name given to the new logical screen.                    
        Command ===> tso ds bozo bozo                                           
        list all files with first qualifier bozo in a screen                    
        named bozo.                                                             
                                                                                
   sd                                                                           
        Rexx.                                                                   
        Show dsnames corresponding to a given pattern in ISRDDN format.         
        We allocate with bpxwdyn the datasets corresponding                     
        to the given pattern to ddnames having a common string                  
        (we chose $).                                                           
        We issue isrddn only $                                                  
                                                                                
        Call examples:                                                          
        Command ===> tso sd               <- this help and isrddn for prefix    
        Command ===> tso sd help          <- this help                          
        Command ===> tso sd boz%.cnt*     <- all datasets for boz%.cnt*         
        Command ===> tso sd **.*cntl*     <- all datasets with string cntl      
        Command ===> tso sd bozo          <- all datasets for alias bozo        
        Command ===> tso sd 'bozo.cntl(jcl01)'     <- bozo.cntl                 
        Command ===> tso sd **.*procli* m *cob*    <- COBOL procedures          
                                                                                
   dsm                                                                          
        Rexx.                                                                   
        Display a sorted member list with Command ===> sort cha                 
                                                                                
        Examples:                                                               
        Command ===> tso dsm           <- using content of variable             
                                          default_dsn in rexx.                  
                                          All members are sorted.               
        Command ===> tso dsm BOZO.CNTL <- All members of BOZO.CNTL              
        Command ===> tso dsm A%B* <- All members with pattern A%B*              
        Command ===> tso dsm BOZO.CNTL A%B*                                     
        Command ===> tso dsm A%B* BOZO.CNTL                                     
                                                                                
   o                                                                            
        Execute TEMPNAME member of own library.                                 
        TEMPNAME contains altlib act appl(exec) and libdef stack                
        to use own rexx and ispf libraries.                                     
        Example:                                                                
        Command ===> tso o
        
   c
        This rexx exec issues MVS or JES2 commands
        and may process Unix commands using as 'standard input'
        a sequential MVS file named:
        prefix.jobname.rexxname.queue
        It responds to the MVS reply given a character string
        present in the reply. 
        To obtain help on rexx C, please type:
        Command ===> tso c 
        This rexx extends a previous rexx exec called c which
        has been renamed into cold.      
                                                                                
   cold     (previous rexx c)                                                                             
        Issues MVS or JES2 command thru  address sdsf isfslash                  
        Example:                                                                
        Command ===> tso c d a         <-- MVS  command D A
        
   xdc
        This called rexx exec writes the content of a JES2 file
        on an output dataset.   
                                                                                
   cmd                                                                          
        Called module.                                                          
        May be used to edit macroless.                                          
        Maximum length of only parameter is 230 with case preserved.            
        Execute the parameter content.                                          
        Example 1: (Edition of a MVS sequential file or z/OS Unix file)         
          cmd = "epdf 'BOZO.SEQ'; c 'abc' 'DEF' all; save; end"                 
          call cmd(cmd)                                                         
        Example 2: (Edition of a z/OS Unix file)                                
          cmd = 'tso oedit test.txt; c mon my all; save; end'                   
          call cmd(cmd)                                                         
        Example 3: (Calling a rexx)                                             
          call cmd("tso time; tso rexx01 parm1 parm2")                          
                                                                                
          The rexx is appending ";end' to cmd with the intent                   
          to leave the auxiliary panel used to issue commands.                  
          There are several ways to remove the last 'end', to stay              
          for instance in edit on a file:                                       
          1) Add to cmd a large enough number of spaces which will              
             shift out the string ';end' added by default at the right          
             of cmd.                                                            
             E.g. cmd = cmd!!copies(' ',400)                                    
          2) Insert 'no end' or 'NO END' or 'noend' or 'NOEND'                  
             anywhere in cmd. The added string 'no end' or                      
             'NO END' or 'noend' or 'NOEND' will be removed                     
             and no string ';end' will be appended to cmd.                      
                                                                                
        Example 4: (Stay in edit)                                               
          cmd = 'tso oedit test.txt; c abc def all    no end'                   
          call cmd(cmd)                                                         
        Example 5: (View output file generated by c rexx,                       
                    c d a issue the 'd a' MVS command)                          
          cmd = "tso time; tso c d a;"copies(' ',400)                           
          call cmd(cmd)                                                         
        Example 6: (Access customized panel to issue commands)                  
          cmd = 'tso noend'                                                     
          call cmd(cmd)                                                         
                                                                                
   whohas                                                                       
        Excute D GRS,RES=(SYSDSN,dsn) with  address sdsf isfslash               
        Examples:                                                               
        Command ===> tso whohas USER01.LOAD                                     
        In ISPF 3.4, before dsname:                                             
        whohas   USER01.LOAD1                                                   
                                                                                
   parmli                                                                       
        Example of use of isrddn only to view and search through                
        concatenated libraries.                                                 
        Command ===> tso parmli                                                 
                                                                                
   parm                                                                         
        It shows PARMLIB datasets in ISRDDN format.                             
        It may be used to find duplicate members and with                       
        V then SRCHFOR, to search through the concatenation.                    
        Alternative to Command ===> ddlist;parmlib                              
        Example of passing a stem as a parameter to an                          
        inline procedure (stem_oedit).                                          
        Examples of call:                                                       
        Command ===> tso parm    /* 16  datasets per ddname     */              
        Command ===> tso parm 1  /* 1   dataset  per ddname     */              
        It exists: Command ===> tso parmlib                                     
                                                                                
   proclib                                                                      
        It shows PROCLIB datasets in ISRDDN format.                             
        It may be used to find duplicate members.                               
        The datasets are shown, by default, by group of 16 to permit            
        V before ddname then Command ===> SRCHFOR ...                           
        (search by group of 16 datasets).                                       
        Examples:                                                               
        Command ===> tso proclib       <-  16 datasets per ddname               
        Command ===> tso proclib 100   <- 100 datasets per ddname               
        Command ===> tso proclib 1     <- one dataset  per ddname               
                                                                                
   julie                                                                        
        To convert from Julian date tso standard date and reverse.              
        Example:                                                                
        Command ===> tso julie                                                  
                                                                                
   lmmstats                                                                     
        To update library member ISPF statistics, with LMMSTATS.                
        Examples:                                                               
        Command ===> cmde                                                       
        ===> lmmstats 'XX.YYYY.BOZO.CNTL(abcde)' created(66/12/01)              
        ===> viewit lmmstats 'XX.YYYY.BOZO.CNTL(abcde)' created(66/12/01)       
                                                                                
   id    To change the id in a library member list.                             
        Example: (Google "fish ascii" or "one line ascii art")                  
        Before:                                                                 
                VIEW      BOZO.CNTL                                             
        Command ===>                                                            
                   Name     Prompt       Size   Created    ...  ID              
        id_______ J        <°(((><          6  2018/03/15  ...  BOZO            
                                                                                
        After:                                                                  
                VIEW      BOZO.CNTL                                             
        Command ===>                                                            
                   Name     Prompt       Size   Created    ...  ID              
        _________ J                         6  2018/03/15  ...  <°(((><         
                                                                                
                                                                                
   abo                                                                          
        rexx which displays information related to                              
        ABO (Automatic Binary Optimizer) such as                                
        Architecture level, execute MVS, JES2, TSO commands                     
        commands and display in View a file with the results                    
        by calling itself.                                                      
                                                                                
        Examples:                                                               
        Command ===> tso abo       <-- information on Automatic Binary Optimizer
        Command ===> tso abo d a   <-- MVS  command D A                         
        Command ===> tso abo $di   <-- JES2 command $DI                         
        Command ===> tso abo tso time                                           
        Command ===> cmde                                                       
        Enter TSO commands below:                                               
        ===> abo tso rex01 Azerty Qwerty                                        
                                                                                
   codepage                                                                     
        rexx which gives the terminal codepage and                              
        the terminal charset while working under ISPF                           
        and other codepage information.                                         
                                                                                
        Example:                                                                
        Command ===> tso codepage                                               
                                                                                
   cp                                                                           
        rexx which shows, under ISPF, the terminal codepage                     
        at top right of screen.                                                 
                                                                                
        Example:                                                                
        Command ===> tso cp                                                     
        Result, top right of screen: 'Terminal codepage: 1140'                  
        Command ===> tso codepage    will give more information                 
                                                                                
   dup                                                                          
        Rexx.                                                                   
        Obtain duplicate member names for specified libraries                   
        using ISRDDN DUP                                                        
        Example:                                                                
        Command ===> tso dup                                                    
                                                                                
                                                                                
   relink                                                                       
        Rexx.                                                                   
        Relinks a load module/program object                                    
        Syntax: Command ===> tso relink dsn(member) parameter(s)                
                                                                                
        If zero parameter follows dsn(member) then show help                    
           and relink JCL with control statements examples to edit              
           in a file.                                                           
        If one parameter only follows dsn(member) then use binder statement     
           SETOPT PARM(parameter)                                               
           and call binder                                                      
           (in case parm is an 8 characters hex string,                         
            prefix hex string by 'SSI=')                                        
        else use parameters as a full binder statement                          
           and call binder.                                                     
                                                                                
        relink   parameter(s) may be obtained from prompt field                 
                 if at most eight characters long.                              
                                                                                
   Example 1:                                                                   
    Command ===> tso relink                                                     
    generates help and JCL file to edit.                                        
                                                                                
   Example 2:                                                                   
   XXXX    VIEW      BOZO.LOAD                                                  
    Command ===>                                                  Scroll ===>   
              Name     Prompt        Size       TTR      AC    AM  RM           
    relink   COBOL01                 00001398   000015   01    31  ANY          
    generates help and JCL file to edit. dsn and member indicated.              
                                                                                
   Example 3:                                                                   
    Command ===> tso relink 'BOZO.LOAD(COBOL01)' SSI=AA190206,REUS=RENT,AC=1    
    will use binder statement        SETOPT PARM(SSI=AA190206,REUS=RENT,AC=1)   
    and call binder.                                                            
                                                                                
   Example 4:                                                                   
    Command ===> cmde                                                           
    Enter TSO commands below:                                                   
    ===> relink 'BOZO.LOAD(COBOL01)' INCLUDE SYSLIB(SUB01)                      
    will use binder statement        INCLUDE SYSLIB(SUB01)                      
    and call binder.                                                            
    dsn is allocated to SYSLIB.                                                 
                                                                                
   Example 5:                                                                   
   XXXX    VIEW      BOZO.LOAD                                                  
    Command ===>                                                  Scroll ===>   
              Name     Prompt        Size       TTR      AC    AM  RM           
    relink   COBOL01  RMODE=24       00001398   000015   01    31  ANY          
                                                                                
   Result:                                                                      
   XXXX    VIEW      BOZO.LOAD                                                  
    Command ===>                                                  Scroll ===>   
              Name     Prompt        Size       TTR      AC    AM  RM           
    ________ COBOL01                 00001398   000015   01    31  24           
                                                                                
                                                                                
   scr                                                                          
        Rexx or edit macro.                                                     
        Syntax: Command ===> tso scr parameter                                  
        There could be zero or one parameter.                                   
                                                                                
        1) With zero parameter:                                                 
        Command ===> tso scr                                                    
        Flushes the panel cache                                                 
        (method from http://ibmmainframes.com/about59391.html                   
        Stefan, Germany)                                                        
                                                                                
        2) With one parameter:                                                  
        Flushes the panel cache and                                             
        set permanent screen name.                                              
        Example:                                                                
        Command ===> tso scr screen1                                            
        Flushes the panel cache and                                             
        set permanently present screen name to screen1                          
                                                                                
        In Edit or View mode:                                                   
        Command ===> scr                                                        
        Command ===> scr screen1                                                
                                                                                
   ssi                                                                          
        Rexx.                                                                   
        Set SSI eight hexadecimal digit constant to                             
        a load module/program object.                                           
        It rebinds the load module/program object setting the SSI.              
                                                                                
        It has two parameters:                                                  
        1) 'dsn(member)' for load module/program object                         
        2) optional SSI                                                         
        SSI      optional parameter, may be obtained from prompt field          
                 Eight hexadecimal characters: 0123456789ABCDEF                 
                 default:   2 hex char followed by yymmdd                       
                                                                                
   Example:                                                                     
    Command ===> tso ssi 'BOZO.LOAD(COBOL01)' AA190206                          
                                                                                
   Example:                                                                     
   XXXX    VIEW      BOZO.LOAD                                                  
    Command ===>                                                  Scroll ===>   
              Name     Prompt   Alias-of    ---- Attributes ----      SSI       
   ssi______ COBOL01   AA123456                     RN RU                       
   ssi______ COBOL02                                RN RU                       
                                                                                
   Result:                                                                      
   XXXX    VIEW      BOZO.LOAD                                                  
    Command ===>                                                  Scroll ===>   
              Name     Prompt   Alias-of    ---- Attributes ----      SSI       
   _________ COBOL01                                RN RU         AA123456      
   _________ COBOL02                                RN RU         FF190506      
                                                                                
                                                                                
   tallych                                                                      
        rexx procedure to count the number of times a character appears         
        in a string using space(string,0) (remove all spaces)                   
                                                                                
Batch job   WHERE  Batch job illustrating a search of a generic member          
        in generic libraries (source, load, etc...) with PDSEASY                
        and post processing.                                                    
                                                                                
   FTPUTF8                                                                      
        JCL model.                                                              
        Transfer a RECFM=FB library with FTP                                    
        from IBM-1140 to UTF-8 and reverse.                                     
        It uses the JCL procedure that I called LMCOPY.                         
                                                                                
   FTPUTF8S                                                                     
        JCL model.                                                              
        Transfer a RECFM=FB sequential file with FTP                            
        from IBM-1140 to UTF-8 and reverse.                                     
        It uses the JCL procedure that I called LMCOPY.                         
                                                                                
   FTP1252                                                                      
        JCL model.                                                              
        Transfer a library with FTP                                             
        from IBM-1140 to Windows-1252 and reverse.                              
                                                                                
   DELDEF                                                                       
        JCL procedure.                                                          
                                                                                
        Define sequential file or library (PDS or PDSE)                         
        (library definition with an ICEGENER dummy write).                      
                                                                                
        Examples of use:                                                        
        //         SET DSN=BOZO.SEQ01                                           
        //DELDEF1  EXEC DELDEF,DSN=&DSN      Sequential                         
        //         SET DSN=BOZO.PDSE01                                          
        //DELDEF2  EXEC DELDEF,DSN=&DSN      PDSE                               
        //DEF.SYSUT2 DD DSORG=PO,DSNTYPE=(LIBRARY,2),                           
        //       SPACE=(TRK,(5,50,100))                                         
                                                                                
                                                                                
   LMCOPY                                                                       
        JCL procedure.                                                          
        ISPF LMCOPY service in batch (corresponds to ISPF 3.3).                 
        Input and output files could have different RECFM and LRECL.            
                                                                                
        Example:                                                                
        // SET I=dsn1                                                           
        // SET O=dsn2                                                           
        // EXEC LMCOPY,I=&I,O=&O                                                
                                                                                
   SUBBAT                                                                       
        Job example.                                                            
        The job sends a Windows bat file hello.bat to a Windows server.         
        It will be converted in Windows 1252 aka IBM-1252, cp1252.              
        It will be executed by a periodically scheduled task                    
        (code at the end of the job) and hello.bat will return                  
        the result to z/OS.                                                     
                                                                                
   GUESSUTF                                                                     
        Called rexx routine.                                                    
        Try to guess whether a string given as a parameter is                   
        in codepage UTF-8 or not.                                               
                                                                                
        Example:                                                                
        call 'GUESSUTF' string                                                  
        answer = result /* 'utf8' or 'not-utf8' */                              
                                                                                
   HUMPTY                                                                       
        Rexx.                                                                   
        Extract data records from an non compressed,                            
        non encrypted, ADRDSSU DUMP of a PDSE with RECFM=FB.                    
        Member data records may be interspersed.                                
        First parameter is the DUMP dataset name.                               
        Second parameter is the PDSE fixed logical record length,               
        default value is 80.                                                    
                                                                                
        Examples:                                                               
        1) Command ===> tso humpty BOZO.PDS.DUMP                                
        2) Command ===> tso humpty BOZO.PDS.DUMP2 40                            
                                                                                
        3) In ISPF 3.4:                                                         
        Command - Enter "/" to select action                                    
        -----------------------------------------------------------             
        humpty    BOZO.PDS.DUMP     <-- humpty before dsname                    
        humpty / 40 7931.PDS.DUMP2                                              
                                                                                
   DUMPTY                                                                       
        Rexx.                                                                   
        Logically DUMP files with ADRDDSU using ALLDATA(*) ALLX                 
                                                                                
        Examples:                                                               
        1) Command ===> tso dumpty BOZO.PDS                                     
                                                                                
        2) In ISPF 3.4:                                                         
        Command - Enter "/" to select action                                    
        -----------------------------------------------------------             
        dumpty    BOZO.PDS          <-- dumpty before dsname                    
                                                                                
   LAZY                                                                         
        Rexx (using itself as an edit macro).                                   
        Display in terminal codepage                                            
         a string, given as a parameter, converted:                             
         to IBM-1047 (Unix),                                                    
         to IBM-1140 (US) if the terminal codepage is different,                
         to IBM-1208 (UTF8).                                                    
                                                                                
        Default string contains pangram:                                        
        THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG                             
                                                                                
        Examples of use:                                                        
         1) Command ===> tso lazy                                               
            displays special characters and above pangram.                      
                                                                                
         2) Command ===> tso lazy Bonjour []{}1                                
            displays a temporary file containing:                               
            Command ===>                                                        
            ****** ****************************                                 
            000001 mine 1147 Bonjour []{}1                                     
                   98984FFFF4C9999A949B55F94444                                 
                   49550114702651649005141F0000                                 
            -----------------------------------                                 
            000002 unix 1047 Bonjour Ý~éè1                                      
                   A98A4FFFF4C9999A94ABCDF34444                                 
                   459701047026516490DD001F0000                                 
            -----------------------------------                                 
            000003 US   1140 Bonjour ¬|éè1                                     
                   EE444FFFF4C9999A94BBCDF94444                                 
                   420001140026516490AB001F0000                                 
            -----------------------------------                                 
            000004 utf8 1208 â?>ù?ÍÊ $)£' SbÐ                                   
                   AA8F4FFFF44666677255773E8A22                                 
                   43680120802FEAF520BDBD122C00                                 
                                                                                
   T                                                                            
        Rexx.                                                                   
        Timer.                                                                  
        Display every minute a small box with the numbers of                    
        minutes elapsed and remaining. Terminal is locked.                      
         __ min __                                                              
        | 1    59 |                                                             
         ¯¯¯¯¯¯¯¯¯                                                              
                                                                                
        To halt, press Attention key two times then                             
        type 'hi' (halt interpretation waiting up to 3 seconds).                
                                                                                
        Examples of use:                                                        
         1) Command ===> tso t          <-- default one hour                    
         2) Command ===> tso t 7        <-- seven minutes                       
                                                                                
   GENERIC                                                                      
        Edit macro with invariant code,                                         
        parm (maximum 200 characters) and queue controlled.                     
        It obtains commands from parm and statements from stack                 
        and interpret them.                                                     
                                                                                
                                                                                
   REXX2MAC                                                                     
        Rexx.                                                                   
        Execute an edit macro on library members                                
                                                                                
        Example 1:                                                              
                VIEW      BOZO.PDS                                              
        Command ===> s *2* rexx2mac macro01 parm1                               
                   Name     Prompt                                              
        _________ MBR1                                                          
        _________ MBR2                                                          
        _________ MBR3                                                          
        will execute edit macro macro01 with parm parm1                         
        on all members with name containing 2.                                  
        Members may be modified by macro.                                       
        parm1 will be in uppercase due to panel zcmd attribute.                 
                                                                                
        Example 2:                                                              
        Command ===> cmde                                                       
        Enter TSO commands below:                                               
        ===> rexx2mac 'bozo.pds(mbr2)' macro01 parm1                            
                                                                                
                                                                                
   REXXMAC                                                                      
        Example of                                                              
        an edit macro in rexx's clothing.                                       
        Purpose:                                                                
                VIEW      BOZO.PDS                                              
        Command ===> s *2* rexx01                                               
                   Name     Prompt                                              
        _________ MBR1                                                          
        _________ MBR2                                                          
        _________ MBR3                                                          
        will execute rexx rexx01 on all members with name                       
        containing 2.                                                           
        If instead of a rexx, I would like an edit macro                        
        to process all members with name containing 2,                          
        I should make the edit macro to look like a rexx.                       
        This is the case with rexxmac.                                          
        Command ===> s *2* rexxmac                                              
                                                                                
   RETP                                                                         
        Called rexx.                                                            
        Get previous commands in a string.                                      
        Commands are separated by ' ; '                                         
                                                                                
        Example of use:                                                         
                                                                                
        In calling program:                                                     
        str = retp()  obtains last command in a string.                         
        str = retp(1) obtains the last command (same as retp() ).               
        str = retp(n) obtains the n-th most recent command                      
                      (until the 25th most recent command).                     
                                                                                
        str = retp('all') gives the previous commands                           
        in a string.                                                            
        Commands are separated by ' ; '                                         
        Commands are the commands with at least one non space                   
        character as would be given by                                          
        Command ===> retp                                                       
        in the same order                                                       
        and their leading and trailing spaces have been stripped.               
        Example of content of str after   str = retp('all')  :                  
        str = "tso time ; save ; cut   aa"                                      
        "tso time" is the last command.                                         
                                                                                
                                                                                
   LASTCMD                                                                      
        rexx.                                                                   
        Get last command or previous commands.                                  
                                                                                
        Used as a called rexx:                                                  
        In calling program:                                                     
        cmd = lastcmd()  obtains last command.                                  
        cmd = lastcmd(1) obtains last command (same as previous)                
        cmd = lastcmd(2) obtains second most recent command                     
        cmd = lastcmd(n) obtains the n-th command                               
                      starting from the most recent command,                    
                      cycling thru the command stack as                         
                      Command ==> retrieve                                      
                                                                                
        On command line:                                                        
        Command ===> tso lastcmd     <-- last command                           
        Command ===> tso lastcmd 2   <-- next to last command                   
        or cycle to last command if command stack contains only                 
        one command.                                                            
                                                                                
        Example of use:                                                         
        When displaying a member list                                           
                  VIEW      BOZO.PDS                                            
          Command ===> s * rexx01 AA BB                                         
                     Name     Prompt                                            
          _________ MBR1                                                        
          _________ MBR2                                                        
          rexx01 executes on all members of the library.                        
          When it executes MBR1, it receives as only parameter                  
          'BOZO.PDS(MBR1)' and has no knowledge of AA BB.                       
          The last command in the command stack is:                             
          s * rexx01 AA BB                                                      
          AA BB is obtained by rexx01 with subword(lastcmd(),4).                
          AA may possibly be an edit macro using parameter BB.                  
                                                                                
                                                                                
   VU                                                                           
        rexx or edit macro.                                                     
                                                                                
        Function is according to the member name:                               
        vu:    View UTF-8 file converted to EBCDIC terminal codepage.           
               Edit UTF-8 file if we add a last parameter 'e' or 'E'            
               to either the rexx or the edit macro.                            
        vuhex: View UTF-8 in hex (unchanged hex code)                           
        eu:    Edit UTF-8 file displayed in EBCDIC terminal codepage            
        euhex: Edit UTF-8 in hex                                                
        Same code in the four rexx.                                             
                                                                                
        If the name of the rexx does not begin by 'E'                           
        and in the case there are parameters, the last one is                   
        not 'e' or 'E', then                                                    
        View a MVS library member or a MVS sequential data set                  
        or a z/OS Unix file                                                     
        (member or sequential data set or z/OS Unix file                        
         already in UTF-8 is converted to terminal codepage)                    
        otherwise use Edit instead of View.                                     
                                                                                
        vu: UTF-8 to EBCDIC terminal codepage conversion by                     
            ISPF service View or Edit.                                          
        May be used with a dbrmlib member.                                      
                                                                                
        Examples of use for VU:                                                 
                                                                                
        1) As a rexx, in a member list:                                         
                VIEW      BOZO.DBRMLIB                                          
        Command ===>                                                            
                   Name     Prompt       Size                                   
        vu_______ BOZO01              <--       View                            
        vu_______ BOZO02    e         <-- force Edit (e in Prompt)              
                                                                                
        2) As a rexx on a command line:                                         
        Command ==> tso vu BOZO.DBRMLIB(BOZO01)       <--       View            
        Command ==> tso vu BOZO.DBRMLIB(BOZO01) e     <-- force Edit            
                                                                                
        3) As a rexx preserving lower case:                                     
        Command ==> cmde                                                        
        Enter TSO commands below:                                               
        ===> vu /u/bozo/bozo01.utf8                    <--      View            
        ===> vu /u/bozo/bozo01.utf8 e              <-- e forces Edit            
        ===> vu /u/bozo/bozo01.utf8 macro(a) parm(b) e <--      Edit            
        ===> vu 'bozo.cntl(inutf8)' macro(a) parm(b)   <--      View            
        ===> vu 'bozo.cntl(inutf8)' macro(a) parm(b) noutf8 e                   
        It is possible to add View or Edit extra parameters                     
           and the special parameter noutf8 to remove                           
           the default parameter utf8.                                          
                                                                                
        4) As an edit macro:                                                    
                   BOZO.DBRMLIB(BOZO01)                                         
        Command ===> vu                         (vu e to force Edit)            
        000001 DBRM   `BOZO    BOZO01   Ð H ì/   B                              
        000002                                                                  
        000003 DBRM   ]       Ï       £@áä< êá ä        äíêë!ê ã!ê              
                                                                                
                                                                                
   SAV                                                                          
        Edit macro which may be used instead of the                             
        'SAVE' Edit command.                                                    
                                                                                
        Apart of the normal SAVE, it saves the library member in                
        a member named, by default, SAVED                                       
        Examples of use, in ISPF EDIT:                                          
        Command ===> sav        <-- normal save and save in member SAVED        
        Command ===> sav SAVED2 <-- normal save and save in member SAVED2       
                                                                                
        The edit macro has a parameter,                                         
        set by default to "yes", save_in_waste_basket.                          
        If the library being edited has fixed record format and                 
        LRECL=80, the member is also saved in a library                         
        prefix.WASTE.BASKET                                                     
        This library is created if it does not exist.                           
                                                                                
   FLAT                                                                         
        FLAT is a rexx and edit macro.                                          
                                                                                
        It sequentializes a library using successively IEBPTPCH and             
        SORT.                                                                   
        The library should have a fixed record format                           
        (RECFM=F or RECFM=FB or RECFM=FBA) and LRECL=80.                        
                                                                                
        The output is in the form of input to the IEBUPDTE utility              
        (which permits to reload a library or to update it).                    
        ISPF statistics are lost.                                               
        FLAT produces, apart of the sequential file, another file               
        containing a JCL sample to reload the first file.                       
        Examples of use:                                                        
                                                                                
        Command ===> tso flat                       <- shows help               
        Command ===> tso flat help                                              
                                                                                
        Command ===> tso flat bozo.cntl                                         
        Command ===> tso flat 'bozo.cntl(mbr1)'     <- same as above            
                                                       mbr1 ignored             
        Created or replaced files:                                              
        BOZO.CNTL.FLAT    and                                                   
        BOZO.CNTL.FLAT.JCL                                                      
                                                                                
                                                                                
        In ISPF 3.4:                                                            
                 Data Sets Matching BOZO                                        
        Command ===>                                                            
                                                                                
        Command - Enter "/" to select action                                    
        ------------------------------------                                    
                 BOZO.CNTL                                                      
        flat     BOZO.CNTL2            <- flat alone.                           
                                                                                
        Created or replaced files:                                              
        BOZO.CNTL2.FLAT    and                                                  
        BOZO.CNTL2.FLAT.JCL                                                     
                                                                                
        In Edit or View:                                                        
                   BOZO.CNTL(MBR1)                                              
        Command ===> flat bozo.source  <- edit macro                            
        000001 //JOB1 JOB ...             It sequentializes the full            
        000002 //STEP01 ...               library BOZO.SOURCE. No edit.         
                                                                                
                   BOZO.CNTL(MBR1)                                              
        Command ===> flat              <- flat alone.                           
        000001 //JOB1 JOB ...             It sequentializes the full            
        000002 //STEP01 ...               library BOZO.CNTL.                    
                                                                                
   UNFLAT                                                                       
        UNFLAT is a rexx.                                                       
                                                                                
        It copies a sequential input file in IEBUPDTE input format to           
        a library with replacement.                                             
        If the output library does not exist, it is created.                    
                                                                                
        There are two parameters:                                               
        1) The input dsn which is a sequential file (or a library member)       
           which has the form of the input file to the IEBUPDTE utility         
           with ./ ADD NAME  and possibly ./ ENDUP control cards.               
           E.g.:                                                                
                                                                                
           ./ADD NAME=MBR1                                                      
           ABC                                                                  
           DEF                                                                  
           ./  ADD  NAME=MBR2,LIST=ALL                                          
           UVW                                                                  
           XYZ                                                                  
           ./ENDUP                                                              
                                                                                
        2) The output dataset (a library).                                      
           If it does not exists, it is created.                                
           If the second parameter is not specified, we create                  
           an output library with name, the name of the input                   
           sequential dataset, to which we add '.UNFLAT'                        
           if a dataset with this name does not exist.                          
           E.g. If the input dataset is 'BOZOO.SEQ', we create                  
           'BOEO.SEQ.UNFLAT' if such dataset does not exist,                    
           otherwise, we exit.                                                  
                                                                                
           The output library, if its name is 'BOZO.LIB',                       
           will contain the members:                                            
           'BOZO.LIB(MBR1)' and                                                 
           'BOZO.LIB(MBR2)'.                                                    
                                                                                
        Examples of use:                                                        
        Command ===> tso unflat BOZO.SEQ                                        
         The output library has its name obtained from the input                
         sequential dataset by adding '.UNFLAT'.                                
         The output library BOZO.SEQ.UNFLAT will be created or updated.         
         If it already exist, its members will be replaced.                     
        Command ===> tso unflat BOZO.SEQ.FLAT BOZO.LIB.UNFLAT                   
         The output library BOZO.LIB.UNFLAT will be created or updated.         
                                                                                
        Method: Split with z/OS Unix utility awk.                               
                                                                                
   SHOWHELP                                                                     
        Subroutine to incorporate in another rexx.                              
        It extracts help lines from source and display it.                      
                                                                                
        This subroutine is incorporated in the rexx which contain               
        the first group of lines to display.                                    
        We assume that the group of lines to display in the calling             
        rexx has its first line containing the string 'Help starts'             
        and its last line containing the string 'Help ends'                     
        The subroutine is placed after the group of lines to display.           
        It is called by: call showhelp and it does not return to caller.        
        Example:                                                                
        /* rexx                                */                               
        /* Help starts                         */                               
        /* This is an example of calling rexx. */                               
        /* Help ends                           */                               
        call showhelp                                                           
        exit                                                                    
        showhelp:                                                               
        ...                                                                     
        return                                                                  
                                                                                
   DDN                                                                          
        Display ddnames and associated dsnames                                  
        in ISRDDN format                                                        
                                                                                
        Input: JCL     (library member or sequential file with JCL) or          
               JES2JCL (under SDSF after issuing SE).                           
        Edit mmacro or rexx.                                                    
        Examples of use:                                                        
                                                                                
        As an edit macro with JES2JCL SE:                                       
                                                                                
             JOB DATA SET DISPLAY                                               
        COMMAND INPUT ===>                                                      
        NP   ££££ DDNAME   StepName                                             
                1 JESMSGLG JES2                                                 
        se      2 JESJCL   JES2                                                 
                                                                                
                     BOZO01   (JOB12345)                                        
        Command ===> ddn                                                        
        ****** *************************                                        
        000001         1 //BOZO     JOB                                         
                                                                                
        Command ===> ddn m *EMBE*              <- member search                 
        Command ===> ddn right                 <- dsorg, lrecl                  
                                                                                
        As an edit macro in a library member containing JCL:                    
        Command ===> ddn                                                        
        Command ===> ddn m *EMBE*              <- member search                 
        Command ===> ddn right                                                  
                   BOZO.CNTL(JCL01)           - 01.0                            
        Command ===> ddn                                                        
        ****** ********************************* Top                            
        000001 //BOZO    JOB '<°(((><',CLASS=D,...                              
        000002 //ASM    EXEC PGM=ASMA90,PARM=NODECK                             
                                                                                
        In ISPF 3.4 as a line command (rexx):                                   
        ddn / cou                              <- count (default)               
        ddn / dup                              <- duplicates                    
        ddn / right                            <- dsorg, lrecl                  
                                                                                
        In a member list as a line command (rexx):                              
        ddn  JCL01  right                                                       
                                                                                
        On command line as a rexx:                                              
        Command ===> tso ddn                   <- this help                     
        Command ===> tso ddn help              <- this help                     
        Command ===> tso ddn 'bozo.cntl(jcl01)' right                           
        Command ===> tso ddn  bozo.cntl(jcl01) <- quotes optional               
                                                                                
   HEREDOC                                                                      
        Inline procedure which is added to an existing rexx.                    
        When heredoc is called, it picks up the content                         
        of the next rexx comment following its call and                         
        returns it and, on request, makes the content of                        
        the comment a stem or puts it in a new stack.                           
                                                                                
        Examples:                                                               
                 string = heredoc()                                             
                 /* line 1  /* sub-comment 1 */                                 
                    line 2 */                                                   
        string contains: 'line 1 line 2'                                        
                                                                                
                 string = heredoc(2)                                            
                 /* line 1  /* sub-comment 1 */                                 
                    line 2 */                                                   
        string contains: 'line 1 /* sub-comment 1 */ line 2'                    
                                                                                
        interpret heredoc() /* say "Hello" */                                   
                                                                                
        cmd = heredoc() /* cat "//'BOZO.DOG'" */                                
        cmd will contain: cat "//'BOZO.DOG'"                                    
                                                                                
        call heredoc "stack",2                                                  
        call bpxwunix '/bin/submit','STACK',out.,err.                           
        "delstack"                                                              
        /*                                                                      
        //BOZO JOB ,CLASS=D,MSGCLASS=S,NOTIFY=&SYSUID,USER=&SYSUID              
        //* JCL comment and rexx sub-comment */                                 
        //IEFBR14 EXEC PGM=IEFBR14,PARM='<°(((><'                               
        ///                                                                     
        */                                                                      
                                                                                
   U2E (rexx E2U has same code as U2E)                                          
        Rexx and edit macro.                                                    
        Synonimous: UTOE and ETOU. All have the same code.                      
                                                                                
        Conversion from terminal codepage to UTF-8                              
        if string '2U' or 'TOU'                                                 
        is included in the name of the rexx.                                    
        Conversion from UTF-8 to terminal codepage                              
        if string '2E' or 'TOE'                                                 
        is included in the name of the rexx.                                    
                                                                                
        e2u: from EBCDIC (terminal codepage) to UTF-8                           
        u2e: from UTF-8 to EBCDIC (terminal codepage)                           
                                                                                
        Conversion is made with EDIT command REPLACE with                       
        conversion parameter either utf8 or ebcdic.                             
                                                                                
        We copy the edited file to a temporary file                             
        using REPLACE with an explicit conversion parameter.                    
        We edit the temporary file.                                             
        The user may use REPLACE or CREATE or CUT                               
        edit commands to save the temporary file.                               
        The temporary file is removed after edition.                            
                                                                                
        Examples of use for e2u                                                 
        (same examples  for u2e):                                               
                                                                                
        1) As a rexx, in a member list:                                         
                VIEW      BOZO.CNTL                                             
        Command ===>                                                            
                   Name     Prompt       Size                                   
        e2u______ BOZO01                                                        
                                                                                
        2) As a rexx on a command line:                                         
        Command ==> tso e2u BOZO.CNTL(BOZO01)                                   
                                                                                
        3) As a rexx preserving lower case:                                     
        Command ==> cmde                                                        
        Enter TSO commands below:                                               
        ===> e2u /u/bozo/bozo01                                                 
        ===> e2u 'bozo.cntl(bozo01)' macro(a) parm(b)                           
                                                                                
        4) As an edit macro:                                                    
                   BOZO.CNTL(BOZO01)                                            
                                                                                
                                                                                
   GLASSES                                                                      
        Rexx and edit macro.                                                    
                                                                                
        We called the rexx glasses as if each codepage was a                    
        different pair of glasses seeing the same data                          
        (the same hexadecimal code) in a different way.                         
        In another way:                                                         
        How a user, in another country, will see my data?                       
                                                                                
        The rexx has one parameter: cp (for codepage).                          
        Examples of values for cp: 1147, FR, 1047, UX, 1140, US.                
        It displays the Windows .bat or .cmd code to                            
        extract from z/OS the present file                                      
        and to display on Windows its content                                   
        as seen on z/OS by a user with terminal codepage cp.                    
                                                                                
        The generated code may be cut and pasted on Windows.                    
                                                                                
        glasses FR to show on Windows how your file                             
        is seen by a z/OS user in France.                                       
                                                                                
        glasses UX to show on Windows how your file                             
        is seen by z/OS Unix.                                                   
        We use the z/OS Unix IBM-1047 code page.                                
                                                                                
        glasses ME generates a script to save your file                         
        using your terminal codepage, in UTF-8.                                 
        It is the default.                                                      
                                                                                
        Examples of use for glasses                                             
                                                                                
        1) As a rexx, in a member list,                                         
           the parameter is in the Prompt field.                                
           The question is: How a French user will see my file?                 
                VIEW      BOZO.CNTL                                             
        Command ===>                                                            
                   Name     Prompt       Size                                   
        glasses__ BOZO01    FR                                                  
                                                                                
        2) As a rexx on a command line:                                         
        Command ==> tso glasses BOZO.CNTL(BOZO01) FR                            
                                                                                
        3) As a rexx preserving lower case:                                     
        Command ==> cmde                                                        
        Enter TSO commands below:                                               
        ===> glasses /u/bozo/bozo01 fr                                          
                                                                                
        4) As an edit macro:                                                    
                   BOZO.CNTL(BOZO01)                                            
        Command ===> glasses                                                    
        Command ===> glasses UX                                                 
        Command ===> glasses 1047                                               
                                                                                
                                                                                
   ARCH                                                                         
        Rexx.                                                                   
        Show Model and Architecture level at top right of screen.               
        Example of use:                                                         
        Command ===> tso arch                                                   
        Result, top right of screen: 'Architecture: ARCH(13)'                   
                                                                                
   XMIT

        This PROC submits a XMIT job which submits a RECEIVE job
        if the return code of its XMIT step is zero.
        It transmits sequential files or libraries.

        Example of use:
        //XMIT   EXEC XMIT,FROM=MVSA,TO=MVSB,
        //            FROMDSN=&SYSUID..CNTL,
        //            TODSN=&SYSUID..CNTL2,
        //            CNTL=MVSC  SDSF where to check
        Transmit a library named &SYSUID..CNTL from JES2 node MVSA
        to JES2 node MVSB. Target library is named  &SYSUID..CNTL2.
        If &SYSUID..CNTL2 does not exist, it is created. 
        If it exists, there is copy with replacement.
        All job outputs (XMIT ans RECEIVE) may be checked under SDSF
        at node MVSC. The names of the XMIT and RECEIVE jobs are
        &sysuid.X and &sysuid.R.
        MVA, MVSB and MVSC may be indentical.                                                                             

  DB                                                                      
        Rexx and edit macro.                                                    
                                                                                
        It executes DB2 DSN subcommands or DB2 SQL statements.
        SQL statements are executed with DSNTEP2. 
        For help and examples:
        Command ===> tso db 
                   
        Example as an edit macro:
        VIEW       ABC.CNTL(SQL01)
        Command ===> db           <- edit macro, also: db ssid(db2x)
        ****** ***************************** Top of Data
        000001        select 'ABC' from sysibm.sysdummy1;
        000002        select 3*4
        000003                 from sysibm.sysdummy1;

        On command line:
        Command ===> tso db select 'ABC' from sysibm.sysdummy1
        Command ===> tso db 'ABC.CNTL(SQL01)'  
        
        In a rexx:
        address mvs "execio" queued() "diskw "ddname" (fini"
        "db dd:"ddname
