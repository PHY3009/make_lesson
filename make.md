## Using Make to automate analysis
adapted from [Software Carpentry](http://software-carpentry.org/)

Download (make-lesson.tar.gz)[http://swcarpentry.github.io/make-novice/make-lesson.tar.gz]

Unpack make-lesson.tar.gz:

~~~
$ tar -xvf make-lesson.tar.gz
~~~

Change into the make-lesson directory:

~~~
$ cd make-lesson
~~~

**Aside: If you are running Python 3, you must convert the python scripts in this file. Do so by:**
~~~
$ 2to3 -w plotcount.py
$ 2to3 -w wordcount.py
~~~

Suppose we have a script, `wordcount.py`, that reads in a text file,
counts the words in this text file, and outputs a data file:

~~~ 
$ python wordcount.py books/isles.txt isles.dat
~~~

If we view the first 5 rows of the data file using `head`,

~~~
$ head -5 isles.dat
~~~

we can see that the file consists of one row per word. Each row shows the word itself, 
the number of occurrences of that word, and the number of occurrences as a percentage of 
the total number of words in the text file.

~~~ 
the 3822 6.7371760973
of 2460 4.33632998414
and 1723 3.03719372466
to 1479 2.60708619778
a 1308 2.30565838181
~~~

 As another example:

~~~ 
$ python wordcount.py books/abyss.txt abyss.dat
$ head -5 abyss.dat
~~~

~~~ 
the 4044 6.35449402891
and 2807 4.41074795726
of 1907 2.99654305468
a 1594 2.50471401634
to 1515 2.38057825267
~~~

Suppose we also have a script, `plotcount.py`, that reads in a data
file and plots the 10 most frequently occurring words:

~~~ 
$ python plotcount.py isles.dat show
~~~

Close the window to exit the plot.

`plotcount.py` can also save the plot as an image (e.g. a PDF):

~~~ 
$ python plotcount.py isles.dat isles.pdf
~~~

Together these scripts implement a common workflow:

1. Read a data file.
2. Perform an analysis on this data file.
3. Write the analysis results to a new file.
4. Plot a graph of the analysis results.
5. Save the graph as an image, so we can put it in a paper.

Before we start using Make, let's clean up the files we just created:
~~~
$ rm *.dat *.jpg
~~~

Create a file, called `Makefile`, with the following content:

~~~ 
# Count words.
isles.dat : books/isles.txt
	python wordcount.py books/isles.txt isles.dat
~~~

This is a simple build file, which for
Make is called a Makefile - a file executed
by Make. Let us go through each line in turn:

* `#` denotes a *comment*. Any text from `#` to the end of the line is
  ignored by Make.
* `isles.dat` is a [target](reference.html#target), a file to be
  created, or built.
* `books/isles.txt` is a [dependency](reference.html#dependency), a
  file that is needed to build or update the target. Targets can have
  zero or more dependencies.
* `:` separates targets from dependencies.
* `python wordcount.py books/isles.txt isles.dat` is an
  [action](reference.html#action), a command to run to build or update
  the target using the dependencies. Targets can have zero or more
  actions.
* Actions are indented using the TAB character, *not* 8 spaces. This
  is a legacy of Make's 1970's origins.
* Together, the target, dependencies, and actions form a
  [rule](reference.html#rule). 

Our rule above describes how to build the target `isles.dat` using the
action `python wordcount.py` and the dependency `books/isles.txt`.

By default, Make looks for a Makefile, called `Makefile`, and we can
run Make as follows:

~~~ 
$ make
~~~

Make prints out the actions it executes:

~~~ 
python wordcount.py books/isles.txt isles.dat
~~~

If we see,

~~~ 
Makefile:3: *** missing separator.  Stop.
~~~

then we have used a space instead of a TAB characters to indent one of
our actions.

We don't have to call our Makefile `Makefile`. However, if we call it
something else we need to tell Make where to find it. This we can do
using `-f` flag. For example:

~~~ 
$ make -f Makefile
~~~

As we have re-run our Makefile, Make now informs us that:

~~~ 
make: `isles.dat' is up to date.
~~~

This is because our target, `isles.dat`, has now been created, and
Make will not create it again. To see how this works, let's pretend to
update one of the text files. Rather than opening the file in an
editor, we can use the shell `touch` command to update its timestamp
(which would happen if we did edit the file):

~~~ 
$ touch books/isles.txt
~~~

If we compare the timestamps of `books/isles.txt` and `isles.dat`,

~~~ 
$ ls -l books/isles.txt isles.dat
~~~

then we see that `isles.dat`, the target, is now older
than`books/isles.txt`, its dependency:

~~~ 
-rw-r--r--    1 mjj      Administ   323972 Jun 12 10:35 books/isles.txt
-rw-r--r--    1 mjj      Administ   182273 Jun 12 09:58 isles.dat
~~~

If we run Make again,

~~~ 
$ make
~~~

then it recreates `isles.dat`:

~~~ 
python wordcount.py books/isles.txt isles.dat
~~~

When it is asked to build a target, Make checks the 'last modification
time' of both the target and its dependencies. If any dependency has
been updated since the target, then the actions are re-run to update
the target.

We may want to remove all our data files so we can explicitly recreate
them all. We can introduce a new target, and associated rule, `clean`:

~~~ 
isles.dat : books/isles.txt wordcount.py
	python wordcount.py books/isles.txt isles.dat

clean : 
	rm -f *.dat
~~~

This is an example of a rule that has no dependencies. `clean` has no
dependencies on any `.dat` file as it makes no sense to create these
just to remove them. We just want to remove the data files whether or
not they exist. If we run Make and specify this target,

~~~ 
$ make clean
~~~

then we get:

~~~ 
rm -f *.dat
~~~

There is no actual thing built called `clean`. Rather, it is a
short-hand that we can use to execute a useful sequence of
actions. Such targets, though very useful, can lead to problems. For
example, let us recreate our data files, create a directory called
`clean`, then run Make:

~~~ 
$ make isles.dat
$ mkdir clean
$ make clean
~~~

We get:

~~~ 
make: `clean' is up to date.
~~~


Let's add another rule to the end of `Makefile`:

~~~
isles.dat : books/isles.txt wordcount.py
	python wordcount.py books/isles.txt isles.dat

isles.pdf : isles.dat plotcount.py
	python plotcount.py isles.dat isles.pdf

clean : 
	rm -f *.dat
	rm -f *.pdf
~~~

the new target isles.pdf depends on the target isles.dat. So to make both, we can simply 
type:

~~~
$ make isles.dat
$ ls
~~~

Let's add a some more processes for other books:

~~~
isles.dat : books/isles.txt wordcount.py
	python wordcount.py books/isles.txt isles.dat

isles.pdf : isles.dat plotcount.py
	python plotcount.py isles.dat isles.pdf

abyss.dat : books/abyss.txt wordcount.py
    python wordcount.py books/abyss.txt abyss.dat

abyss.pdf : abyss.dat plotcount.py
	python plotcount.py abyss.dat abyss.pdf
	
last.dat : books/last.txt wordcount.py
    python wordcount.py books/last.txt last.dat

last.pdf : last.dat plotcount.py
	python plotcount.py last.dat last.pdf

clean : 
	rm -f *.dat
	rm -f *.pdf
~~~

To run all of the commands, we need to type make <TARGET> for each one:
~~~
$ make isles.pdf
$ make abyss.pdf
$ make last.pdf
~~~

OR we can add a target `all` which will build the last of the dependencies.

~~~
all: isles.pdf abyss.pdf last.pdf

isles.dat : books/isles.txt wordcount.py
	python wordcount.py books/isles.txt isles.dat

isles.pdf : isles.dat plotcount.py
	python plotcount.py isles.dat isles.pdf

abyss.dat : books/abyss.txt wordcount.py
	python wordcount.py books/abyss.txt abyss.dat

abyss.pdf : abyss.dat plotcount.py
	python plotcount.py abyss.dat abyss.pdf
	
last.dat : books/last.txt wordcount.py
	python wordcount.py books/last.txt last.dat

last.pdf : last.dat plotcount.py
	python plotcount.py last.dat last.pdf

clean : 
	rm -f *.dat
	rm -f *.pdf
~~~

Our Makefile has a lot of duplication. For example, the names of text
files and data files are repeated in many places throughout the
Makefile. Makefiles are a form of code and, in any code, repeated code
can lead to problems e.g. we rename a data file in one part of the
Makefile but forget the rename it elsewhere. Let us set about removing
some of this repetition.

Make provides an automatic variable for this, `$@` which means the target of 
the current rule. We can use this to make our Makefile more concise:

~~~
all: isles.pdf abyss.pdf last.pdf

isles.dat : books/isles.txt wordcount.py
	python wordcount.py books/isles.txt $@

isles.pdf : isles.dat plotcount.py
	python plotcount.py isles.dat $@

abyss.dat : books/abyss.txt wordcount.py
	python wordcount.py books/abyss.txt $@

abyss.pdf : abyss.dat plotcount.py
	python plotcount.py abyss.dat $@
	
last.dat : books/last.txt wordcount.py
	python wordcount.py books/last.txt $@

last.pdf : last.dat plotcount.py
	python plotcount.py last.dat $@

clean : 
	rm -f *.dat
	rm -f *.pdf
~~~

Another automatic variable is, `$<` which means the first dependency of 
the current rule. We can use this to make our Makefile more concise:

~~~
all: isles.pdf abyss.pdf last.pdf

isles.dat : books/isles.txt wordcount.py
	python wordcount.py $< $@

isles.pdf : isles.dat plotcount.py
	python plotcount.py $< $@

abyss.dat : books/abyss.txt wordcount.py
	python wordcount.py $< $@

abyss.pdf : abyss.dat plotcount.py
	python plotcount.py $< $@
	
last.dat : books/last.txt wordcount.py
	python wordcount.py $< $@

last.pdf : last.dat plotcount.py
	python plotcount.py $< $@

clean : 
	rm -f *.dat
	rm -f *.pdf
~~~

Finally, we can use two other special variables to make it even more concise:

~~~
all: isles.pdf abyss.pdf last.pdf

%.dat : books/%.txt wordcount.py
	python wordcount.py $< $@

%.pdf : %.dat plotcount.py
	python plotcount.py $< $@
	
clean : 
	rm -f *.dat
	rm -f *.pdf
~~~