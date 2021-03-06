---
title: Command-Line Programs
teaching: 60
exercises: 120
questions:
- "How can I write Python programs that will work like Unix command-line tools?"
objectives:
- "Use the values of command-line arguments in a program."
- "Handle flags and files separately in a command-line program."
- "Read data from standard input in a program so that it can be used in a pipeline."
keypoints:
- "The `sys` library connects a Python program to the system it is running on."
- "The list `sys.argv` contains the command-line arguments that a program was run with."
- "Avoid silent failures."
- "The pseudo-file `sys.stdin` connects to a program's standard input."
- "The pseudo-file `sys.stdout` connects to a program's standard output."
---

The Jupyter Notebook and other interactive tools are great for prototyping code and exploring data,
but sooner or later we will want to use our program in a pipeline
or run it in a shell script to process thousands of data files.


## Running Versus Importing

Running a Python script in bash is very similar to
importing that file in Python.
The biggest difference is that we don't want anything
to happen when we import a file, until we intentionally call the functions it contains,
whereas when running a script, we expect to see some
output printed to the console. 
 
In fact, you can use the same file for importing or for running as a script. Let's try this out.

> ## Line-check function
> At this point, you should have a function in a Jupyter notebook that takes two input files and checks that they have
> the same number of lines. 
> This could be a very useful function, that you might want to use again!
> To 'save' this function for re-use, copy and paste it into a text editor of your choice. 
> save the file with a meaningful name of your choice, making sure to use the .py extension.
>
> Now, start a new Jupyter notebook. 
> 
> 1. try importing your file as a module. For instance, if you file is called `parsing.py`, type `import parsing`.
> 2. Call your function in a new cell of your notebook, specifying the files you want to run it on. 
{: .challenge}

Voila! You can now save functions you write for later reuse. This development approach, Jupyter notebook -> module file, can be quite useful.

What if instead of importing your function, you want to run it as a stand-alone script? Try this:

~~~
python parsing.py
~~~
{: .language-bash}

What happened?

The problem is, that we have not called our function *within* the script. Edit your script now to include a function call after the function definition. Rerun your script.

Now, open a new Jupyter notebook again, and import your file again. Is this what we want to happen?

In order for a Python script to work as expected
when imported or when run as a script,
we typically put the part of the script
that produces output (i.e., calls the functions in the right order), inside a 'main' function:

~~~
def my_avg(x, y):
    return (x+y)/2

def main():
    my_avg(3,7)
~~~
{: .language-python}

This will stop your specific call to `my_avg()` from running when you import the module, but there is now no 
function call to `main()`, so we won't get output when we run it as a script either. This looks a bit like we are 
going around in circles...

However, there is a nifty trick in Python! Everything in Python is what we call an 'object', and objects have some special features defined behind the scenes. These features are normally hidden from view, but we can access them if we want to. One special feature is `__name__`

Let's see how this works:

> ## The value of `__name__`
>
> In your Jupyter notebook, with your module loaded, 
> type `print(parser.__name__)`
>
> Now, modify your script to include the following line at the bottom:
>
> ~~~
> print(__name__)
> ~~~
> {: .language-python}
{: .challenge-python}

The value of `__name__` gets 'filled in' when we import or call the code, and it is different when we import the code
as a module, or call it as a script from the command line.

We can control the behaviour of our script using this property of `__name__`.

After your `main()` function definition, include an `if` statement that only calls `main` if the file is run as 
a script:

~~~
def my_avg(x, y):
    return (x+y)/2

def main():
    my_avg(3,7)

if __name__ == '__main__':
    main()
~~~
{: .language-python}

Wonderful! However, we don't really want to hard code our specific  data into our program. It would be better if we could specify our data at the command line.

> ## FastQC analogy
>
> Everyone now has experience running the command-line program FastQC. You can ask for help:
> 
> ~~~
> fastqc -h
> ~~~
> {: .language-bash}
>
> ~~~
>           FastQC - A high throughput sequence QC analysis tool
> SYNOPSIS
>
> fastqc seqfile1 seqfile2 .. seqfileN
>
> fastqc [-o output dir] [--(no)extract] [-f fastq|bam|sam] 
>        [-c contaminant file] seqfile1 .. seqfileN
> ~~~
> {: .output}
>
> You can use optional arguments and flags:
>
> ~~~
> fastqc -o my_project/analysis/ --kmers 10 my_input_file.fastq.gz
> ~~~
> {: .language-bash}
>
> And you can wrap it all up inside a bash or batch script:
> 
> ~~~
> #SBATCH --job-name=fastqc
> #SBATCH --time=00:10:00
> #SBATCH --ntasks=1
> #SBATCH --mem=1GB
> 
> module add fastqc
>
> fastqc my_input_file.fastq.gz
> ~~~
> {: .language-bash}
>
> We can write python programs that work just like fastqc!
{: .callout}

In order to do that,
we need to make our programs work like other Unix command-line tools.

## Command-Line Arguments

Using the text editor of your choice,
save the following in a text file called `sys_version.py`:

~~~
import sys
print('version is', sys.version)
~~~
{: .language-python}

The first line imports a library called `sys`,
which is short for "system".
It defines values such as `sys.version`,
which describes which version of Python we are running.
We can run this script from the command line like this:

~~~
$ python sys_version.py
~~~
{: .language-bash}

~~~
version is 3.4.3+ (default, Jul 28 2015, 13:17:50)
[GCC 4.9.3]
~~~
{: .output}

Congratulations! You have written and called your first command line program.

Create another file called `argv_list.py` and save the following text to it.

~~~
import sys
print('sys.argv is', sys.argv)
~~~
{: .language-python}

> ## `sys.argv` type
> What is the type of `sys.argv`? Take a guess.
>
> Now, modify your `argv_list.py` script to print out the type of `sys.argv`.
>
> Re-run your script. Does the result make sense?
{: .challenge}

The strange name `argv` stands for "argument values".
Whenever Python runs a program,
it takes all of the values given on the command line
and puts them in the list `sys.argv`
so that the program can determine what they were.
If we run this program with no arguments:

~~~
$ python argv_list.py
~~~
{: .language-bash}

~~~
sys.argv is ['argv_list.py']
~~~
{: .output}

the only thing in the list is the full path to our script,
which is always `sys.argv[0]`.
If we run it with a few arguments, however:

~~~
$ python argv_list.py first second third
~~~
{: .language-bash}

~~~
sys.argv is ['argv_list.py', 'first', 'second', 'third']
~~~
{: .output}

then Python adds each of those arguments to that magic list.

> ## calculate the mean
>
> 1. Write a python script that takes two command line integers
> and calculates their mean:
>
> ~~~
> $ calc_mean.py 3 23
> ~~~
> {: .language-bash}
> 
> ~~~
> 13.0
> ~~~
> {: .output}
>
> 2. Modify your program to calculate the mean of any number of command line arguments.
> 
{: .challenge}

> ## The Right Way to Do It
#>
#> If our programs can take complex parameters or multiple filenames,
#> we shouldn't handle `sys.argv` directly.
#> Instead,
#> we should use Python's `argparse` library,
#> which handles common cases in a systematic way,
#> and also makes it easy for us to provide sensible error messages for our users.
#> We will not cover this module in this lesson
#> but you can go to Tshepang Lekhonkhobe's
#> [Argparse tutorial](http://docs.python.org/dev/howto/argparse.html)
#> that is part of Python's Official Documentation.
#{: .callout}
#
### Handling Multiple Files
#
#The next step is to teach our program how to handle multiple files.
#Since 60 lines of output per file is a lot to page through,
#we'll start by using three smaller files,
#each of which has three days of data for two patients:
#
#~~~
#$ ls small-*.csv
#~~~
#{: .language-bash}
#
#~~~
#small-01.csv small-02.csv small-03.csv
#~~~
#{: .output}
#
#~~~
#$ cat small-01.csv
#~~~
#{: .language-bash}
#
#~~~
#0,0,1
#0,1,2
#~~~
#{: .output}
#
#~~~
#$ python ../code/readings_02.py small-01.csv
#~~~
#{: .language-bash}
#
#~~~
#0.333333333333
#1.0
#~~~
#{: .output}
#
#Using small data files as input also allows us to check our results more easily:
#here,
#for example,
#we can see that our program is calculating the mean correctly for each line,
#whereas we were really taking it on faith before.
#This is yet another rule of programming:
#*test the simple things first*.
#
#We want our program to process each file separately,
#so we need a loop that executes once for each filename.
#If we specify the files on the command line,
#the filenames will be in `sys.argv`,
#but we need to be careful:
#`sys.argv[0]` will always be the name of our script,
#rather than the name of a file.
#We also need to handle an unknown number of filenames,
#since our program could be run for any number of files.
#
#The solution to both problems is to loop over the contents of `sys.argv[1:]`.
#The '1' tells Python to start the slice at location 1,
#so the program's name isn't included;
#since we've left off the upper bound,
#the slice runs to the end of the list,
#and includes all the filenames.
#Here's our changed program
#`readings_03.py`:
#
#~~~
#$ cat ../code/readings_03.py
#~~~
#{: .language-bash}
#
#~~~
#import sys
#import numpy
#
#def main():
#    script = sys.argv[0]
#    for filename in sys.argv[1:]:
#        data = numpy.loadtxt(filename, delimiter=',')
#        for m in numpy.mean(data, axis=1):
#            print(m)
#
#if __name__ == '__main__':
#   main()
#~~~
#{: .language-python}
#
#and here it is in action:
#
#~~~
#$ python ../code/readings_03.py small-01.csv small-02.csv
#~~~
#{: .language-bash}
#
#~~~
#0.333333333333
#1.0
#13.6666666667
#11.0
#~~~
#{: .output}
#
#> ## The Right Way to Do It
#>
#> At this point,
#> we have created three versions of our script called `readings_01.py`,
#> `readings_02.py`, and `readings_03.py`.
#> We wouldn't do this in real life:
#> instead,
#> we would have one file called `readings.py` that we committed to version control
#> every time we got an enhancement working.
#> For teaching,
#> though,
#> we need all the successive versions side by side.
#{: .callout}
#
### Handling Command-Line Flags
#
#The next step is to teach our program to pay attention to the `--min`, `--mean`, and `--max` flags.
#These always appear before the names of the files,
#so we could just do this:
#
#~~~
#$ cat ../code/readings_04.py
#~~~
#{: .language-bash}
#
#~~~
#import sys
#import numpy
#
#def main():
#    script = sys.argv[0]
#    action = sys.argv[1]
#    filenames = sys.argv[2:]
#
#    for f in filenames:
#        data = numpy.loadtxt(f, delimiter=',')
#
#        if action == '--min':
#            values = numpy.min(data, axis=1)
#        elif action == '--mean':
#            values = numpy.mean(data, axis=1)
#        elif action == '--max':
#            values = numpy.max(data, axis=1)
#
#        for m in values:
#            print(m)
#
#if __name__ == '__main__':
#   main()
#~~~
#{: .language-python}
#
#This works:
#
#~~~
#$ python ../code/readings_04.py --max small-01.csv
#~~~
#{: .language-bash}
#
#~~~
#1.0
#2.0
#~~~
#{: .output}
#
#but there are several things wrong with it:
#
#1.  `main` is too large to read comfortably.
#
#2. If we do not specify at least two additional arguments on the
#   command-line, one for the **flag** and one for the **filename**, but only
#   one, the program will not throw an exception but will run. It assumes that the file
#   list is empty, as `sys.argv[1]` will be considered the `action`, even if it
#   is a filename. [Silent failures]({{ page.root }}/reference/#silence-failure)  like this
#   are always hard to debug.
#
#3. The program should check if the submitted `action` is one of the three recognized flags.
#
#This version pulls the processing of each file out of the loop into a function of its own.
#It also checks that `action` is one of the allowed flags
#before doing any processing,
#so that the program fails fast:
#
#~~~
#$ cat ../code/readings_05.py
#~~~
#{: .language-bash}
#
#~~~
#import sys
#import numpy
#
#def main():
#    script = sys.argv[0]
#    action = sys.argv[1]
#    filenames = sys.argv[2:]
#    assert action in ['--min', '--mean', '--max'], \
#           'Action is not one of --min, --mean, or --max: ' + action
#    for f in filenames:
#        process(f, action)
#
#def process(filename, action):
#    data = numpy.loadtxt(filename, delimiter=',')
#
#    if action == '--min':
#        values = numpy.min(data, axis=1)
#    elif action == '--mean':
#        values = numpy.mean(data, axis=1)
#    elif action == '--max':
#        values = numpy.max(data, axis=1)
#
#    for m in values:
#        print(m)
#
#if __name__ == '__main__':
#   main()
#~~~
#{: .language-python}
#
#This is four lines longer than its predecessor,
#but broken into more digestible chunks of 8 and 12 lines.
#
### Handling Standard Input
#
#The next thing our program has to do is read data from standard input if no filenames are given
#so that we can put it in a pipeline,
#redirect input to it,
#and so on.
#Let's experiment in another script called `count_stdin.py`:
#
#~~~
#$ cat ../code/count_stdin.py
#~~~
#{: .language-bash}
#
#~~~
#import sys
#
#count = 0
#for line in sys.stdin:
#    count += 1
#
#print(count, 'lines in standard input')
#~~~
#{: .language-python}
#
#This little program reads lines from a special "file" called `sys.stdin`,
#which is automatically connected to the program's standard input.
#We don't have to open it --- Python and the operating system
#take care of that when the program starts up ---
#but we can do almost anything with it that we could do to a regular file.
#Let's try running it as if it were a regular command-line program:
#
#~~~
#$ python ../code/count_stdin.py < small-01.csv
#~~~
#{: .language-bash}
#
#~~~
#2 lines in standard input
#~~~
#{: .output}
#
#A common mistake is to try to run something that reads from standard input like this:
#
#~~~
#$ python ../code/count_stdin.py small-01.csv
#~~~
#{: .language-bash}
#
#i.e., to forget the `<` character that redirects the file to standard input.
#In this case,
#there's nothing in standard input,
#so the program waits at the start of the loop for someone to type something on the keyboard.
#Since there's no way for us to do this,
#our program is stuck,
#and we have to halt it using the `Interrupt` option from the `Kernel` menu in the Notebook.
#
#We now need to rewrite the program so that it loads data from `sys.stdin`
#if no filenames are provided.
#Luckily,
#`numpy.loadtxt` can handle either a filename or an open file as its first parameter,
#so we don't actually need to change `process`.
#Only `main` changes:
#
#~~~
#$ cat ../code/readings_06.py
#~~~
#{: .language-bash}
#
#~~~
#import sys
#import numpy
#
#def main():
#    script = sys.argv[0]
#    action = sys.argv[1]
#    filenames = sys.argv[2:]
#    assert action in ['--min', '--mean', '--max'], \
#           'Action is not one of --min, --mean, or --max: ' + action
#    if len(filenames) == 0:
#        process(sys.stdin, action)
#    else:
#        for f in filenames:
#            process(f, action)
#
#def process(filename, action):
#    data = numpy.loadtxt(filename, delimiter=',')
#
#    if action == '--min':
#        values = numpy.min(data, axis=1)
#    elif action == '--mean':
#        values = numpy.mean(data, axis=1)
#    elif action == '--max':
#        values = numpy.max(data, axis=1)
#
#    for m in values:
#        print(m)
#
#if __name__ == '__main__':
#   main()
#~~~
#{: .language-python}
#
#Let's try it out:
#
#~~~
#$ python ../code/readings_06.py --mean < small-01.csv
#~~~
#{: .language-bash}
#
#~~~
#0.333333333333
#1.0
#~~~
#{: .output}
#
#That's better.
#In fact,
#that's done:
#the program now does everything we set out to do.
#
#> ## Arithmetic on the Command Line
#>
#> Write a command-line program that does addition and subtraction:
#>
#> ~~~
#> $ python arith.py add 1 2
#> ~~~
#> {: .language-python}
#>
#> ~~~
#> 3
#> ~~~
#> {: .output}
#>
#> ~~~
#> $ python arith.py subtract 3 4
#> ~~~
#> {: .language-python}
#>
#> ~~~
#> -1
#> ~~~
#> {: .output}
#>
#> > ## Solution
#> > ~~~
#> > import sys
#> >
#> > def main():
#> >     assert len(sys.argv) == 4, 'Need exactly 3 arguments'
#> >
#> >     operator = sys.argv[1]
#> >     assert operator in ['add', 'subtract', 'multiply', 'divide'], \
#> >         'Operator is not one of add, subtract, multiply, or divide: bailing out'
#> >     try:
#> >         operand1, operand2 = float(sys.argv[2]), float(sys.argv[3])
#> >     except ValueError:
#> >         print('cannot convert input to a number: bailing out')
#> >         return
#> >
#> >     do_arithmetic(operand1, operator, operand2)
#> >
#> > def do_arithmetic(operand1, operator, operand2):
#> >
#> >     if operator == 'add':
#> >         value = operand1 + operand2
#> >     elif operator == 'subtract':
#> >         value = operand1 - operand2
#> >     elif operator == 'multiply':
#> >         value = operand1 * operand2
#> >     elif operator == 'divide':
#> >         value = operand1 / operand2
#> >     print(value)
#> >
#> > main()
#> > ~~~
#> > {: .language-python}
#> {: .solution}
#{: .challenge}
#
#> ## Finding Particular Files
#>
#> Using the `glob` module introduced [earlier]({{ page.root }}/04-files/),
#> write a simple version of `ls` that shows files in the current directory
#> with a particular suffix.
#> A call to this script should look like this:
#>
#> ~~~
#> $ python my_ls.py py
#> ~~~
#> {: .language-python}
#>
#> ~~~
#> left.py
#> right.py
#> zero.py
#> ~~~
#> {: .output}
#>
#> > ## Solution
#> > ~~~
#> > import sys
#> > import glob
#> >
#> > def main():
#> >     '''prints names of all files with sys.argv as suffix'''
#> >     assert len(sys.argv) >= 2, 'Argument list cannot be empty'
#> >     suffix = sys.argv[1] # NB: behaviour is not as you'd expect if sys.argv[1] is *
#> >     glob_input = '*.' + suffix # construct the input
#> >     glob_output = sorted(glob.glob(glob_input)) # call the glob function
#> >     for item in glob_output: # print the output
#> >         print(item)
#> >     return
#> >
#> > main()
#> > ~~~
#> > {: .language-python}
#> {: .solution}
#{: .challenge}
#
#> ## Changing Flags
#>
#> Rewrite `readings.py` so that it uses `-n`, `-m`, and `-x`
#> instead of `--min`, `--mean`, and `--max` respectively.
#> Is the code easier to read?
#> Is the program easier to understand?
#>
#> > ## Solution
#> > ~~~
#> > import sys
#> > import numpy
#> >
#> > def main():
#> >     script = sys.argv[0]
#> >     action = sys.argv[1]
#> >     filenames = sys.argv[2:]
#> >     assert action in ['-n', '-m', '-x'], \
#> >            'Action is not one of -n, -m, or -x: ' + action
#> >     if len(filenames) == 0:
#> >         process(sys.stdin, action)
#> >     else:
#> >         for f in filenames:
#> >             process(f, action)
#> >
#> > def process(filename, action):
#> >     data = numpy.loadtxt(filename, delimiter=',')
#> >
#> >     if action == '-n':
#> >         values = numpy.min(data, axis=1)
#> >     elif action == '-m':
#> >         values = numpy.mean(data, axis=1)
#> >     elif action == '-x':
#> >         values = numpy.max(data, axis=1)
#> >
#> >     for m in values:
#> >         print(m)
#> >
#> > main()
#> > ~~~
#> > {: .language-python}
#> {: .solution}
#{: .challenge}
#
#> ## Adding a Help Message
#>
#> Separately,
#> modify `readings.py` so that if no parameters are given
#> (i.e., no action is specified and no filenames are given),
#> it prints a message explaining how it should be used.
#>
#> > ## Solution
#> > ~~~
#> > # this is code/readings_08.py
#> > import sys
#> > import numpy
#> >
#> > def main():
#> >     script = sys.argv[0]
#> >     if len(sys.argv) == 1: # no arguments, so print help message
#> >         print("""Usage: python readings_08.py action filenames
#> >               action must be one of --min --mean --max
#> >               if filenames is blank, input is taken from stdin;
#> >               otherwise, each filename in the list of arguments is processed in turn""")
#> >         return
#> >
#> >     action = sys.argv[1]
#> >     filenames = sys.argv[2:]
#> >     assert action in ['--min', '--mean', '--max'], \
#> >            'Action is not one of --min, --mean, or --max: ' + action
#> >     if len(filenames) == 0:
#> >         process(sys.stdin, action)
#> >     else:
#> >         for f in filenames:
#> >             process(f, action)
#> >
#> > def process(filename, action):
#> >     data = numpy.loadtxt(filename, delimiter=',')
#> >
#> >     if action == '--min':
#> >         values = numpy.min(data, axis=1)
#> >     elif action == '--mean':
#> >         values = numpy.mean(data, axis=1)
#> >     elif action == '--max':
#> >         values = numpy.max(data, axis=1)
#> >
#> >     for m in values:
#> >         print(m)
#> >
#> > main()
#> > ~~~
#> > {: .language-python}
#> {: .solution}
#{: .challenge}
#
#> ## Adding a Default Action
#>
#> Separately,
#> modify `readings.py` so that if no action is given
#> it displays the means of the data.
#>
#> > ## Solution
#> > ~~~
#> > import sys
#> > import numpy
#> >
#> > def main():
#> >     script = sys.argv[0]
#> >     action = sys.argv[1]
#> >     if action not in ['--min', '--mean', '--max']: # if no action given
#> >         action = '--mean'    # set a default action, that being mean
#> >         filenames = sys.argv[1:] # start the filenames one place earlier in the argv list
#> >     else:
#> >         filenames = sys.argv[2:]
#> >
#> >     if len(filenames) == 0:
#> >         process(sys.stdin, action)
#> >     else:
#> >         for f in filenames:
#> >             process(f, action)
#> >
#> > def process(filename, action):
#> >     data = numpy.loadtxt(filename, delimiter=',')
#> >
#> >     if action == '--min':
#> >         values = numpy.min(data, axis=1)
#> >     elif action == '--mean':
#> >         values = numpy.mean(data, axis=1)
#> >     elif action == '--max':
#> >         values = numpy.max(data, axis=1)
#> >
#> >     for m in values:
#> >         print(m)
#> >
#> > main()
#> > ~~~
#> > {: .language-python}
#> {: .solution}
#{: .challenge}
#
#> ## A File-Checker
#>
#> Write a program called `check.py` that takes the names of one or more
#> inflammation data files as arguments
#> and checks that all the files have the same number of rows and columns.
#> What is the best way to test your program?
#>
#> > ## Solution
#> > ~~~
#> > import sys
#> > import numpy
#> >
#> > def main():
#> >     script = sys.argv[0]
#> >     filenames = sys.argv[1:]
#> >     if len(filenames) <=1: #nothing to check
#> >         print('Only 1 file specified on input')
#> >     else:
#> >         nrow0, ncol0 = row_col_count(filenames[0])
#> >         print('First file %s: %d rows and %d columns' % (filenames[0], nrow0, ncol0))
#> >         for f in filenames[1:]:
#> >             nrow, ncol = row_col_count(f)
#> >             if nrow != nrow0 or ncol != ncol0:
#> >                 print('File %s does not check: %d rows and %d columns' % (f, nrow, ncol))
#> >             else:
#> >                 print('File %s checks' % f)
#> >         return
#> >
#> > def row_col_count(filename):
#> >     try:
#> >         nrow, ncol = numpy.loadtxt(filename, delimiter=',').shape
#> >     except ValueError:
#> >         # 'ValueError' error is raised when numpy encounters lines that
#> >         # have different number of data elements in them than the rest of the lines,
#> >         # or when lines have non-numeric elements
#> >         nrow, ncol = (0, 0)
#> >     return nrow, ncol
#> >
#> > main()
#> > ~~~
#> > {: .language-python}
#> {: .solution}
#{: .challenge}
#
#> ## Counting Lines
#>
#> Write a program called `line_count.py` that works like the Unix `wc` command:
#>
#> *   If no filenames are given, it reports the number of lines in standard input.
#> *   If one or more filenames are given, it reports the number of lines in each,
#>     followed by the total number of lines.
#>
#> > ## Solution
#> > ~~~
#> > import sys
#> >
#> > def main():
#> >     '''print each input filename and the number of lines in it,
#> >        and print the sum of the number of lines'''
#> >     filenames = sys.argv[1:]
#> >     sum_nlines = 0 #initialize counting variable
#> >
#> >     if len(filenames) == 0: # no filenames, just stdin
#> >         sum_nlines = count_file_like(sys.stdin)
#> >         print('stdin: %d' % sum_nlines)
#> >     else:
#> >         for f in filenames:
#> >             n = count_file(f)
#> >             print('%s %d' % (f, n))
#> >             sum_nlines += n
#> >         print('total: %d' % sum_nlines)
#> >
#> > def count_file(filename):
#> >     '''count the number of lines in a file'''
#> >     f = open(filename,'r')
#> >     nlines = len(f.readlines())
#> >     f.close()
#> >     return(nlines)
#> >
#> > def count_file_like(file_like):
#> >     '''count the number of lines in a file-like object (eg stdin)'''
#> >     n = 0
#> >     for line in file_like:
#> >         n = n+1
#> >     return n
#> >
#> > main()
#> >
#> > ~~~
#> > {: .language-python}
#> {: .solution}
#{: .challenge}
#
#> ## Generate an Error Message
#>
#> Write a program called `check_arguments.py` that prints usage
#> then exits the program if no arguments are provided.
#> (Hint: You can use `sys.exit()` to exit the program.)
#>
#> ~~~
#> $ python check_arguments.py
#> ~~~
#> {: .language-bash}
#>
#> ~~~
#> usage: python check_argument.py filename.txt
#> ~~~
#> {: .output}
#>
#> ~~~
#> $ python check_arguments.py filename.txt
#> ~~~
#> {: .language-bash}
#>
#> ~~~
#> Thanks for specifying arguments!
#> ~~~
#> {: .output}
#{: .challenge}
#
{% include links.md %}
