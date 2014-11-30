# OFEXPORT2

*Updated $DATE.*

## Table of Contents

- [Introduction](#introduction)
- [Audience](#audience)
- [How it works](#how-it-works)
- [Installation](#installation)
- [Uninstallation](#uninstallation)
- [Usage](#usage)
    - [Overview](#overview)
    - [Filtering](#filtering)
- [Reference](#reference)
    - [Output and Formats](#output-and-formats)
    - [Project vs Context Mode](#project-vs-context-mode)
    - [Matching and Regular Expressions](#matching-and-regular-expressions)
    - [Date Filters](#date-filters)
    - [Sorting] (#sorting)
    - [Examples](#examples)
- [Writing a Template](#writing-a-template)
- [Building it Yourself](#building-it-yourself)
- [ofexport vs ofexport2](#ofexport-vs-ofexport2)
- [Other Approaches Considered](#other-approaches-considered)
- [Known Issues](#known-issues)

## Introduction

**ofexport2** is a tool for exporting OmniFocus data to a variety of file formats, a succesor to [ofexport](https://github.com/psidnell/ofexport/blob/master/DOCUMENTATION.md).

Before proceeding, please select the required version of this document:

- [Latest Stable Release: $VERSION](https://github.com/psidnell/ofexport2/blob/ofexport-v2-$VERSION/README.md)
- [Development Version](https://github.com/psidnell/ofexport2/blob/master/README.md)

This is an early version and at the time of writing I'm making major changes. If you need something reliable and with decent documentation then the original [ofexport](https://github.com/psidnell/ofexport/blob/master/DOCUMENTATION.md) may be the safer bet.

## Audience ##

To be able to use ofexport there are some pre-requisites. You need to:

- Have OmniFocus installed.
- Be comfortable using bash and the command line.
- Have Java 8 or know how to install it.
- Have read and appreciated The Hitchhikers Guide to the Galaxy.

Without all of the above I want nothing more to do with you. Goodbye.

## How it works

1. The tool reads the entire OmniFocus database.
2. Various command line filters are applied to eliminate unwanted data, sort, etc.
3. The remaining data is printed to the console or saved to a file in some specific format.

Currently supported export formats are:

1. Plain Text
2. Markdown
3. TaskPaper
4. CSV
5. OPML
6. XML
7. JSON

The key technologies used are:

1. [OGNL](http://commons.apache.org/proper/commons-ognl/) for specifying filters.
2. [FreeMarker](http://http://freemarker.org) for writing templates.
3. [Java 8](https://java.com/en/download/index.jsp) for the main command line program.

Other formats can be created by simply creating new FreeMarker templates.

## Installation ##

Installation is entirely manual and done from the command line, but just a matter or downloading/unpacking the zip and adding it's bin directory to your path.

### 1. You should have Java 8 already installed.

You can verify this by typing:

    java -version

You should see output similar to:

    java version "1.8.0_25"  
    Java(TM) SE Runtime Environment (build 1.8.0_25-b17)  
    Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)  

### 2. Download:

Download either:

- The latest stable version: [ofexport-v2-$VERSION.zip](https://github.com/psidnell/ofexport2/archive/ofexport-v2-$VERSION.zip)
- The current development version: [master.zip](https://github.com/psidnell/ofexport2/archive/master.zip)

Unzip this file and move/rename the root folder as you wish. For now I'm going to assume you moved and renamed it to **~/Applications/ofexport2**.

### 3. Set Execute Permission on the ofexport Shell Script ###

On the command line type:

    chmod +x ~/Applications/ofexport2/bin/ofexport

Make sure it's working by running:

    ~/Applications/ofexport2/bin/ofexport2

It should print it's command line options.

### 4. Add ofexport2 to your path

Create/Edit your **~/.bash_profile** and **add** the following lines:

    export PATH=$PATH:~/Applications/ofexport2/bin
    alias of2=ofexport2

The second line above isn't necessary but creates an alias for the program and makes the examples in this document more concise.

When done reload your environment by typing:

    . ~/.bash_profile

And verify everything has worked by typing **ofexport2** (or **of2**) and ensuring it prints it's command line options.

## Uninstallation ###

Simply delete the ofexport2 folder and remove the lines you added to your .bash_profile.

## Usage ##

### Overview ###

To print the contents of a named project (In this case I have a project called ofexport2) type:

    of2 -pn ofexport2

This outputs the following:

    Home
      ofexport2
        [ ] Create "installer"
          [ ] Print version in help
          [X] Add license, docs etc.
          [X] Create "release process"
          [X] Generate README.md from template
          [ ] maven site generation
        [X] Filters - finish - test
        [ ] Code Quality
          [ ] Coverage
          [X] Address TODOs
          [ ] Timing and stats
          [ ] Add logging
          [X] basic Javadoc
          [X] Only integration tests should use real database
          [X] format code

The default output format is a simple text list where uncompleted tasks are prefixed with a [ ] and completed tasks are prefixed with a [X].

The tool has searched all the projects for those that have the name "ofexport2"  (-pn specifies project name). For any that match it shows all the items directly above it (in this case my "Home" folder) and any items beneath it.

### Filtering ###

The usage of "-pn" above is an example of a filter. Any number of filters can be used and each filter is run on the results of the last. Thus filters can only reduce what appears in the output.

For example:

    of2 -pn ofexport2 -te '!completed'

(The single quotes are to prevent bash from seeing the '!' - it has special meaning in bash.)

The output will be:

    Home
      ofexport2
        [ ] Create "installer"
          [ ] need this month date option for TODO/DONE
          [ ] Print version in help
          [ ] maven site?
        [ ] Code Quality
          [ ] Coverage
          [ ] Timing and stats
          [ ] Add logging

The case the "-te" option is a task expression (actually an [OGNL](http://commons.apache.org/proper/commons-ognl/) expression) that is eliminating tasks that have been completed.

In OGNL '!' means "not" and "completed" is one of several attributes that a Task has.

Folders, Projects, Tasks and Contexts all have attributes that you can use in filters. To get a complete list of the attributes available you can type:

    of2 -i
    
This will print all the attributes for all the types, for example this is just some of the Task attributes:

    Task:
        available (boolean): item is available.
        blocked (boolean): item is blocked.
        completed (boolean): item is complete.
        completionDate (date): date item was completed.
        contextName (string): contextName.
        deferDate (date): date item is to start.
        dueDate (date): date item is due.
        flagged (boolean): item is flagged.
        etc ...

And typing simply:

    of2
    
Will list all of the filtering options currently available.

The filtering expressions can be any valid OGNL expression, these can be complex logical expressions:

    of2 -pe 'flagged && !available && taskCount > 1'

These expressions provide fine grained control of what's printed if required. For example if you have the following amongst your projects:

- Folder
    - Project
        - [ ] Task X
            - [X] Sub Task X
            - [ ] Sub Task Y
            - [ ] Sub Task Z

If you search for a node containing "X" as follows:

    of2 -te "name.contains(\"X\") && completed==false"
    
Then you will get:

    Folder
      Project
        [] Task X

This may seem odd because normally when a node matches yoy'd see all nodes beneath.
           
However, because the expression applies tasks, even though the root task matches, the expression will also be applied to the sub tasks where it fails. 

If you wanted to see all the children of the matching task whether completed or not and whatever their name you could try:

    of2 -te "(name.contains(\"X\") && completed==false) || included"
           
And you would get:

    Folder
      Project
        [ ] Task X
          [X] Sub Task X
          [ ] Sub Task Y
          [ ] Sub Task Z

This makes use of the special "included" attribute which is used internally by ofexport during filtering. If the current expression has evaluated to true on any node above one being matched, then evaluated will be true. If you need to do something complex then this may prove useful.

When you use -fn, -tn or -cn, they actually expand internally to be

    name="<your search>" || included 

# Reference #

#### Output and Formats ####

Output can be written to a file by using the "-o" option, e.g.

    of2 -pn "My Project" -o pyproj.md
    
The output will be in "Markdown" format because the file suffix is "md".

The supported suffixes are:

- md: Markdown format
- taskpaper: TaskPaper format
- txt: Text format
- opml: OPML format
- csv: CSV format
- html: HTML format

If you want to specify a format different from the one derived from the output file (or are printing to the console) you can use "-f <fmt>".

The format name (specified or derived from the filename suffix) is used to find a FreeMarker template file in **config/templates**.

#### Project vs Context Mode ####

Normally ofexport2 is in project mode, i.e. the project hierachy is used for filterng and output.

By using the "-c" option, the tool operates in context mode.

#### Matching and Regular Expressions #####

To search for a task that contains a substring:

    of2 -te 'name.contains("X")'
    
To use regular expressions:

    of2 -te 'name.matches(".*ollocks.*")'
    
Note that the part after the "." here is a java method call from the String class, you can use any method or expression returns a boolean:

    of2 -te 'name.matches(".*ollocks.*") && name.contains("gly") && name.length()>4'

#### Date Filters ####

There are various ways to match on dates, for example:

    of2 -te 'completed && completionDate > date("2014-11-26")'
    of2 -te 'completionDate==date("today")'
    of2 -te 'within(completionDate,"25th","yesterday")'
 
We're making use of various special functions here:

- **date** takes a date in string form and converts it to a Date object for use in the expression.
- **within** expressions takes 3 arguments (attribName, fromString, toString)

The strings formats of dates that are accepted are:

- **"2014-11-19"**: specific date (yyyy-mm-dd).
- **"Mon"**, **"mon"**, **"Monday"**, **"monday"**: the monday of this week (weeks start on Monday).
- **"-mon"**: The monday of last week.
- **"+mon"**: the monday of next week.
- **"Jan"**,**"jan"**,**"January"**,**"january"**: the 1st of January of this year.
- **"-Jan"**,**"-jan"**,**"-January"**,**"-january"**: the 1st of January last year.
- **"+Jan"**,**"+jan"**,**"+January"**,**"+january"**: the 1st of January next year.
- **"1d"**,"**+1day"**,**"-2days"**: days in the future/past.
- **"1w"**,"**+1week"**,**"-2weeks"**: weeks in the future/past.
- **"1m"**,"**+1month"**,**"-2months"**: months in the future/past.
- **"1y"**,"**+1year"**,**"-2years"**: months in the future/past.
- **"1st"**,"**2nd"**,**"23rd"**: day of this month.

### Sorting ###

By default items are sorted in the order they appear in OmniFocus. To specify an alternate order, for example sort by flagged status (unflagged then flagged):

    of2 -pn proj -ts flagged
    
To reverse the order:

    of2 -pn proj -ts r:flagged
    
It's possible (much like with filters) to chain multiple sort fields, for example to sort by flagged and then due:

    of2 -pn proj -ts r:flagged -ts dueDate

### Examples ###

TBD

## Writing a Template ##

TBD

## Building It Yourself ##

The build is a straight forward java [maven 3](http://maven.apache.org) build.

After installing maven, cd into the ofexport folder and run:

    mvn clean package 

The build folder contains two utility scripts:

- **update-site.sh** recreates the maven site reports (doc/site). 
- **pre-release.sh** recreates several files with versions/dates updated.
- **release.sh** runs the maven release goals.

Before releasing can succeed you will need to update the pom file with your own:

- developerConnection
- distributionManagement/repository/url

## ofexport vs ofexport2

The original ofexport was written in python since python comes supplied with
OS X.

However, this version is written in Java. While this is inconvenient in that it
requires installing java, it does provide access to a lot of useful technologies
such as FreeMarker and Jackson. I also write better Java than Python.

## Other Approaches Considered

I originally wanted to access the OmniFocus data using AppleScript (or JavaScript
on Yosemite) and did actually get as far as a working prototype that serialised
JSON data from the osascript command back to the controlling program. While it was
in the end quite simple to do it was unbelievably slow, taking sometimes minutes for
a large export rather than about a second when accessing the database directly.

## Known Issues ##

- Task/Project notes are stripped back to ASCII on export because wide characters seem corrupted when I retrieve them. This could be down to the encoding OmniFocus uses or it could be an issue with the SQLite Java driver. I experimented with various obvious specific encodings but that didn't help.
- Context availability is something I haven't been able to deduce from the database.
- Perspective data is something I haven't managed to decode.
- In  OmniFocus, child Contexts/Tasks are interleaved, as are child Projects/Folders. In ofexport they are not.