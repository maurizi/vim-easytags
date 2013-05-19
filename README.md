# Miscellaneous auto-load Vim scripts

The git repository at [github.com/xolox/vim-misc] [repository] contains Vim
scripts that are used by most of the [Vim plug-ins I've written] [plugins] yet
don't really belong with any single one of the plug-ins. Basically it's an
extended standard library of Vim script functions that I wrote during the
development of my Vim plug-ins.

The miscellaneous scripts are bundled with each of my plug-ins using git
merges, so that a repository checkout of a plug-in contains everything that's
needed to get started. This means the git repository of the miscellaneous
scripts is only used to track changes in a central, public place.

## Documentation for the included functions

The documentation of the 41 functions below was extracted from 12 Vim scripts
on May 19, 2013 at 23:27.

### Handling of special buffers

The functions defined here make it easier to deal with special Vim buffers
that contain text generated by a Vim plug-in. For example my [vim-notes
plug-in] [vim-notes] generates several such buffers:

- [:RecentNotes] [RecentNotes] lists recently modified notes
- [:ShowTaggedNotes] [ShowTaggedNotes] lists notes grouped by tags
- etc.

Because the text in these buffers is generated, Vim shouldn't bother with
swap files and it should never prompt the user whether to save changes to
the generated text.

[vim-notes]: http://peterodding.com/code/vim/notes/
[RecentNotes]: http://peterodding.com/code/vim/notes/#recentnotes_command
[ShowTaggedNotes]: http://peterodding.com/code/vim/notes/#showtaggednotes_command

#### The `xolox#misc#buffer#is_empty()` function

Checks if the current buffer is an empty, unchanged buffer which can be
reused. Returns 1 if an empty buffer is found, 0 otherwise.

#### The `xolox#misc#buffer#prepare()` function

Open a special buffer, i.e. a buffer that will hold generated contents,
not directly edited by the user. The buffer can be customized by passing a
dictionary with the following key/value pairs as the first argument:

- **name** (required): The base name of the buffer (i.e. the base name of
  the file loaded in the buffer, even though it isn't really a file and
  nothing is really 'loaded' :-)
- **path** (required): The pathname of the buffer. May be relevant if
  [:lcd] [lcd] or ['autochdir'] [acd] is being used.

[lcd]: http://vimdoc.sourceforge.net/htmldoc/editing.html#:lcd
[acd]: http://vimdoc.sourceforge.net/htmldoc/options.html#'autochdir'

#### The `xolox#misc#buffer#lock()` function

Lock a special buffer so that its contents can no longer be edited.

#### The `xolox#misc#buffer#unlock()` function

Unlock a special buffer so that its content can be updated.

### Compatibility checking

This Vim script defines a version number for the miscellaneous scripts. Each
of my plug-ins compares their expected version of the miscellaneous scripts
against the version number defined inside the miscellaneous scripts.

The version number is incremented whenever a change in the miscellaneous
scripts breaks backwards compatibility. This enables my Vim plug-ins to fail
early when they detect an incompatible version, instead of breaking at the
worst possible moments :-).

#### The `xolox#misc#compat#check()` function

Expects two arguments: The name of a Vim plug-in and the version of the
miscellaneous scripts expected by the plug-in. When the active version of
the miscellaneous scripts has a different version, this will raise an
error message that explains what went wrong.

### Tab completion for user defined commands

#### The `xolox#misc#complete#keywords()` function

This function can be used to perform keyword completion for user defined
Vim commands based on the contents of the current buffer. Here's an
example of how you would use it:

    :command -nargs=* -complete=customlist,xolox#misc#complete#keywords MyCmd call s:MyCmd(<f-args>)

### String escaping functions

#### The `xolox#misc#escape#pattern()` function

Takes a single string argument and converts it into a [:substitute]
[subcmd] / [substitute()] [subfun] pattern string that matches the given
string literally.

[subfun]: http://vimdoc.sourceforge.net/htmldoc/eval.html#substitute()
[subcmd]: http://vimdoc.sourceforge.net/htmldoc/change.html#:substitute

#### The `xolox#misc#escape#substitute()` function

Takes a single string argument and converts it into a [:substitute]
[subcmd] / [substitute()] [subfun] replacement string that inserts the
given string literally.

#### The `xolox#misc#escape#shell()` function

Takes a single string argument and converts it into a quoted command line
argument.

I was going to add a long rant here about Vim's ['shellslash' option]
[shellslash], but really, it won't make any difference. Let's just suffice
to say that I have yet to encounter a single person out there who uses
this option for its intended purpose (running a UNIX style shell on
Microsoft Windows).

[shellslash]: http://vimdoc.sourceforge.net/htmldoc/options.html#'shellslash'

### List handling functions

#### The `xolox#misc#list#unique()` function

Remove duplicate values from the given list in-place (preserves order).

#### The `xolox#misc#list#binsert()` function

Performs in-place binary insertion, which depending on your use case can
be more efficient than calling Vim's [sort()] [sort] function after each
insertion (in cases where a single, final sort is not an option). Expects
three arguments:

1. A list
2. A value to insert
3. 1 (true) when case should be ignored, 0 (false) otherwise

[sort]: http://vimdoc.sourceforge.net/htmldoc/eval.html#sort()

### Functions to interact with the user

#### The `xolox#misc#msg#info()` function

Show a formatted informational message to the user. This function has the
same argument handling as Vim's [printf()] [printf] function.

[printf]: http://vimdoc.sourceforge.net/htmldoc/eval.html#printf()

#### The `xolox#misc#msg#warn()` function

Show a formatted warning message to the user. This function has the same
argument handling as Vim's [printf()] [printf] function.

#### The `xolox#misc#msg#debug()` function

Show a formatted debugging message to the user, if the user has enabled
increased verbosity by setting Vim's ['verbose'] [verbose] option to one
(1) or higher. This function has the same argument handling as Vim's
[printf()] [printf] function.

[verbose]: http://vimdoc.sourceforge.net/htmldoc/options.html#'verbose'

### Integration between Vim and its environment

#### The `xolox#misc#open#file()` function

Given a pathname as the first argument, this opens the file with the
program associated with the file type. So for example a text file might
open in Vim, an `*.html` file would probably open in your web browser and
a media file would open in a media player.

This should work on Windows, Mac OS X and most Linux distributions. If
this fails to find a file association, you can pass one or more external
commands to try as additional arguments. For example:

    :call xolox#misc#open#file('/path/to/my/file', 'firefox', 'google-chrome')

This generally shouldn't be necessary but it might come in handy now and
then.

#### The `xolox#misc#open#url()` function

Given a URL as the first argument, this opens the URL in your preferred or
best available web browser:

- In GUI environments a graphical web browser will open (or a new tab will
  be created in an existing window)
- In console Vim without a GUI environment, when you have any of `lynx`,
  `links` or `w3m` installed it will launch a command line web browser in
  front of Vim (temporarily suspending Vim)

### Vim and plug-in option handling

#### The `xolox#misc#option#get()` function

Expects one or two arguments: 1. The name of a variable and 2. the default
value if the variable does not exist.

Returns the value of the variable from a buffer local variable, global
variable or the default value, depending on which is defined.

This is used by some of my Vim plug-ins for option handling, so that users
can customize options for specific buffers.

#### The `xolox#misc#option#split()` function

Given a multi-value Vim option like ['runtimepath'] [rtp] this returns a
list of strings. For example:

    :echo xolox#misc#option#split(&runtimepath)
    ['/home/peter/Projects/Vim/misc',
     '/home/peter/Projects/Vim/colorscheme-switcher',
     '/home/peter/Projects/Vim/easytags',
     ...]

[rtp]: http://vimdoc.sourceforge.net/htmldoc/options.html#'runtimepath'

#### The `xolox#misc#option#join()` function

Given a list of strings like the ones returned by
`xolox#misc#option#split()`, this joins the strings together into a
single value that can be used to set a Vim option.

#### The `xolox#misc#option#split_tags()` function

Customized version of `xolox#misc#option#split()` with specialized
handling for Vim's ['tags' option] [tags].

[tags]: http://vimdoc.sourceforge.net/htmldoc/options.html#'tags'

#### The `xolox#misc#option#join_tags()` function

Customized version of `xolox#misc#option#join()` with specialized
handling for Vim's ['tags' option] [tags].

#### The `xolox#misc#option#eval_tags()` function

Evaluate Vim's ['tags' option] [tags] without looking at the file
system, i.e. this will report tags files that don't exist yet. Expects
the value of the ['tags' option] [tags] as the first argument. If the
optional second argument is 1 (true) only the first match is returned,
otherwise (so by default) a list with all matches is returned.

### Operating system interfaces

#### The `xolox#misc#os#is_win()` function

Returns 1 (true) when on Microsoft Windows, 0 (false) otherwise.

#### The `xolox#misc#os#exec()` function

Execute an external command (hiding the console on Microsoft Windows when
my [vim-shell plug-in] [vim-shell] is installed).

Expects a dictionary with the following key/value pairs as the first
argument:

- **command** (required): The command line to execute
- **async** (optional): set this to 1 (true) to execute the command in the
  background (asynchronously)
- **stdin** (optional): a string or list of strings with the input for the
  external command
- **check** (optional): set this to 0 (false) to disable checking of the
  exit code of the external command (by default an exception will be
  raised when the command fails)

Returns a dictionary with one or more of the following key/value pairs:

- **command** (always available): the generated command line that was used
  to run the external command
- **exit_code** (only in synchronous mode): the exit status of the
  external command (an integer, zero on success)
- **stdout** (only in synchronous mode): the output of the command on the
  standard output stream (a list of strings, one for each line)
- **stderr** (only in synchronous mode): the output of the command on the
  standard error stream (as a list of strings, one for each line)

[vim-shell]: http://peterodding.com/code/vim/shell/

### Pathname manipulation functions

#### The `xolox#misc#path#which()` function

Scan the executable search path (`$PATH`) for one or more external
programs. Expects one or more string arguments with program names. Returns
a list with the absolute pathnames of all found programs. Here's an
example:

    :echo xolox#misc#path#which('gvim', 'vim')
    ['/usr/local/bin/gvim',
     '/usr/bin/gvim',
     '/usr/local/bin/vim',
     '/usr/bin/vim']

#### The `xolox#misc#path#split()` function

Split a pathname (the first and only argument) into a list of pathname
components.

On Windows, pathnames starting with two slashes or backslashes are UNC
paths where the leading slashes are significant... In this case we split
like this:

- Input: `'//server/share/directory'`
- Result: `['//server', 'share', 'directory']`

Everything except Windows is treated like UNIX until someone has a better
suggestion :-). In this case we split like this:

- Input: `'/foo/bar/baz'`
- Result: `['/', 'foo', 'bar', 'baz']`

To join a list of pathname components back into a single pathname string,
use the `xolox#misc#path#join()` function.

#### The `xolox#misc#path#join()` function

Join a list of pathname components (the first and only argument) into a
single pathname string. This is the counterpart to the
`xolox#misc#path#split()` function and it expects a list of pathname
components as returned by `xolox#misc#path#split()`.

#### The `xolox#misc#path#directory_separator()` function

Find the preferred directory separator for the platform and settings.

#### The `xolox#misc#path#absolute()` function

Canonicalize and resolve a pathname, *regardless of whether it exists*.
This is intended to support string comparison to determine whether two
pathnames point to the same directory or file.

#### The `xolox#misc#path#relative()` function

Make an absolute pathname (the first argument) relative to a directory
(the second argument).

#### The `xolox#misc#path#merge()` function

Join a directory pathname and filename into a single pathname.

#### The `xolox#misc#path#commonprefix()` function

Find the common prefix of path components in a list of pathnames.

#### The `xolox#misc#path#encode()` function

Encode a pathname so it can be used as a filename. This uses URL encoding
to encode special characters.

#### The `xolox#misc#path#decode()` function

Decode a pathname previously encoded with `xolox#misc#path#encode()`.

#### The `xolox#misc#path#is_relative()` function

Returns true (1) when the pathname given as the first argument is
relative, false (0) otherwise.

#### The `xolox#misc#path#tempdir()` function

Create a temporary directory and return the pathname of the directory.

### String handling

#### The `xolox#misc#str#compact()` function

Compact whitespace in the string given as the first argument.

#### The `xolox#misc#str#trim()` function

Trim all whitespace from the start and end of the string given as the
first argument.

### Timing of long during operations

#### The `xolox#misc#timer#start()` function

Start a timer. This returns a list which can later be passed to
`xolox#misc#timer#stop()`.

#### The `xolox#misc#timer#stop()` function

Show a formatted debugging message to the user, if the user has enabled
increased verbosity by setting Vim's ['verbose'] [verbose] option to one
(1) or higher.

This function has the same argument handling as Vim's [printf()] [printf]
function with one difference: At the point where you want the elapsed time
to be embedded, you write `%s` and you pass the list returned by
`xolox#misc#timer#start()` as an argument.

[verbose]: http://vimdoc.sourceforge.net/htmldoc/options.html#'verbose'
[printf]: http://vimdoc.sourceforge.net/htmldoc/eval.html#printf()

#### The `xolox#misc#timer#format_timespan()` function

Format a time stamp (a string containing a formatted floating point
number) into a human friendly format, for example 70 seconds is phrased as
"1 minute and 10 seconds".

## Management of changes to the miscellaneous scripts

### How does it work?

Here's how I merge the miscellaneous scripts into a Vim plug-in repository:

1. Let git know about the `vim-misc` repository by adding the remote GitHub
   repository:

        git remote add -f vim-misc https://github.com/xolox/vim-misc.git

2. Merge the two directory trees without clobbering the `README.md` and/or
   `.gitignore` files, thanks to the selected merge strategy and options:

        git checkout master
        git merge --no-commit -s recursive -X ours vim-misc/master
        git commit -m "Merge vim-misc repository as overlay"

3. While steps 1 and 2 need to be done only once for a given repository, the
   following commands are needed every time I want to pull and merge the latest
   changes:

        git checkout master
        git fetch vim-misc master
        git merge --no-commit -s recursive -X ours vim-misc/master
        git commit -m "Merged changes to miscellaneous scripts"

### Why make things so complex?

I came up with this solution after multiple years of back and forth between Vim
Online users, the GitHub crowd and my own sanity:

1. When I started publishing my first Vim plug-ins (in June 2010) I would
   prepare ZIP archives for Vim Online using makefiles. The makefiles would
   make sure the miscellaneous scripts were included in the uploaded
   distributions. This had two disadvantages: It lost git history and the
   repositories on GitHub were not usable out of the box, so [I got complaints
   from GitHub (Pathogen) users] [github-complaints].

2. My second attempt to solve the problem used git submodules which seemed like
   the ideal solution until I actually started using them in March 2011:
   Submodules are not initialized during a normal `git clone`, you need to use
   `git clone --recursive` instead but Vim plug-in managers like [Pathogen]
   [pathogen] and [Vundle] [vundle] don't do this (at least [they didn't when I
   tried] [vundle-discussion]) so people would end up with broken checkouts.

3. After finding out that git submodules were not going to solve my problems I
   searched for other inclusion strategies supported by git. After a while I
   came upon the [subtree merge strategy] [merge-strategy] which I started
   using in May 2011 and stuck with for more than two years (because it
   generally worked fine and seemed quite robust).

4. In April 2013 the flat layout of the repository started bothering me because
   it broke my personal workflow, so I changed it to the proper directory
   layout of a Vim plug-in. Why did it break my workflow? Because I couldn't
   get my [vim-reload] [reload] plug-in to properly reload miscellaneous
   scripts without nasty hacks. Note to self: [Dropbox does not respect
   symbolic links] [dropbox-vote-350] and Vim doesn't like them either ([E746]
   [E746]).

### Compatibility issues

Regardless of the inclusion strategies discussed above, my current scheme has a
flaw: If more than one of my plug-ins are installed in a Vim profile using
[Pathogen] [pathogen] or [Vundle] [vundle], the miscellaneous autoload scripts
will all be loaded from the subdirectory of one single plug-in.

This means that when I break compatibility in the miscellaneous scripts, I have
to make sure to merge the changes into all of my plug-ins. Even then, if a user
has more than one of my plug-ins installed but updates only one of them, the
other plug-ins (that are not yet up to date) can break (because of the
backwards incompatible change).

The `xolox#misc#compat#check()` function makes sure that incompatibilities are
detected early so that the user knows which plug-in to update if
incompatibilities arise.

## Contact

If you have questions, bug reports, suggestions, etc. the author can be
contacted at <peter@peterodding.com>. The latest version is available at
<http://peterodding.com/code/vim/misc> and <http://github.com/xolox/vim-misc>.

## License

This software is licensed under the [MIT license] [mit].  
© 2013 Peter Odding &lt;<peter@peterodding.com>&gt;.


[dropbox-vote-350]: https://www.dropbox.com/votebox/350/preserve-implement-symlink-behaviour
[E746]: http://vimdoc.sourceforge.net/htmldoc/eval.html#E746
[github-complaints]: https://github.com/xolox/vim-easytags/issues/1
[merge-strategy]: http://www.kernel.org/pub/software/scm/git/docs/howto/using-merge-subtree.html
[mit]: http://en.wikipedia.org/wiki/MIT_License
[pathogen]: http://www.vim.org/scripts/script.php?script_id=2332
[plugins]: http://peterodding.com/code/vim/
[reload]: http://peterodding.com/code/vim/reload
[repository]: https://github.com/xolox/vim-misc
[vundle-discussion]: https://github.com/gmarik/vundle/pull/41
[vundle]: https://github.com/gmarik/vundle
