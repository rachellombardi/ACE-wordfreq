# 2022 ACE Consortium Meeting Wordcount Tutorial for Submitting Multiple Jobs

Imagine you have a collection of books or texts, and you want to analyze how word
usage varies from book to book or author to author. 

## Analyzing One Text

### Test the Command

We can analyze one text by running the `wordcount.py` script, with the 
name of the book we want to analyze: 

	$ ./wordcount.py Flu_Fall_2022.txt

If you run the `ls` command, you should see a new file with the prefix `counts`
which has the results of this python script. This is the output we want to 
produce within an HTCondor job. For now, remove the output: 

	$ rm counts.Flu_Fall_2022.tsv

### Create a Submit File

To submit a single job that runs this command and analyzes the 
Flu_Fall_2022 text, we need to translate this command 
into HTCondor submit file syntax. The two main components we care about 
are (1) the actual command and (2) the needed input files. 

The command gets turned into the submit file `executable` and `arguments` options: 

	executable = wordcount.py
	arguments = Flu_Fall_2022.txt	

The input file for this job is the `Flu_Fall_2022.txt` 
text file. We include that in the following submit file option: 

	transfer_input_files    = Flu_Fall_2022.txt

There are other submit file options that control other aspects of the job, like 
where to save error and logging information, and how many resources to request per 
job. 

This tutorial has a sample submit file (`wordcount.sub`) with most of these submit file options filled in: 

	$ cat wordcount.sub

	executable = 
	arguments = 

	transfer_input_files    = 

	output        = logs/job.$(Cluster).$(Process).out
	error         = logs/job.$(Cluster).$(Process).error
	log           = logs/job.$(Cluster).$(Process).log

	request_cpus   = 1
	request_memory = 512MB
	request_disk   = 512MB

	queue 1     

Open (or create) this file with a terminal-based text editor (like `vi` or `nano`) and 
add the executable, arguments, and input information described above. 

### Submit and Monitor the Job

After saving the submit file, submit the job: 

	$ condor_submit wordcount.submit

You can check the job's progress using `condor_q`. Once it finishes, you should 
see the same `counts.Flu_Fall_2022.tsv` output. 

## Analyzing Multiple Books

Now suppose you wanted to analyze multiple books - more than one at a time. 
You could create a separate submit file for each book, and submit all of the
files manually, but you'd have a lot of file lines to modify each time
(in particular, the "arguments" and "transfer_input_files" line from the 
previous submit file). 

This would be overly verbose and tedious. HTCondor has options that make it easy to 
submit many jobs from one submit file. 

### Make a List of Inputs

First we want to make a list of inputs that we want to use for our jobs. This 
should be a list where each item on the list corresponds to a job. 

In this example, our inputs are the different text files for different books. We 
want each job to analyze a different book, so our list should just contain the 
names of these text files. We can easily create this list by using an `ls` command and 
sending the output to a file: 

	$ ls *.txt > text.list 

### Modify the Submit File

Next, we will make changes to our submit file so that it submits a job for 
each book title in our list (seen in the `text.list` file). 

Create a copy of our existing submit file, that we can use for this job submission. 

	$ cp wordcount.sub wordcount-many.sub

Then, open the file with a text editor and go to the end. We want to tell the 
`queue` keyword to use our list of inputs to submit jobs. The default syntax looks like this: 

 queue <item> from <list> 
 
 Therefore, when we modify this syntax to fit our example, we get: 

	queue book from text.list 

This statement works a little bit like a for loop. For every item in the `text.list` 
file, HTCondor will create a job. Each item can be referenced elsewhere in the submit 
file using the `text` variable name. 

Therefore, every time we used the name of the text in our submit file (in the previous example, 
everywhere you see "Flu_Fall_2022.txt") should be 
replaced with a variable. HTCondor's variable syntax looks like this: `$(variablename)`

So the following lines in the submit file should be changed to use the variable `$(text)`: 

	arguments = $(text)

	transfer_input_files = $(text)

### Submit and Monitor the Job

We're now ready to submit all of our jobs. 

	$ condor_submit wordcount-many.submit

This will now submit five jobs (one for each book on our list). Once all five 
have finished running, we should see "counts" files for each book in the directory. 
