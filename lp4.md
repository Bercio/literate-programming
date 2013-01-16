# Literate Programming

"Its like writing sphagetti code then shredding the code into little pieces, throwing those pieces into a blender, and finally painting the paste onto an essay. Tasty!"

This is the fourth cycle of literate programming. Here, we augment substitution lines to play a more active role in the processing of the parts. We also add in switch typing/naming within a block. 


## How to write a literate program

Use markdown. Each heading is a new block and can be referenced by using _"title". This substitution has more features, documented below. 

Within each block, one can write directives and/or initiate a type change in the coding blocks by starting a line with capital letters. 

For example, to say that a file should be saved from the block's text, one can write FILE filename type
where type states which named internal block to start with; it is optional if there is only one internal code block. 

In addition, a jump of two levels or more in the heading yields heading directives that pertain to just the parent block. All caps inside a code block is a constant and/or macro. 


### Runnable Code

One should also be able to run JavaScript code directly in the interpreter. I think a decent convention would be underscore backtick code  backtick.        

    _`javascript code`

The eval output (last evaluated value) is what is placed in the code to replace the backtick call.

To eval long blocks of code (the above is limited to a single line), use the pipe syntax:  _"Some block||eval()"  

We use two pipes as the second part should be a type/name of codeblock if needed, e.g.,  _"Some block|js|eval()"  

Within the parenthetical can be an expression that should be evaluated to start the block. 

### Full pipe syntax

The substitution expression should be of the form "block name|internalname.ext|command(arg1, arg2)|commmand2(...)|..."

Examples:  _"Great|jack|marked(1)",  _"Great|md",  _"Great||", _"Great|jack.md",  "|jack"  will load the internal block named jack


### Multi-level substitutions

There may be need to run substitutions after a first pass or more. For example, compiling a jade template into html and then running the substitutions that put in the text of the template. 

The number of underscores gives which loop number the substitution happens. In other words, for each loop iteration, one underscore is removed until only one is left at which point the substitution is made. 

Example:

    #example
        _"nav jade"
        #text
             __"markdown text"
    

## Structure

The program is structured into a module file and a command line file. 

The command line file's structure is in section "Cli" while the module file, which is the heart of the code and designed to work-in browser eventually, is in "LP module"


## The lp module

This module takes in a literate program as a string of markdown and possibly some options (not used yet). 

It takes the string and makes a document that has the markdown parsed out into various blocks. The parsing is down line-by-line. The two main calls are `lineparser` which parses the lines, creating the document,  and the `makeFiles` command which will compile the blocks in the `files` array to a full string that can then be saved or whatever (that's what the command line does). 


JS  

    /*global require, module*/
    /*jslint evil:true*/

    var beautify = require('js-beautify').js_beautify;
    var jshint = require('jshint').JSHINT;
    var marked = require('marked');



    _"Utility Trim"
    _"Raw String"
    _"Apply Trim"
    
    module.exports.compile = function (md, options) {

        var Block, Doc; 
        
        _"Document constructor"

        _"Block constructor"

        var lineparser = _"Parse lines";      

        Doc.prototype.lineparser = lineparser;
        Doc.prototype.makeFiles = _"Make files";

        var doc = lineparser(md, options);
        
        doc.makeFiles();
 
        return doc;
    };



FILE lib/literate-programming.js 


## Document parsing

Each literate program gets its own document style. It starts off life with lineparser. 

Each line is of one of four basic types:

1. Header. This signifies a new block. If it jumps in level by two or more levels, then it is considered an internal block instead of a new independent block. Thus, it is attached to the parent and is not seen from the outside. Think tests and examples. 
2. Code line. This is a line indented with 4 spaces (maybe a tab?) If that is true, it gets put into the code block of the current type. 
3. Directive/Type switch. If a line starts with all caps or a ., then it has the potential to be a directive (such as FILE command)  or a to switch the code block to a new type or name. What happens is very dependent on what is found there.  
4. Plain text. This just is for reading and gets put into a plain text block without it being useful, presumably. 

All lines in a block do get put into storage in order for safe-keeping (some raw output, for example, could be useful).

### Parse lines

This is the function that takes in a literate program, splits it into lines, and parses them, returning a structure for compilation. 

The Document consists mostly of blocks constructed by the Block constructor. The doc.processors is where the magic happens. 

JS

    function (lp, options) {
      var i, line, nn; 
      var doc = new Doc(options); 

      var lines = md.split("\n");
      doc.cur.level = 0; 
      var n = lines.length;
      for (i = 0; i < n; i += 1) {
        line = lines[i];
        nn = doc.processors.length;
        for (var ii = 0; ii < nn; ii += 1) {
            if (doc.processors[ii](line, doc) ) {
                doc.cur.full.push(line);
                break;
            }
        }
      }
      return doc;
    }

Each processor, corresponding to the 4 types mentioned above, will check to see if the line matches its type. If so, they do their default action, return true, the line is stored in the full block for posterity, and the other processors are skipped. 


### Default processors

The processors array, a property of the Document, allows us to change the behavior of the parser based on directives. They should return true if processing is done for the line. The argument is always the current line and the doc structure. 

JS

    [ 
    _"Code parser", 
    _"Head parser", 
    _"Directives parser", 
    _"Plain parser" 
    ]



### Code parser


We look for 4 spaces of indentation. If so, it is code and we store it in the current block's code block for the current type. 

Note that tabs do not trigger a code block. This allows for the use of tabs for multiple paragraphs in a list. The tab then 4 indents does trigger code. That's where the insanity stops. 

JS

    function (line, doc) {
      var cur = doc.cur;
      var reg = /^(?: {4}|(?:\t {4}))(.*)$/;
      var match = reg.exec(line);
      if (match) {
        cur.code[cur.type].push(match[1]);
        return true;

      _"|Add empty line"
        
      } else {
        return false;
      }
    }

JS Add empty line

Added the following clause to add empty lines to the code. Stuff before and after the code block is probably trimmed, but in between extra lines could be added. This was to enable blank lines being there which is important for markdown and other markup languages. 

      } else if (line.match(/^\s*$/)  ) {
        var carr = cur.code[cur.type];
        if (carr && carr.length > 0 && carr[carr.length -1 ] !== "") {
            cur.code[cur.type].push(line);
        }
        return false; // so that it can be added to the plain parser as well



## Directives parser

We will implement more directives. Directives will be recognized as, at the start of a line, as all caps and a matching word. This may be conflict, but seems unlikely. A space in front would defeat the regex. Periods are also allowed in constants. At least two capital letters are required.

    function (line, doc) {
      var fileext = /^\.([A-Z]+)(?:$|\s+(.*)$)/;
      var match = fileext.exec(line);
      if (match) {
        doc.switchType(match[1], match[2]); 
        return true;
      }
      var reg = /^([A-Z][A-Z\.]*[A-Z])(?:$|\s+(.*)$)/;
      match = reg.exec(line);
      if (match) {
        if (doc.directives.hasOwnProperty(match[1])) {
            doc.directives[match[1]](match[2], doc);
            return true;
        } else if (doc.types.hasOwnProperty(match[1].toLowerCase()) ){
            doc.switchType(match[1], match[2]);
            return true;   
        } else {
            return false;
        }
      } else {
        return false;
      }
    }

The function takes in a line and the doc structure. It either returns true if a successful directive match/execution occurs or it returns false. The directives object is an object of functions whose keys are the directive names and whose arguments are the rest of the line (if anything) and the doc object that contains the processors and current block structure. 

### Switch type 

Here is the function for switching the type of code block one is parsing. The syntax is the type (alread parsed and passed in) and then name of the block followed by pipes for the different functions to act on it. If there is no name, then use a pipe anyway. Examples:  `JS for running | run  ` or  `JS | hint`

Anything works for the name.

Will need to make a more refined options parser at some point. 

    function (type, options) {
        var doc = this;
        var cur = doc.cur;

        type = type.toLowerCase(); 
        if (typeof options === "undefined") {
            options = "";
        }
        options = options.split("|");
        var name = options.shift();
        if (name) {
            name.trim();
            cur.type = name.toLowerCase()+"."+type;
        } else {
            cur.type = "."+type;
        }

        if (! cur.code.hasOwnProperty(cur.type) ) {
            cur.code[cur.type] = doc.makeCode();
        }

And now we work on get the options to parse. The syntax is an optional number to indicate when to process (0 for pre, 1+ for during, nothing for post), followed by whatever until parentheses, followed by optional arguments separated by commas. Examples: `0 marked (great, file)` and `marked` and `marked awesome(great)`

        
        var funmatch, funreg = /^(\d*)\s*([^(]+)(?:\(([^)]*)\))?$/;
        var i, n = options.length, option;
        for (i = 0; i < n; i += 1) {
            option = options[i].trim();
            funmatch = option.match(funreg);
            if (funmatch === null ) {
                doc.log("Failed parsing (" + name +" ): " + option);
                continue;
            }
            if (funmatch[1] === "0") {
                //add to pre
            } else if (funmatch[1] ) {
                // add to during
            } else {
                //add to post
            }
        }
    /*        
        while ( (options) && ( (match = options.match(funmatch) ) !== null) ) {

            funname = match[1].toUpperCase();
            if (doc.directives.hasOwnProperty(funname) ) {
                doc.directives[funname](match[2],  doc);
                //.call( {doc:doc, cur:cur, type:type}, match[2].split(",").trim())
               
            } else {
                doc.log.push("No such directive: " + funname + " " + type + " "+ options);
            }
            options = match[3];
        }
    */

    }

## Subsection Headings


Subsection headings refer to the higher level previous code block. One can write "#### Test"   (cap not matters) to write a test section for "## Great code".  This allows for the display of the markdown to have these sections hidden by default. 
Some possibilities as above, but also a doc type for writing user documentation. The literate programming is the developer's documentation, but how to use whatever can be written from within as well. 

By going two extra levels, we can recognize levels for testing and so forth without polluting the global namespace. Two levels could be those, such as examples, that should be for general review while three levels could be for edge case test cases, hidden by default. 

## Chunk headings

Parse out the section headings. These constitute "#+" followed by a name. Grab what follows. Parse code blocks indicated by 4 spaces in. One block per section reported, but can be broken up. Stitched together in sequence. 

To do this programmatically, we will split the whole text by newlines. At each line, analyze whether there is a "#" starting it--if so, get text after for name of section. At each line, check if there are four spaces. If so, add the line of code to the block. 

Each subheading is nested in the one above. So we need structures that hold the block and heading hierarchy.


 





### Head parser

We recognize a heading by the start of a line having '#'. We count the number of sharps, let's call it level. The old level is oldLevel.

If level <= oldLevel +1, then we have a new subsection of code.

If level >= oldLevel +2, then this is a Heading Directive and we try to match it with a directive. It feeds directly into the oldLevel section and is not globally visible. 

For new global blocks, we use the heading string as the block name. We lower case the whole name to avoid capitalization issues (it was really annoying!)

    function (line, doc) {
      var level, oldLevel, cur, name, cname;
      var head = /^(\#+)\s*(.+)$/;
      var match = head.exec(line);
      if (match) {
        name = match[2].trim().toLowerCase();
        oldLevel = doc.level || 0;
        level = match[1].length;

        _"Remove empty code blocks"

        cur = new Block();
        cur.name = name;
        cur.type = doc.type;    
        cur.code[cur.type] = doc.makeCode();


        // this shortcircuits if it is a directive heading
        _"Directive heading" 
                
        doc.blocks[name] = cur; 
        doc.cur = cur; 
        doc.level = level;
        doc.name = name;
        // new processors for each section
        doc.processors = [].concat(doc.defaultProcessors);
        
        return true;
      } 
      return false;
    }

In the above, we are defining the default processors again fresh. This prevents any kind of manipulations from leaking from one section to another. It could be a performance penalty, but probably not a big deal. Garbage collection should remove old processors. 

#### Remove empty code blocks

    cur = doc.cur; 
    for (cname in cur.code) {
        if (cur.code[cname].length === 0) {
            delete cur.code[cname];
        }
    }


This suffered from having empty lines put into the code block. Solution: do not add empty lines unless there is a non-empty line of code before it.

#### Directive heading

Here we want to change the current block to take the current part, but it needs to be added to the parent's subdire

This will hide all blocks that have +2 level change or higher. Need to implment automatic running of subdire blocks. Might change this to seek a match for a directive heading. Or could use it to hide blocks; so look for substitution 

        if (level >= oldLevel +2) {
            level = oldLevel; 
            cur.parent = doc.cur.parent|| doc.cur;

            doc.cur = cur; 
            cur.parent.subdire.push(cur);
            return true;
        }



### Plain parser

This is a default. It means there is nothing special about the line. So we simply add it to the current full block.

    function (line, doc) {
      doc.cur.plain.push(line);
      return true;
    }

### Merge in options

In order to have more custom behavior, such as linking in behavior for visual editing, we want to be able to pass in options to the functions. 

We have just created the doc object. Now we take it and merge it in with the options object. 

    if (options) {
        var key;
        for (key in options) {
            this[key] = options[key];
        }
    }


## Constructors 

### Document constructor

    Doc = function (options) {
        this.blocks = {};
        this.cur = new Block();
        this.files = [];
        this.log = [];
        this.subtimes = 0;
        this.type = ".";

        this.types = {js: 1, md: 1, html: 1, css: 1}; 

        this.directives = _"Directives";

        this.commander = _"Doc commander";

        this.commands = _"Default doc commands";

        this.constants = {};


        this.processors = [].concat(this.defaultProcessors);
      

        _"Merge in options"

        return this;
    };

    Doc.prototype.maxsub = 1e5;

    Doc.prototype.oneSub = _"Line analysis";
    Doc.prototype.fullSub = _"The full substitution";

    Doc.prototype.defaultProcessors = _"Default processors";

    Doc.prototype.switchType = _"Switch type";

    Doc.prototype.makeCode = _"Make code block";


    Doc.prototype.pre = function (code) { //, block, doc) {
        return code.trim();
    };

    Doc.prototype.post = function (code) { //, block) { //, doc) {
        return code;
    };

    Doc.prototype.during = function (code){  //, block, doc, counter) {
        return code;
    };

#### Doc commander

We need to write a parser for the commands argument. It takes in something between the pipes as well as the code, block. 

    function (command, code, block) {
        var doc = this;
        var reg = /([A-Za-z]+)\s*\( ([^)]*)\)/;
        var match = reg.exec(command);
        var args;
        command = match[1];
        if (typeof doc[command] === "function" ) {
            args = match[2].split(",").trim();
            doc.commands[command].apply({code: code, block:block, doc:doc}, args);
        } else {
            return code; 
        }   
    } 

#### Default doc commands


    {"eval" : function () {
            return eval(this.code);
        }
    }

#### Make code block

We need an array with some extra methods.

    function () {
        var doc = this;
        var ret = [];
        ret.pre = doc.pre;
        ret.post = doc.post;
        ret.during = doc.during;
        return ret;
    }


### Block constructor

    Block = function () {

        this.code = {};
        this.full = [];
        this.plain = [];
        this.subdire = [];

        return this;
    };



## Compiling the document 

## Line analysis

So we have three basic things that we might see in code that the compiler needs to do something about:

1. Substitutions
2. Constants
3. Evaling

This is a method of a doc; it takes in a string for the code and the current block. It returns a string if replacements happened or false if none happened.

We run through the code string, matching block name/js code block/constant. We note the location, get the replacement text, and continue on. We also keep a look out for multi-level, preparing to reduce the level. 

Once all matches are found, we replace the text in the code block. 

We return 1 if there was a replacement, 0 if not.

For evaling, no substitutions are done. It is just straight, one line code. If evaling a block is needed use _"block to run|eval"


    function oneSub (codeBlocks, name, block) {
        
        var doc = this;

        _"Max sub limit"
        

        var code = codeBlocks[name];


        var reg = /(?:(\_+)(?:(?:\"([^"]+)\")|(?:\`([^`]+)\`))|([A-Z][A-Z.]*[A-Z]))/g;
        var rep = [];
        var match, ret, type, pieces, where, comp, ctype, ext;

        var blocks = doc.blocks;

        while ( (match = reg.exec(code) ) !== null ) {
            
            //multi-level 
        
            if (match[2]) {
                
                //split off the piping
                pieces = match[2].split("|").trim();
                where = pieces.shift().toLowerCase(); 

                if (where) {
                    if (doc.blocks.hasOwnProperty(where) ){
                        _"Matching block, multi-level"
                        comp = doc.fullSub(blocks[where]);
                    } else {
                        // no block to substitute; ignore
                        continue;
                    }
                } else {
                    // use the code already compiled in codeBlocks
                    _"Matching block, multi-level"
                    comp = codeBlocks;
                }
                    
                _"Substitute parsing"

                rep.push([match[0], ret]);
                //console.log(ret);
                               
            } else if (match[3]) {
                // code
                _"Matching block, multi-level"
                
                rep.push([match[0], eval(match[3])]);

            } else {
                // constant
                if (doc.constants.hasOwnProperty(match[4])) {
                  rep.push([match[0], doc.constants[match[4]]]);
                }
            }

        }
        //do the replacements or return false
        if (rep.length > 0) {
            for (var i = 0; i < rep.length; i += 1) {
                code = code.replace(rep[i][0], rep[i][1].rawString());
            }

            codeBlocks[name] = code; 

            return 1; 
            
        } else {
            return 0;
        }
    }
        
We need the function fullSub which runs through the code repeatedly until all substitutions are made. 

### Matching block, multi-level

There is a match, but the level is not yet ready  for full substitution

                    if (match[1] && match[1].length > 1) {
                        rep.push([match[0], match[0].slice(1)]);
                        continue;
                    }

### Max sub limit

We need to regard against infinite recursion of substituting. We do this by having a maximum loop limit. 

        if (doc.subtimes >= doc.maxsub) {
            console.log("maxed out", block.name);
            return false;
        } else {
            doc.subtimes += 1;
        }

### Substitute parsing


Either the substitution specifies the bit to insert or we use the current name's type to pull an unnamed bit from the same text. If nothing, we continue. 

The bit between the first pipe and second pipe (if any) should be the type and type only. 

 
    type = pieces.shift();

    ext = name.split(".")[1].trim().toLowerCase();

Possibilities for matching type:
1. type is the full name and a good match. Safest _"|jack.js"
2. type is the extension. So we add "." and see if that works _"|js". Would not match jack.js
3. type is the name, but with extension. Use the default extension of the type or just a dot,  _"|jack"  becomes jack.js if looked at in cool.js block. Also checked is jack.
4. If all that fails, then loop through the keys trying to match text. Unpredictable.


    if (type) {
        ret = comp[type] || 
            comp["."+type] || 
            comp[type+"."+ext] || comp[type+"."];
        if (!ret) {
            for (ctype in comp) {
                if (ctype.match(type) ) {
                    ret = comp[ctype];
                    break;
                }
            }
        }
    } else {
        // try the extension from this type or try the no extension
       ret = comp["."+ext] || comp["."];
    }

    // grab something!
    if (!ret) {
        for (ctype in comp) {
            ret = comp[ctype];
        }
    }

    while (pieces.length >0) {
       ret =  doc.commands(pieces.shift(), ret, block); 
    }




### The full substitution 

This compiles a block to its fullly substituted values.

JS

    function fullSub (block) {
        var doc = this;
        var name ; 
        var  code={}, blockCode;


Check if already compiled. If so, return that.

        if (block.hasOwnProperty("compiled") ) {
            return block.compiled;
        } else {
            block.compiled = {};
        }

Cycle through the named code blocks, doing pre compiles

        for (name in block.code) {

            blockCode  = block.code[name];
            code[name] = blockCode.join("\n");
            code[name] = blockCode.pre(code[name], block, doc);
        
        }

Loop through all the names in each loop over substitution runs. This allows for subbing in at just the right moment. By using another code block with its own substitution, probably just about anything is possible. I think. Need to think up an example. 


            var counter = 0, go=1;
            while (go) {
                go = 0;
                counter += 1;
                for (name in code) {
                    go += doc.oneSub(code, name, block);
                    code[name] = block.code[name].during(code[name], block, doc, counter);
                }
            }
        
Loop through the names one more time to get any post compile directives. 


        for (name in code) {
            code[name] = block.code[name].post(code[name], block, doc); 
            block.compiled[name] = code[name]; 
        }


        return block.compiled;
    }

There are three hooks for processing: pre, post, and during. The during object takes in an extra parameter that states which phase the process is in. 


###### Example

awe is already joned and so it can be fed into marked immediately and consumed.

cool gets markedup after pre and then the eq is substituted in and in the second round it all is done. 

long takes three loops to compile. in the first loop, it is marked. in the second loop, long gets equation subbed in. in the third loop, it is installed. 

so the piping can process post compiled while the command switch can compile pre and during. 

HTML snip

    <p class="awesome">_"|awe|marked"/p>
    <p> class="lesscool">__"|cool"</p>
    <p> class="long">___"|long"</p>

MD awe 

    great work **dude**

MD cool marked(0)

    totally rad _"|eq"

MATH eq 

    \[\sin(x^2)*\int_5^8 \]

MD long marked(1)

    nearly done __"|eq"




So we may need delayed substitutions as well here.

HTML snip

    <p>Continuing the previous snip</p>

###### Example

HTML

    <p class="cool">Need some cool color</p>

CSS

    p.cool {color:blue}

HTML jack

    <p>not called</p>

CSS jack
    p {color:red}

HTML

    <p>Also in original block.</p>

This should be a good format. Let's say it is in a section called cool. Then we can get it by _"cool"; this will pull in the HTML in an html block while it will pull in CSS in a CSS block. Any other type will pull in nothing.  But we could add in _"cool|.css"  to pull in the css block. or _"cool|jack" to get jack. If multiple jacks, then  _"cool|jack.css". Space after pipe optional. So no pipes in name.  If we had a mark down section to convert to html, then we could do _"cool|.md|marked"  Any number of processors are possible. 



## Recommendations

A lot of the literate programming for JavaScript might involve creating functions. One could just write it as 

    function name (parameters) {
      code
    }

in some code block. Let's say it is in section highfun

And then later on we could import them into some other control structure by using

    var fun = _"highfun";

or one could write

    var funobj = {
      fun : _"highfun",
    };

Note that we do want the heading to be inlined for this though one could write it on separate lines. 







### File instead

Could have directives on the line so FILE name  jshint  jstidy   or FILE name jsmin or this can also be a place to run the tests and not save if they fail or save to some other place. 

## Plugin functionality

Here we map out how to dynamically load directives, constants, and heading directives. We also establish automatically loaded files that can be suppressed by command line options. 


## Directives


Create the object that holds the directives. It will contain object names for any directives that need to be instituted. Each of the values should be a function that takes in the material on the line post-command and the doc which gives access to the current block and name. 


    { 
        "FILE" : _"File directive",
        "RAW" : _"Raw directive",

Below should be split off


        "JS.TIDY" : _"JS tidy",
        "JS.HINT" : _"JS hint",
        "MD.HTML" : _"Marked",
        "MARKED" : _"Marked",
        "JS.PRE" : _"JS Pre"
    }


### Core directvies


#### File directive
     
The command is `FILE fname.ext` where fname.ext is the filename and extension to use. 

    function (options, doc) {
        options = options.trim();
        var match = options.match(/^(\S+)\s*(.*)$/);
        if (match) {
            doc.files.push([match[1], doc.name, match[2]]);
            doc.cur.file = match[1];        
        } else {
            doc.log("No file name for file: "+options+","+ doc.name);
        }
    }

#### Raw directive

Ignore the code bit and give back the raw full block. Mainly useful for markdown documents. 

To do this, we take out the code that would have been parsed in the pre mode. It loops once, doing nothing. Then in the post, we return the full portion. 

    function (options, doc) {
       doc.cur.pre = function () {
            return ""; 
       };

       doc.cur.post = function (code, block) {
           return block.plain.join("\n");
       };
    }


#### Load directive

This is to load other literate programs. It can compile them and save the files in the root directory (no arguments on LOAD) or it can start in a section and return the compiled version as the return value of that block. If multiple LOADs used in the same section, they get concatenated together. The Loading takes place immediately if no section, otherwise it starts in the compile phase. 


    function (options, doc) {
        options = options.split(","); 
        if (options.length > 1) {

            var pre = doc.cur.post;
            doc.cur.pre = function (code, block, doc) {
                code = pre(code, block, doc);
                _"Parse and load file"

               var section = options[1].trim();
               var code += exdoc.fullSub(section, exdoc); 

               return code;    
            };
        } else {
            _"Parse and load file";

            exdoc.

        }
    };


##### Parse and load file


        var fname = option[0].trim();
        if (fname) {
            if (doc.hasOwnProperty("external") ) {
                if (doc.external.hasOwnPropety(fname) ) {

                }
            } else {
                doc.external = {};
            }

do some parsing and loading

            doc.external[fname] = exdoc;
            //hackery!
            exdoc.fullSub = doc.fullSub;
            exdoc.oneSub = doc.oneSub;



#### Require directive

This is to load up various directives, constants, etc. It can be either a literate program with optional entry point(s) or a js file that compiles to an object whose keys can then be a list that adds. The object would have "whatever key" : [ {type: {object} } ]; the value can be a singleton instead of an array.

#### Version directive

Version control directive for the literate program. Generally at the base of the first intro block. This would be useful for setting up a npm-like setup. 

### Default directives

These are directives that are generally used, but whose loading can be turned off with a command line flag. They are hosted in a separate file. 

They often involve processing the compiled text and probably use various other modules. 




### Prototype of Directive

It runs immediately upon encountering. One probably wants to do some options parsing and then have something that works on doc.cur.pre/during/post 

    function (options, doc) {
        //post/pre/during
       var post = doc.cur.post;
       doc.cur.post = function (code, block, doc) {
           code = post(code, block, doc);
           code = //something awesome happens here
           return code;
       };

        // no return
    }


### JS tidy

Run the compiled code through JSBeautify 

    function (options, doc) {
       var post = doc.cur.post;
       doc.cur.post = function (code, block, doc) {
           code = post(code, block, doc);
           return beautify(code, options ||{ indent_size: 2, "jslint_happy": true } );
       };
    }
   
Needs js-beautify installed: `npm install js-beautify`

### JS hint

Run the compiled code through JSHint and output the results to console.

    function (options, doc) {
       var post = doc.cur.post;
       doc.cur.post = function (code, block, doc) {
           code = post(code, block, doc);
           jshint(code, options ||{ } );
           var data = jshint.data();
           block.jshint = {data:data, errors: [], implieds :[], unused :[]};
           var lines = code.split("\n");
           var log = [], err, i;
           for (i = 0; i < jshint.errors.length; i += 1) {
               err = jshint.errors[i];
               log.push("E "+ err.line+","+err.character+": "+err.reason +
                "  "+ lines[err.line-1]);
                block.jshint.errors.push({"line#": err.line, character: err.character, reason: err.reason, line: lines[err.line-1]} );
           }
           if (data.hasOwnProperty("implieds") ) {
             for (i = 0; i < data.implieds.length; i += 1) {
                 err = data.implieds[i];
                 log.push("Implied Gobal "+ err.line+": "+err.name +
                "  "+ lines[err.line[0]-1]);
                  block.jshint.implieds.push({"line#": err.line, name:err.name, line: lines[err.line[0]-1]} );

             }            
           }
          if (data.hasOwnProperty("unused") ) {
             for (i = 0; i < data.unused.length; i += 1) {
                 err = data.unused[i];
                 log.push("Unused "+ err.line+": "+err.name +
                "  "+ lines[err.line-1]);
                block.jshint.unused.push({"line#": err.line, name:err.name, line: lines[err.line-1]} );

             }            
           }


           if (log.length > 0 ) {
             log = ("!! JSHint:" + block.file+"\n"+log.join("\n"));
           } else {
             log = ("JSHint CLEAN: " + block.file);
           }

           doc.log.push(log);
           return code;
       };
    }

Needs jshint installed: `npm install jshint`   

### Marked

Run the text through the marked script to get html. Need to escape out underscored substitutions. 
    
    function (options, doc) {

        var lpsnip = [], mathsnip = [];

        var masklit = function (match) {
            lpsnip.push(match);
            return "LITPROSNIP"+(lpsnip.length -1);
        };

        var maskmath = function (match) {
            mathsnip.push(match);
            return "MATHSNIP"+(mathsnip.length-1);
        };

        var unmasklit = function (match, number) {
            return lpsnip[parseInt(number, 10)];
        };

        var unmaskmath = function (match, number) {
            return mathsnip[parseInt(number, 10)];
        };

        var modify = function (code, block, doc, options) {
            code = code.replace(/\_+(\"[^"]+\"|\`[^`]+\`)/g, masklit); 
            code = code.replace(/\$\$[^$]+\$\$|\$[^$\n]+\$|\\\(((?:[^\\]|\\(?!\)))+)\\\)|\\\[((?:[^\\]|\\(?!\]))+)\\\]/g, maskmath);
            code = marked(code);
            if (options.length > 0) {
                var elem = options[0];
                options = options.slice(1);
                _"Create attribute list"
                code = "<" + elem + " "+attributes+">"+code+"</"+elem+">";
            }
            code = code.replace(/LITPROSNIP(\d+)/g, unmasklit);
            code = code.replace(/MATHSNIP(\d+)/g, unmaskmath);
            return code;
        };

        _"Options on when"

    }

Needs marked installed: `npm install marked`   

### JS Pre

Encapsulate code into a Pre code element. Escape characters if necessary. Need to test. 

    function (options, doc) {

        var modify = function (code, block, doc, options) {
            _"Create attribute list"
            _"Escape code"
            return "<pre " + attributes + "><code>"+code+"</code></pre>";
        };

        _"Options on when"

    }  

#### Escape code

Replace <> with their equivalents. 

    code = code.replace(/</g, "&lt;");
    code = code.replace(/>/g, "&gt;");
    code = code.replace(/\&/g, "&amp;");

#### Unescape code

And to undo the escapes: 

    code = code.replace(/\&lt\;/g, "<");
    code = code.replace(/\&gt\;/g, ">");
    code = code.replace(/\&amp\;/g, "&");


### Options on when

When to process the text can be dependent. A common idiom will be to have a number as the first option which will say on which entry. It should be either -1 (pre), 0 (post), or a positive integer for during. If it does not fit that pattern, then a default of 0 for post is assumed. We assume there is a function in the environment named modify is assumed to work on the code and returns the new code text. 

    options = options.split(",") || [];

    if (options.length === 0) {
        options.push(0);
    } 

    var tempnum, tempmod; 

    if ( (tempnum = parseInt(options[0], 10) ) == options[0] ) {
        if (tempnum  === -1) {
            tempmod = doc.cur.pre;
            doc.cur.pre = function (code, block, doc) {
                code = tempmod(code, block, doc);
                code = modify(code, block, doc, options.slice(1));
                return code;
            };
        } else if (tempnum > 0) {
            tempmod = doc.cur.during;
            doc.cur.during = function (code, block, doc, counter) {
                code = tempmod(code, block, doc, counter);
                if (counter === tempnum) {
                    code = modify(code, block, doc, options.slice(1));
                }
                return code;
            };
        } else {
            tempmod = doc.cur.post;
            doc.cur.post = function (code, block, doc) {
                code = tempmod(code, block, doc);
                code = modify(code, block, doc, options.slice(1));
                return code;
            };    
        }
    }

### Create attribute list

We want to create an attribute list for html elements. The convention is that everything that does not have an equals sign is a class name. So we will string them together and throw them into the class, making sure each is a single word. The others we throw in as is. The id of the element is the block name though it can be overwritten with id="whatever". Options is an array that has the attributes.

    var i, option, attributes = "", klass = [], temp, 
        id = block.name.replace(/\s/g, "_"); // id may not contain spaces
    for (i = 0; i < options.length; i += 1) {
        option = options[i];
        if ( option.indexOf("=") !== -1 ) {
            // attribute found, check if id
            temp = option.split(/\s+/);
            if (temp[0] === "id" && temp[1] === "=") {
                id = temp[2];
            } else {
                attributes += option;
            }
        } else { // class
            klass.push(option.trim());
        }
    }
    attributes = "id='"+id+"' " + "class='"+klass.join(" ")+"' "+ attributes;



## Make files

We now want to assemble all the code. This function takes in a parsed lp doc and uses the files array to know which files to make and how to weave them together. 

So the plan is to go through each item in files. Call the full substitute method on it and then save it. 

    function () {
        var doc = this;
        var compiled = doc.compiled = {};
        var files = doc.files;
        var fname, blockname, text, ext, type, ret, ctype, comp;
        for (var i = 0; i < files.length; i  += 1) {
            fname = files[i][0];
            blockname = files[i][1];
            comp = doc.fullSub(doc.blocks[blockname]);
            _"Get correct text block"

            compiled[fname] = [ret, blockname];
        }
    }

The pre and post functions are hooks for running sections of code at appropriate times. They are implemented using directives.


### Get correct text block

We need to get the right block of text. First we check if there is a request from the file directive. If not, then see if we can get the extension.

Probably should refactor this to be a function as it is used elsewhere as well.

type needs to be rethought out. need some piping.

    type = files[i][2];
    ext = fname.split(".");
    ext = ext[ext.length -1].trim();
    if (type) {
        ret = comp[type] || 
            comp["."+type] || 
            comp[type+"."+ext] || comp[type+"."];
        if (!ret) {
            for (ctype in comp) {
                if (ctype.match(type) ) {
                    ret = comp[ctype];
                    break;
                }
            }
        }
    } else {
        // try the extension from this type or try the no extension
       ret = comp[fname] || comp["."+ext] || comp["."];
    }



### Utility Trim

This was lifted from JavaScript the Definitive Guide: 

    // Define the ES5 String.trim() method if one does not already exist.
    // This method returns a string with whitespace removed from the start and end.
    String.prototype.trim = String.prototype.trim || function() {
       if (!this) return this;                // Don't alter the empty string
       return this.replace(/^\s+|\s+$/g, ""); // Regular expression magic
    };

### Apply Trim 

Trims all objects with a trim function in an array and returns a copy. Mainly used here for post-split cleanup.

    Array.prototype.trim = Array.prototype.trim || function () {
        var ret = [];
        var arr = this;
        var i, n = arr.length;
        for (i = 0; i < n; i += 1) {
            if (typeof arr[i].trim === "function") {
                ret[i] = arr[i].trim();
            }
        }
        return ret; 
    };

### Raw String

This code is for dealing with using replace with arbitrary strings. The dollar sign has special meaning in replacement strings. Since $$ is used for implementing math mode, its use in a replacement string leads to problems. But the return value from a function in a replacement string does not have this problem. Thus, I augmented the String type to have a method, rawString which produces a function that returns the string. 

    String.prototype.rawString = String.prototype.rawString || function () {
        var ret = this;
        return function () {return ret;};
    };



## Cli 

This is the command line file. It loads the literate programming document, sends it to the module to get a doc object, and then sends the file component to the save command. An optional second command-line 

    #!/usr/bin/env node

    /*global process, require, console*/
    var program = require('commander');
    var fs = require('fs');
    var lp = require('../lib/literate-programming');

    _"Command line options"


    var save = _"Save files";
 
_"Load file"

    var doc = lp.compile(md);
    save(doc.compiled, dir); 


    _"Cli log"

FILE bin/literate-programming.js
JS.HINT


### Load file

Get the filename from the command line arguments. It should be third item in [proccess.argv](http://nodejs.org/api/process.html#process_process_argv).  

No need to worry about async here so we use the sync version of [readFile](http://nodejs.org/api/fs.html#fs_fs_readfilesync_filename_encoding).

    var filename = process.argv[2];
    if (!filename) {
        console.log("Usage: litpro file-to-compile optional:directory-to-place-result");
        process.exit();
    }
    var dir = process.argv[3];
    var md = fs.readFileSync(filename, 'utf8');


### Save files
    
Given array of name and text, save the file. dir will change the directory where to place it. This should be the root directory of all the files. Use the filenames to do different directories. 

    function (files, dir) {
        if (dir) {
            process.chdir(dir);
        }
        for (var name in files) {
            // name, text
          console.log(name + " saved");
          fs.writeFileSync(name, files[name][0], 'utf8');
        }

    }


### Cli log

This is where we report the logs. 

    console.log(doc.log.join("\n\n"));

### Command line options

Here we define what the various configuration options are. 


    program
        .version('0.1')
        .usage('[options] <file>')
        .option('-d --dir <root>', 'Root directory')
    ;

    program.parse(process.argv);

    if ((! program.args[0]) ) {
        console.log("Need a file");
        process.exit();
    }

    var dir = program.dir; 
    var md = fs.readFileSync(program.args[0], 'utf8');

#### Command arguments doc

    Currently there is only one flag: -d or --dir  with a directory that specifies the root directory where the compiled files go. 



## References

I always have to look up the RegEx stuff. Here I created regexs and used their [exec](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/RegExp/exec) method to get the chunks of interest. 

[MDN RegExp page](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/RegExp)


## README

MD

    literate-programming
    ====================
    
    Write a program using markdown to write out your thoughts and the bits of code that go with those thoughts. This program weaves the bits together into usable fiels. 

    ## Installation

    This is not yet operational, but soon: 

        npm install -g literate-programming

    ## Using

    The command installed is literate-programming and it has some command flags and one primary argument which is the markdown document containing the program, or at least the structure of the project. 

        literate-programming sample.md 

    will create the files specified in sample.md

    ### Command flags
    _"Command arguments doc"


    ## Document syntax

    A literate program is a markdown document with some special conventions. 

    The basic idea is that each header line (regardless of level) demarcates a full block. Any code blocks within a full block are concatenated together and are the code portion of the block. 

    Each code block can contain whatever kind of code, but there are three special syntaxes (space between underscore and quote should not be present; it is there to avoid processing): 

    1. _ "Block name" This tells the compiler to compile the block with "Block name" and then replace the _ "Block name" with that code.
    2. _ `javascript code`  One can execute arbitrary javascript code within the backticks, but the parser limits what can be in there to one line. This can be circumvented by having a block name substitution inside the backticks. 
    3. CONSTANTS/MACROS all caps are for constants or macro functions that insert their output in place of the caps. 

    For both 1 and 3, if there is no match, then the text is unchanged. One can have more than one underscore for 1 and 2; this delays the substitution until another loop. It allows for the mixing of various markup languages and different processing points in the life cycle of compilation.

    Outside of a code block, if a line starts with all caps, this is potentially a directive. For example, the `FILE` directive takes the name of a file and it will compile the current block and save it to a file. 

    If a heading level jumps down by two or more levels (say level 2 going to level 4), then this is also a potential directive. It allows for the use of a TEST section, for example, that can automatically run some tests on a compiled block.

    ## Nifty parts of writing literate programming

    You can write code in the currently live document that has no effect, put in ideas in the future, etc. Only those on a compile path will be seen. 

    You can have your code in any order you wish. 

    You can "paste" multiple blocks of code using the same block name. 

    You can put distracting data checks/sanitation/transformations into another block and focus on the algorithm without the use of functions (which can be distracting). 

    You can use JavaScript to script out the compilation of documents, a hybrid of static and dynamic. 

    ## LICENSE

    [MIT-LICENSE](https://github.com/jostylr/literate-programming/blob/master/LICENSE)

FILE README.md

## TODO

FILE TODO.md
RAW

An in-browser version is planned. The intent is to have it be an IDE for the literate program. 

More docs.

Allow for mutlitple code blocks in one block; each mini block should either add something to the whole or it could contribute to another kind; so one bit could be html, one bit could be css and one could be js and another markdown. ???

Allow for a plugin setup for directives and constants. 

Add in core directives: require (plugin), load (other literate programs), version....

Fix up javacsript code parsing in backticks to do subs. 
 

_"|trim.js" instead of Apply trim block

Using  VARS to write down the variables being used at the top of the block. Then use _"Substitute parsing|vars" to list out the variables.

    var [insert string of comma separated variables]; // name of block 


For IDE, implement: https://github.com/mleibman/SlickGrid

For diff saving: http://prettydiff.com/diffview.js  from http://stackoverflow.com/questions/3053587/javascript-based-diff-utility


For grid data input:  https://github.com/mleibman/SlickGrid

For scroll syncing https://github.com/sakabako/scrollMonitor




### Testbed

CSS

    p {color:red}

JS 
    $('p').blink();

HTML

    <p>Argh</p>


This would also give 

## NPM package

The requisite npm package file.

JSON

    {
      "name": "literate-programming",
      "description": "A literate programming compile script. Write your program in markdown.",
      "version": "0.1.0",
      "homepage": "https://github.com/jostylr/literate-programming",
      "author": {
        "name": "James Taylor",
        "email": "jostylr@gmail.com"
      },
      "repository": {
        "type": "git",
        "url": "git://github.com/jostylr/literate-programming.git"
      },
      "bugs": {
        "url": "https://github.com/jostylr/literate-programming/issues"
      },
      "licenses": [
        {
          "type": "MIT",
          "url": "https://github.com/jostylr/literate-programming/blob/master/LICENSE-MIT"
        }
      ],
      "main": "lib/literate-programming.js",
      "engines": {
        "node": ">0.6"
      },
      "dependencies":{
        "marked" : "~0.2.7",
        "js-beautify": "~0.3.1",
        "jshint" : "~0.9.1",
        "commander" : "~1.1.1"
      },
      "keywords": ["literate programming"],
      "preferGlobal": "true",
      "bin": {
        "literate-programming" : "bin/literate-programming.js"
      }
    }

FILE package.json
JS.HINT

## LICENSE-MIT

TEXT

    The MIT License (MIT)
    Copyright (c) 2013 James Taylor

    Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

FILE LICENSE



## The Docs

Here we write the basic docs for the user.

    lp3 is the third iteration of the literate programming attempt by jostylr. It handles multiple files, substitutions, macros, and subheading directives. 

    To run this, the basic command is

        node lp3.js file.md
    
    where file.md is the literate program to be parsed.

    The literate program has multiple sections that get weaved together. 

    _"Directives Doc"

    _"Substitution Doc"

    _"Subheading directives Doc"

    _"Further information on literate programming"


 //FILE: lp3Doc.md

### Further information on literate programming






## Directive Implementation

* FILE already present; puts the code block and its substitutions in a file. regex?
* ADD take this code block and add it to another codeblock. Specify pre, post, and a possible priority level. 
* DEBUG Some debugging output code. 
* RETURN returns control to other code block
* LOAD give file name, location, section to import, version number. Create a literate program manager kind of thing like npm. Have tests that run to make sure compatible. Could do latest if it passes and works back until working version or something. 
* MODULE grab a module from a repository:  install command, link to repo/other comments
* COMMAND write a command line snippet that will be run. SECURITY problem. 
* VERSION gives a version label to the section. Semantics is the version number followed by any comments about it. 
* AFTER/BEFORE/REPLACE takes in section name, regex string to match and will place the code block after the match/before the match/instead of the match respectively. It will do 


### JavaScript Directives

* VARS A list of the variables for the snippet of interest. This is mainly to deal with loop variables. 
* HEREVARS places the variables of a section there. It can be a comma separated list of headings; the vars will be placed in that order.
* SET/TEST/RESULT is a test code for snippets. SET will set variable values for multiple tests, TEST will then run the snippet and RESULT checks it. 

In all of these a parenthetical right after the term will link it to that section. Otherwise, it relates to the current code block section. 

TEST for example could have some code above it (with substitutions) and then TEST will create one (many) tests of that code block. To get back to the rest of the code block, use RETURN. 

### Directives object

    { 
      ADD : _"Add",
      DEBUG : _"Debug",
      FIDDLE : _"Fiddle",
      RETURN : _"Return",
      LOAD : _"Load",
      MODULE : _"Module",
      REQUIRE : _"Require",
      VERSION : _"Version
    }
    

### Load 

The load directive allows one to load another literate program. This means one has to specify a location (file system or url), a version number, and a section heading to start the grabbing/compiling. One should detect looping, with failure as the probable result. 

MODULE npm, semver, [semver](https://github.com/isaacs/node-semver)

    
##### DOC

The LOAD directive is of the form LOAD: (section heading), (path or URL), (number and dots for version as in [npm's versioning](https://npmjs.org/doc/json.html#version) )

What it does is find the literate program specified, check versioning/tests, compile the code, and leave it as an otherwise normal block in the named section. 

In the file, there can be a directive about versions. There can be multiple versions.

##### EXAMPLE

Let's assume there is an example.md literate program with a section "Test Me". Then in a descriptive fashion, we would wax on about what we are using and why and whatever. Then the directive:

    LOAD: "Test Me", https://github.com/jostylr/literateprogramming/lp1.md, 4.2 
 
"Test Me" should have a directive VERSION:4.2

Then we could write some test code to confirm the behavior. 

### Add

Take this code block and add it to another codeblock. Specify pre, post, and a possible priority level. 

    function (params, current) {
      params = params.split(',');
      var block = params[0];
      var type = params[1];
      if ( (type !== "pre") && (type !== "post") ) {
        error(current, type);
      }
      var number = params[2] || 1;
      number = parseFloat(number);
      if (blocks.hasOwnProperty(block) ) {
        blocks[block][type].push([number, current]);
      } else {
        blocks[block] = [];
      }
    }


##### DOC

The format is  `ADD: "block name", "pre|post", "?#"`  where we are adding the current block to the given block name. We put it before/after depending on pre/post. If there is a number, then the larger it is, the further away from the given code block it is relative to other adders. A default of 1 is assumed. 


### Debug

This is a place that can input debugging code.

    function (params, current) {
      if (debug) {

      }
    }


* DEBUG Some debugging output code. 
* FIDDLE Something that allows one to specify different parameters and generate multiple compiles/runs or something. 
* RETURN returns control to other code block
* LOAD give file name, location, section to import, version number. Create a literate program manager kind of thing like npm. Have tests that run to make sure compatible. Could do latest if it passes and works back until working version or something. 
* MODULE different than LOAD. It loads a npm module or whatever is specfied. 
* REQUIRE is a marker for where to place modules for that section of code. All MODULEs are placed at that point, in the order they are seen.
* VERSION gives a version label to the section. Semantics is the version number followed by any comments about it. 

