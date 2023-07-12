gmi100
======

Gemini CLI protocol client written in 100 lines of ANSI C.

![demo.gif](demo.gif)

Build, run and usage
--------------------

**See EXAMPLE file with dump from terminal that shows everything.**

You should be able to compile with any C compiler that supports at c89
standard.  [OpenSSL][0] is the only dependency.

	$ ./build       # Build, you can use any compiler really
	$ ./gmi100      # Run and type first URL to start browsing
	> gemini.circumlunar.space

Program prompts with line of dots where you can do few things:

1. Type Gemini URL to visit specific site.
2. Type `q` or `c` or `x` to quit.
3. Type `n` or `j` or press Enter (`RET`) to load next site page.
4. Type `r` or `0` or `k` to refresh current page.
5. Type `b` or `p` or `h` to go back in browsing history.
6. Type a number of link on current page, for example: `12`.


Configuration
-------------

Right now there is no convenient method of configuration.  You have to
do it manually in code and recompile.  To change:

- path to history file - modify first argument of `fopen`.
- width of the lines - modify value of `W` variable.
- maximum number of printed lines - modify value of `H` variable.
- used memory (might be important for very big sites) modify `malloc`.
- shortcuts - modify cases in `switch` statement.

I would like to add possibility to modify some of those things without
modifying the source code.  I could use external configuration file
like in Suckless software or pass constant values to program with
compiler flags.  But I would prefer to do it with program arguments or
environment variables.  We will see.


How browsing history works
--------------------------

Browsing history in gmi100 works differently than regular "stack" way
that is commonly used in browsers and other regular modern software.
It is inspired by how Emacs handles undo history.  That means with the
single "back" button you can go back and forward in browsing history.
Also with that you will never loose any page you visited from history
file and I was able to write this implementation in only few lines.

After you run the program it will open or create history .gmi100 file.
Then every page you visits that is not a redirection to other page and
doesn't ask you for input will be appended at the end of history file.
File is never cleaned up by program itself to make history persistent
between sessions but that means cleaning up browsing history is your
responsibility.  But this also gives you an control over history file
content.  You can for example append some links that you want to visit
in next session to have easier access to them just by running program
and pressing "b" which will navigate to last link from history file.

During browsing session typing "b" in program prompt for the first
time will result in navigation to last link in history file.  Then if
you type "b" again it will open second to last link from history.  But
it will also append that link at the end.  You can input "b" multiple
times and it will always go back by one link in history and append it
at then end of history file at the same time.  Only if you decide to
navigate to other page by typing URL or choosing link number you will
break that cycle.  Then history "pointer" will go back to the very
bottom of the history file.  Example:

	gmi100 input session      pos  .gmi100 history file content
	========================  ===  ===============================
	
	> ......................       <EMPTY HISTORY FILE>
	
	> tilde.pink............  >>>  tilde.pink
	
	> 2.....................       tilde.pink
	                          >>>  tilde.pink/documentation.gmi
	
	> 2.....................       tilde.pink
	                               tilde.pink/documentation.gmi
	                          >>>  tilde.pink/docs/gemini.gmi
	
	> b.....................       tilde.pink
	                          >>>  tilde.pink/documentation.gmi
	                               tilde.pink/docs/gemini.gmi
	                               tilde.pink/documentation.gmi
	
	> b.....................  >>>  tilde.pink
	                               tilde.pink/documentation.gmi
	                               tilde.pink/docs/gemini.gmi
	                               tilde.pink/documentation.gmi
	                               tilde.pink
	
	> 3.....................       tilde.pink
	                               tilde.pink/documentation.gmi
	                               tilde.pink/docs/gemini.gmi
	                               tilde.pink/documentation.gmi
	                               tilde.pink
	                          >>>  gemini.circumlunar.space/


Motivation, decisions and thoughts
----------------------------------

Authors of Gemini protocol claims that it should be possible to write
Gemini client in modern language [in less than 100 lines of code][1].
There are few projects that do that in programming languages with
garbage collectors, build in dynamic data structures and useful std
libraries for string manipulation, parsing URLs etc.

Intuition suggest that such achievement is not possible in plain C.
Even tho I decided to start this silly project and see how far I can
go with just ANSI C, std libraries and one dependency - OpenSSL.

It took me around 3 weeks of lazy slow programming to get to this
point but results exceeded my expectations.  It turned out that it's
not only achievable but also it's possible to include many convenient
features like persistent browsing history, links formatting, wrapping
of lines, pagination and some error handling.

My goal was to write in c89 standard avoiding any dirty tricks that
could buy me more lines like defining imports and constant values in
compiler command or writing multiple things in single line separated
with semicolon.  I think that final result can be called a normal C
code but OFC it is very dense, hard to read and uses practices that
are normally not recommended.  Even tho I call it a success.

I was not able to make better line wrapping work.  Ideally lines
should wrap at last whitespace that fits within defined boundary and
respects wide characters.  The best I could do in given constrains was
to do a hard line wrap after defined number of bytes.  Yes - bytes, so
it is possible to split wide character in half at the end of the line.
This is the only thing that bothers me.  Line wrapping itself is very
necessary to make pagination and pagination is necessary to make this
program usable on terminals that does not support scrolling.  Maybe it
would be better to somehow integrate gmi100 with pager like "less".
Then I don't have to implement pagination and line wrapping at all.
Idea for another time.

I'm very happy that I was able to make browsing history work using
external file and not and array like in most small implementation I
have read.  With that this program is actually usable for me.  I'm
very happy about how the history works which is out of the ordinary
but I allows to have back and forward navigation with single logic.
With that I could fit 2 functionalities in single implementation.

I'm also very happy about links formatting.  Without this small
adjustment of output text I would not like to use this program for
actual browsing of Gemini space.

I thought about adding "default site" being the Gemini capsule that
opens by default when you run the program.  But that can be easily
done with small shell script or alias so I'm not going to do it.

```sh
echo "some.default.page.com" | gmi100
```

I's amazing how much can fit in 100 lines of C.


[0]: https://www.openssl.org/
[1]: https://gemini.circumlunar.space/docs/faq.gmi