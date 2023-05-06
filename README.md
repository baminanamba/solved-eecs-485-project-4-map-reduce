Download Link: https://assignmentchef.com/product/solved-eecs-485-project-4-map-reduce
<br>
<h1>Introduction</h1>

In this project, you will implement a MapReduce server in Python. This will be a single machine, multi-process, multi-threaded server that will execute user-submitted MapReduce jobs. It will run each job to completion, handling failures along the way, and write the output of the job to a given directory. Once you have completed this project, you will be able to run any MapReduce job on your machine, using a MapReduce implementation you wrote!

There are two primary modules in this project: the <sub>Master </sub>, which will listen for MapReduce jobs, manage the jobs, distribute work amongst the workers, and handle faults. <sub>Worker</sub> modules register themselves with the master, and then await commands, performing map, reduce or sorting (grouping) tasks based on instructions given by the master.

You will not write map reduce programs, but rather the MapReduce server. We have provided several sample map/reduce programs that you can use to test your MapReduce server.

Refer to the <a href="https://eecs485staff.github.io/p4-mapreduce/python-processes-threads-sockets.html">Python processes, threads and sockets tutorial</a> for background and examples.

<strong>Table of Contents</strong>

Setup

Run the MapReduce Server

MapReduce Server Specification

Walk-through example

Testing

Submitting and grading Further Reading

<h1>Setup</h1>

<strong>Group registration</strong>

Register your group on the <a href="https://autograder.io/">Autograder</a>.

<h2>Project folder</h2>

Create a folder for this project (<a href="https://eecs485staff.github.io/p1-insta485-static/setup_os.html#create-a-folder">instructions</a><a href="https://eecs485staff.github.io/p1-insta485-static/setup_os.html#create-a-folder">)</a>. Your folder location might be different.

$ pwd

/Users/awdeorio/src/eecs485/p4-mapreduce

<h2>Version control</h2>

Set up version control using the <a href="https://eecs485staff.github.io/p1-insta485-static/setup_git.html">Version control tutorial</a>. You might also take a second look at the <a href="https://eecs280staff.github.io/p1-stats/setup_git.html#version-control-for-a-team">Version control for a team</a> tutorial.

After you’re done, you should have a local repository with a “clean” status and your local repository should be connected to a remote GitLab repository.

$ pwd

/Users/awdeorio/src/eecs485/p4-mapreduce

$ git status

On branch master

Your branch is up-to-date with ‘origin/master’.




nothing to commit, working tree clean

$ git remote -v

origin https://gitlab.eecs.umich.edu/awdeorio/p4-mapreduce.git (fetch) origin https://gitlab.eecs.umich.edu/awdeorio/p4-mapreduce.git (push) You should have a <sub>.gitignore</sub> file (<a href="https://eecs485staff.github.io/p4-mapreduce/setup_git.html#add-a-gitignore-file">instructions</a>).

$ pwd

/Users/awdeorio/src/eecs485/p4-mapreduce

$ head .gitignore

This is a sample .gitignore file that’s useful for EECS 485 projects.

<em>… </em>

<h2>Python virtual environment</h2>

Create a Python virtual environment using the Project 1 <a href="https://eecs485staff.github.io/p1-insta485-static/setup_virtual_env.html">Python Virtual Environment Tutorial</a>.

Check that you have a Python virtual environment, and that it’s activated (remember <sub>source </sub>env/bin/activate ).

$ pwd

/Users/awdeorio/src/eecs485/p4-mapreduce

$ ls -d env env

$ echo $VIRTUAL_ENV

/Users/awdeorio/src/eecs485/p4-mapreduce/env

<h2>Starter files</h2>

Download and unpack the starter files.

$ pwd

/Users/awdeorio/src/eecs485/p4-mapreduce

$ wget https://eecs485staff.github.io/p4-mapreduce/starter_files.tar.gz $ tar -xvzf starter_files.tar.gz

Move the starter files to your project directory and remove the original <sub>starter_files/ </sub>directory and tarball.

$ pwd

/Users/awdeorio/src/eecs485/p4-mapreduce

$ mv starter_files/<strong>*</strong> .

$ rm -rf starter_files starter_files.tar.gz You should see these files.

$ tree

<em>. </em>

├── mapreduce

│   ├── __init__.py

│   ├── master

│   │   ├── __init__.py

│   │   └── __main__.py

│   ├── submit.py

│   ├── utils.py

│   └── worker

│       ├── __init__.py

│       └── __main__.py

├── setup.py

├── tests

│   ├── correct

│   │   ├── grep_correct.txt

│   │   └── word_count_correct.txt

│   ├── exec

│   │   ├── grep_map.py

│   │   ├── grep_reduce.py │   │   ├── wc_map.sh

│   │   ├── wc_map_slow.sh

│   │   ├── wc_reduce.sh

│   │   └── wc_reduce_slow.sh

│   ├── input

│   │   ├── file01

<em>            … </em>

│   │   └── file08

│   ├── input_small

│   │   ├── file01

│   │   └── file02

│   ├── testdata

<em>        … </em>

│   ├── test_worker_8.py

│   └── utils.py

└── VERSION

Make sure to activate the Virtual Environment and install packages with Pip.

source env/bin/activate

pip install -e . # install the correct dep

Here’s a brief description of each of the starter files.

<table width="588">

 <tbody>

  <tr>

   <td width="180">VERSION</td>

   <td width="408">Version of the starter files</td>

  </tr>

  <tr>

   <td width="180">mapreduce</td>

   <td width="408">mapreduce Python package skeleton files</td>

  </tr>

  <tr>

   <td width="180">mapreduce/master/</td>

   <td width="408">mapreduce master skeleton module, implement this</td>

  </tr>

  <tr>

   <td width="180">mapreduce/worker/</td>

   <td width="408">mapreduce worker skeleton module, implement this</td>

  </tr>

  <tr>

   <td width="180">mapreduce/submit.py</td>

   <td width="408">Provided code to submit a new MapReduce job</td>

  </tr>

  <tr>

   <td width="180">mapreduce/utils.py</td>

   <td width="408">Code shared between master and worker</td>

  </tr>

  <tr>

   <td width="180">setup.py</td>

   <td width="408">mapreduce Python package configuration</td>

  </tr>

  <tr>

   <td width="180">tests/</td>

   <td width="408">Public unit tests</td>

  </tr>

  <tr>

   <td width="180">tests/exec/</td>

   <td width="408">Sample mapreduce programs, all use stdin and stdout</td>

  </tr>

  <tr>

   <td width="180">tests/correct/</td>

   <td width="408">Sample mapreduce program correct output</td>

  </tr>

  <tr>

   <td width="180">tests/input/</td>

   <td width="408">Sample mapreduce program input</td>

  </tr>

  <tr>

   <td width="180">tests/input_small/</td>

   <td width="408">Sample mapreduce program input for fast testing</td>

  </tr>

  <tr>

   <td width="180">testdata/</td>

   <td width="408">Files used by our public tests</td>

  </tr>

 </tbody>

</table>

Before making any changes to the clean starter files, it’s a good idea to make a commit to your Git repository.

<h2>Libraries</h2>

Complete the <a href="https://eecs485staff.github.io/p4-mapreduce/python-processes-threads-sockets.html">Python processes, threads and sockets tutorial</a>.

Here are some quick links to the libraries we used in our instructor implementation.

<a href="https://docs.python.org/3/library/threading.html">Python Multithreading</a>

<a href="https://docs.python.org/3/library/socket.html">Python Sockets</a>

<a href="https://amoffat.github.io/sh/">Python SH Module</a>

<a href="https://docs.python.org/3/library/json.html">Python JSON Library</a>

<a href="https://docs.python.org/3/library/logging.html">Python Logging facility</a>

We’ve also provided <a href="https://eecs485staff.github.io/p4-mapreduce/example.html#logging">sample logging code</a>

<h1>Run the MapReduce server</h1>

You will write a <sub>mapreduce</sub> Python package includes <sub>master</sub> and <sub>worker</sub> modules. Launch a master with the command line entry point <sub>mapreduce-master</sub> and a worker with <sub>mapreduce</sub>worker . We’ve also provided <sub>mapreduce-submit</sub> to send a new job to the master.

<h2>Start a master and workers</h2>

The starter code will run out of the box, it just won’t do anything. The master and the worker run as seperate processes, so you will have to start them up separately. This will start up a master which will listen on port 6000 using TCP. Then, we start up two workers, and tell them that they should communicate with the master on port 6000, and then tell them which port to listen on. The ampersand ( <sub>&amp; </sub>) means to start the process in the background.

$ mapreduce-master 6000 &amp;

$ mapreduce-worker 6000 6001 &amp;

$ mapreduce-worker 6000 6002 &amp;

See your processes running in the background. Note: use <sub>pgrep -lf</sub> on OSX and <sub>pgrep -af</sub> on GNU/Linux systems.

$ pgrep -lf mapreduce-worker

15364 /usr/local/Cellar/python3/3.6.3/Frameworks/Python.framework/Versions/3.6/Resources/P 15365 /usr/local/Cellar/python3/3.6.3/Frameworks/Python.framework/Versions/3.6/Resources/P $ pgrep -lf mapreduce-master

15353 /usr/local/Cellar/python3/3.6.3/Frameworks/Python.framework/Versions/3.6/Resources/P

Stop your processes.

$ pkill -f mapreduce-master

$ pkill -f mapreduce-worker

$ pgrep -lf mapreduce-worker  <em># no output, because no processes</em>

$ pgrep -lf mapreduce-master  <em># no output, because no processes</em>

<h2>Submit a MapReduce job</h2>

Lastly, we have also provided <sub>mapreduce/submit.py </sub>. It sends a job to the Master’s main TCP socket. You can specify the job using command line arguments.

$ mapreduce-submit –help

Usage: mapreduce-submit [OPTIONS]




Top level command line interface.




Options:

-p, –port INTEGER      Master port number, default = 6000

-i, –input DIRECTORY   Input directory, default=./tests/input

-o, –output DIRECTORY  Output directory, default=./output

-m, –mapper FILE       Mapper executable, default=./tests/exec/wc_map.sh

-r, –reducer FILE      Reducer executable,                           default=./tests/exec/wc_reduce.sh   –nmappers INTEGER      Number of mappers, default=4

–nreducers INTEGER     Number of reducers, default=1   –help                  Show this message and exit.

Here’s how to run a job. Later, we’ll simplify starting the server using a shell script. Right now we expect the job to fail because Master and Worker are not implemented.

$ pgrep -f mapreduce-master  <em># check if you already started it </em>$ pgrep -f mapreduce-worker  <em># check if you already started it</em>

$ mapreduce-master 6000 &amp;

$ mapreduce-worker 6000 6001 &amp;

$ mapreduce-worker 6000 6002 &amp;

$ mapreduce-submit –mapper tests/exec/wc_map.sh –reducer tests/exec/wc_reduce.sh

<h2>Init script</h2>

The MapReduce server is an example of a <em>service</em> (or <em>daemon</em>), a program that runs in the background. We’ll write an <em>init script</em> to start, stop and check on the map reduce master and worker processes. It should be a shell script named <sub>bin/mapreduce </sub>. Print the messages in the following examples.

Be sure to follow the shell script best practices (<a href="https://eecs485staff.github.io/p1-insta485-static/#utility-scripts">Tutorial</a><a href="https://eecs485staff.github.io/p1-insta485-static/#utility-scripts">)</a>.

<h2>Start server</h2>

Exit 1 if a master or worker is already running. Otherwise, execute the following commands.

mapreduce-master 6000 &amp; sleep 2

mapreduce-worker 6000 6001 &amp; mapreduce-worker 6000 6002 &amp;

Example

$ ./bin/mapreduce start starting mapreduce …

Example: accidentally start server when it’s already running.

$ ./bin/mapreduce start

Error: mapreduce-master is already running

<h2>Stop server</h2>

Execute the following commands. Notice that <sub>|| true</sub> will prevent a failed “nice” shutdown message from causing the script to exit early. Also notice that we automatically figure out the correct option for Netcat ( <sub>nc </sub>).

<em># Detect GNU vs BSD netcat.  We need netcat to close the connection after # sending a message, which requires different options.</em>

set +o pipefail  <em># Avoid erroneous failures due to grep returning non-zero </em><strong>if </strong>nc -h 2&gt;&amp;1 | grep -q “-c”; <strong>then   </strong>NC<strong>=</strong>“nc -c”

<strong>elif </strong>nc -h 2&gt;&amp;1 | grep -q “-N”; <strong>then   </strong>NC<strong>=</strong>“nc -N”

<strong>elif </strong>nc -h 2&gt;&amp;1 | grep -q “-C”; <strong>then   </strong>NC<strong>=</strong>“nc -C” <strong>else   </strong>echo “Error detecting netcat version.”   exit 1 <strong>fi </strong>set -o pipefail




echo ‘{“message_type”: “shutdown”}’ | $NC localhost 6000 <strong>||</strong> true sleep 2  <em># give the master time to receive signal and send to workers</em>

Then, check if a master or worker is still running and kill the process. The following example is for the master.

<strong>if </strong>pgrep -lf mapreduce-master; <strong>then   </strong>echo “killing mapreduce master …”   pkill -f mapreduce-master <strong>fi</strong>

Example 1, server responds to shutdown message.

$ ./bin/mapreduce stop stopping mapreduce …

Example 2, server doesn’t respond to shutdown message and process is killed.

./bin/mapreduce stop stopping mapreduce …

killing mapreduce master … killing mapreduce worker …

<h2>Server status</h2>

Example

$ ./bin/mapreduce start starting mapreduce … $ ./bin/mapreduce status master running workers running $ ./bin/mapreduce stop stopping mapreduce … $ ./bin/mapreduce status master not running workers not running

<h2>Restart server</h2>

Example

$ ./bin/mapreduce restart stopping mapreduce … starting mapreduce …

<h1>MapReduce server specification</h1>

Here, we describe the functionality of the MapReduce server. The fun part is that we are only defining the functionality and the communication spec, the implementation is entirely up to you. You must follow our exact specifications below, and the Master and the Worker should work independently (i.e. do not add any more data or dependencies between the two classes). Remember that the Master/Workers are listening on TCP/UDP sockets for all incoming messages. <strong>Note</strong>: To test your server, we will will only be checking for the messages we listed below. You should <em>not</em> rely on any communication other than the messages listed below.

As soon as the Master/Worker receives a message on its main TCP socket, it should handle that message to completion before continuing to listen on the TCP socket. In this spec, let’s say every message is handled in a function called <sub>handle_msg </sub>. When the message returns and ends execution, the Master will continue listening in an infinite while loop for new messages. Each TCP message should be communicated using a new TCP connection. <em>Note:</em> All communication in this project will be strings formatted using JSON; sockets receive strings but your thread must parse it into JSON.

We put [Master/Worker] before the subsections below to identify which class should handle the given functionality.

<h2>Code organization</h2>

Your code will go inside the mapreduce/master and mapreduce/worker packages, where you will define the two classes (we got you started in <sub>mapreduce/master/__main__.py</sub> and

mapreduce/worker/__main__.py ). Since we are using Python packages, you may create new files

as you see fit inside each package. We have also provided a <sub>utils.py</sub> inside <sub>mapreduce/</sub> which you can use to house code common to both Worker and Master. We will only define the communication specs for the Master and the Worker, but the actual implementation of the classes is entirely up to you.

<h2>Master overview</h2>

The Master should accept only one command line argument.

port_number : The primary TCP port that the Master should listen on.

On startup, the Master should do the following:

Create a new folder <sub>tmp </sub>. This is where we will store all intermediate files used by the MapReduce server.

Hint: <sub>os.makedirs()</sub> is an easier way to setup your <sub>tmp</sub> directories and can prevent some of the errors that functions like <sub>os.mkdir()</sub> might cause.

Hint: use os.path.join() on file paths to prevent fileNotFound errors on the autograder.

If <sub>tmp</sub> already exists, keep it

Delete any old mapreduce job folders in <sub>tmp </sub>. HINT: see the Python <a href="https://docs.python.org/3/library/glob.html">glob</a> module and use “tmp/job-*” .

Create a new thread, which will listen for UDP heartbeat messages from the workers. This should listen on ( port_number – 1 )

Create any additional threads or setup you think you may need. Another thread for fault tolerance could be helpful.

Create a new TCP socket on the given <sub>port_number</sub> and call the <sub>listen()</sub> function. Note:

only one <sub>listen()</sub> thread should remain open for the whole lifetime of the master.

Wait for incoming messages! Ignore invalid messages, including those that fail JSON decoding. To ignore these messages use a try/except when you to try to load the message as shown below

try:

msg = json.loads(msg) except JSONDecodeError:     continue

<h2>Worker overview</h2>

The Worker should accept two command line arguments.

master_port : The TCP socket that the Master is actively listening on (same as the <sub>port_number</sub>

in the Master constructor)

worker_port : The TCP socket that this worker should listen on to receive instructions from the

master

On initialization, each Worker should do a similar sequence of actions as the Master:

Get the process ID of the Worker. This will be the Worker’s unique ID, which it should then use to register with the master.

Create a new TCP socket on the given <sub>worker_port</sub> and call the <sub>listen()</sub> function. Note: only one <sub>listen()</sub> thread should remain open for the whole lifetime of the worker. Ignore invalid messages, including those that fail JSON decoding.

Send the <sub>register</sub> message to the Master

Upon receiving the <sub>register_ack</sub> message, create a new thread which will be responsible for sending heartbeat messages to the Master.

<strong>NOTE:</strong> The Master should safely ignore any heartbeat messages from a Worker before that Worker successfully registers with the Master.

<h2>Shutdown [master + worker]</h2>

Because all of of our tests require shutdown to function properly, it should be implemented first. The Master can receive a special message to initiate server shutdown. The shutdown message will be of the following form and will be received on the main TCP socket:

{

<strong>“message_type”</strong>: “shutdown”

}

The Master should forward this message to all of the Workers that have registered with it. The

Workers, upon receiving the shutdown message, should terminate as soon as possible. If the Worker is already in the middle of executing a task (as described below), it is okay for it to complete that task before being able to handle the shutdown message as both these happen inside a single thread.

After forwarding the message to all Workers, the Master should terminate itself.

At this point, you should be able to pass <sub>test_master_0 </sub>’s first part, and <sub>test_worker_0 </sub>completely. Another shutdown test is <sub>test_integration_0 </sub>, but you’ll need to implement worker registration first.

<h2>Worker registration [master + worker]</h2>

The Master should keep track of all Workers at any given time so that the work is only distributed among the ready Workers. Workers can be in the following states:

ready : Worker is ready to accept work busy : Worker is performing a task

dead : Worker has failed to ping for some amount of time

The Master must listen for registration messages from Workers. Once a Worker is ready to listen for instructions, it should send a message like this to the Master

{

<strong>“message_type”</strong> : “register”,

<strong>“worker_host”</strong> : string,

<strong>“worker_port”</strong> : int,

<strong>“worker_pid”</strong> : int

}

The Master will then respond with a message acknowledging the Worker has registered, formatted like this. After this message has been received, the Worker should start sending heartbeats. More on this later.

{

<strong>“message_type”</strong>: “register_ack”,

<strong>“worker_host”</strong>: string,

<strong>“worker_port”</strong>: int,   <strong>“worker_pid”</strong> : int

}

After the first Worker registers with the Master, the Master should check the job queue

(described later) if it has any work it can assign to the Worker (because a job could have arrived at the Master before any Workers registered). If the Master is already executing a map/group/reduce, it can wait until the next stage or wait until completion of the complete current task to assign the Worker any tasks.

At this point, you should be able to pass test_master_0 and test_worker_1 and test_integration_0 .

<h2>New job request [master]</h2>

In the event of a new job, the Master will receive the following message on its main TCP socket:

{

<strong>“message_type”</strong>: “new_master_job”,

<strong>“input_directory”</strong>: string,

<strong>“output_directory”</strong>: string,

<strong>“mapper_executable”</strong>: string,

<strong>“reducer_executable”</strong>: string,

<strong>“num_mappers”</strong> : int,

<strong>“num_reducers”</strong> : int

}

In response to a job request, the Master will create a set of new directories where all of the temporary files for the job will go, of the form <sub>tmp/job-{id} </sub>, where id is the current job counter (starting at 0 just like all counters). The directory structure will resemble this example(you should create 4 new folders for each job):

tmp   job-0/     mapper-output/     grouper-output/     reducer-output/   job-1/     mapper-output/     grouper-output/     reducer-output/

Remember, each MapReduce job occurs in 3 stages: Mapping, Grouping, Reducing. Workers will do the mapping and reducing using the given executable files independently, but the Master and Workers will have to cooperate to do the Grouping Stage. After the directories are setup, the

Master should check if there are any Workers ready to work and check whether the MapReduce server is currently executing a job. If the server is busy, or there are no available Workers, the job should be added to an internal queue (described next) and end the function execution. If there are workers and the server is not busy, than the Master can begin job execution.

At this point, you should be able to pass <sub>test_master_1 </sub>.

<h2>Job queue [master]</h2>

If a Master receives a new job while it is already executing one or when there were no ready workers, it should accept the job, create the directories, and store the job in an internal queue until the current one has finished. <strong>Note that this means that the current job’s map, group, and reduce tasks must be complete before the next job’s Mapping Stage can begin.</strong> As soon as a job finishes, the Master should process the next pending job if there is one (and if there are ready Workers) by starting its Mapping Stage. For simplicity, in this project, your MapReduce server will only execute one MapReduce task at any time.

As noted earlier, when you see the first Worker register to work, you should check the job queue for pending jobs.

<h2>Input partitioning [master]</h2>

To start off the Mapping Stage, the Master should scan the input directory and partition the input files in ‘X’ parts (where ‘X’ is the number of map tasks specified in the incoming job). After partitioning the input, the Master needs to let each Worker know what work it is responsible for. Each Worker could get zero, one, or many such tasks. The Master will send a JSON message of the following form to each Worker (on each Worker’s specific TCP socket), letting them know that they have work to do:

{

<strong>“message_type”</strong>: “new_worker_job”,   <strong>“input_files”</strong>: [list of strings],

<strong>“executable”</strong>: string,

<strong>“output_directory”</strong>: string

<strong>“worker_pid”</strong>: int }

Consider the case where there are 2 Workers available, 5 input files and 4 map tasks specified. The master should create 4 tasks, 3 with one input file each and 1 with 2 input files. It would then attempt to balance these tasks among all the workers. In this case, it would send 2 map tasks to each worker. The master does not need to wait for a done message before it assigns more tasks to a Worker, a Worker should be able to handle multiple tasks at the same time.

<h2>Mapping [workers]</h2>

When a worker receives this new task message, its <sub>handle_msg</sub> will start execution of the given executable over the specified input file, while directing the output to the given

output_directory (one output file per input file and you should run the executable on each

input file). The input is passed to the executable through standard in and is outputted to a specific file. The output file names should be the same as the input file (overwrite file if it already exists). The <sub>output_directory</sub> in the Mapping Stage will always be the mapper-output folder (i.e. tmp/job-{id}/mapper-output/ ).

For example, the Master should specify the input file is <sub>data/input/file_001.txt</sub> and the output file tmp/job-0/mapper-output/file_001.txt

Hint: See the command line package <sub>sh</sub> listed in the Libraries section. See <sub>sh.Command(…) </sub>, and the <sub>_in</sub> and <sub>_out</sub> arguments in order to funnel the input and output easily.

The Worker should be agnostic to map or reduce tasks. Regardless of the type of operation, the Worker is responsible for running the specified executable over the input files one by one, and piping to the output directory for each input file. Once a Worker has finished its task, it should send a TCP message to the Master’s main socket of the form:

{

<strong>“message_type”</strong>: “status”,

<strong>“output_files”</strong> : [list of strings],

<strong>“status”</strong>: “finished”,

<strong>“worker_pid”</strong>: int }

At this point, you should be able to pass test_worker_3 , test_worker_4 , test_worker_5 .

<h2>Grouping [master + workers]</h2>

Once all of the mappers have finished, the Master will start the Grouping Stage. This should begin right after the LAST Worker finishes the Mapping Stage (i.e. you will get a finished message from the last Worker and the <sub>handle_msg</sub> handling that message will continue this Grouping Stage).

To start the Grouping Stage, the Master looks at all of the files created by the mappers, and assigns Workers to sort and merge the files. Sorting in the Grouping Stage should happen by <strong>line </strong>not by key. If there are more files than Workers, the Master should attempt to balance the files evenly among them. If there are fewer files than Workers, it is okay if some Workers sit idle during this stage. Each Worker will be responsible for merging some number of files into one larger file. The Master will then take these files, merge them into one larger file, and then partition that file into the correct number of files for the reducers. The messages sent to the Workers should look like this:

{

<strong>“message_type”</strong>: “new_sort_job”,

<strong>“input_files”</strong>: [list of strings],

<strong>“output_file”</strong>: string,

<strong>“worker_pid”</strong>: int

}

Once the Worker has finished, it should send back a message formatted as follows:

{

<strong>“message_type”</strong>: “status”,

<strong>“output_file”</strong> : string,

<strong>“status”</strong>: “finished”

<strong>“worker_pid”</strong>: int

}

Once the Master has received all of the output files from the Grouping Stage, it should read through those files and group them into one large sorted file. You shouldn’t assume that all of the data from the Grouping Stage can fit into program memory at once. However, because all of the individual files are sorted alphabetically, we can read each file line by line and use the merge function from merge sort to create the single large sorted file. The <sub>merge</sub> function from Python’s heapq library will be helpful in doing this.

Use the following naming convention for the output of the Grouping Stage workers: <sub>grouped0 </sub>, grouped1 … etc. This help avoid duplicate or clobbered files when a worker fails.

Use the following naming convention for the output of the Grouping Stage master, which will be the inputs for the Reducing Stage. File should be named <sub>reduceX </sub>, where <sub>X</sub> is the reduce task number. If there are 4 reduce tasks specified, the master should create <sub>reduce1 </sub>, <sub>reduce2 </sub>,

reduce3 , <sub>reduce4</sub> in the grouper output directory.

At this point, you should be able to pass <sub>test_master_2 </sub>. This tests that your master can send the correct messages for mapping, grouping, and the reduce phases.

<h2>Reducing [workers]</h2>

To the Worker, this is the same as the Mapping Stage – it doesn’t need to know if it is running a map or reduce task. The Worker just runs the executable it is told to run – the Master is responsible for making sure it tells the Worker to run the correct map or reduce executable. The output_directory in the Reducing Stage will always be the <sub>reducer-output</sub> folder. Again, use

the same output file name as the input file.

Once a Worker has finished its task, it should send a TCP message to the Master’s main socket of the form:

{

<strong>“message_type”</strong>: “status”,

<strong>“output_files”</strong> : [list of strings],

<strong>“status”</strong>: “finished”

<strong>“worker_pid”</strong>: int

}

<h2>Wrapping up [master]</h2>

As soon as the master has received the last “finished” message for the reduce tasks for a given job, the Master should move the output files from the <sub>reducer-output</sub> directory to the final output directory specified by the original job creation message (The value specified by the output_directory key). In the final output directory, the files should be renamed <sub>outputfilex </sub>,

where <sub>x</sub> is the final output file number. If there are 4 final output files, the master should rename them <sub>outputfile1, outputfile2, outputfile3, outputfile4 </sub>. Create the output directory if it doesn’t already exist. Check the job queue for the next available job, or go back to listening for jobs if there isn’t one currently in the job queue.

<h2>Fault tolerance and heartbeats [master + worker]</h2>

Workers can die at any time and may not finish tasks that you send them. Your Master must accommodate for this. If a Worker misses more than 5 pings in a row, you should assume that it has died, and assign whatever work it was responsible for to another Worker machine.

Each Worker will have a heartbeat thread to send updates to Master via UDP. The messages should look like this, and should be sent every 2 seconds:

{

<strong>“message_type”</strong>: “heartbeat”,

<strong>“worker_pid”</strong>: int

}

If a Worker dies before completing <strong>all</strong> the tasks assigned to it, then <strong>all</strong> of those tasks (completed or not) should be redistributed to live Workers. At each point of the execution (mapping, grouping, reducing) the Master should attempt to evenly distribute work among available Workers. If a Worker dies while it is executing a task, the Master will have to assign that task to another Worker. You should mark the failed Worker as dead, but do not remove it from the Master’s internal data structures. This is due to constraints on the Python dictionary data structure. It can result in an error when keys are modified while iterating over the dictionary. For more info on this, please refer to this <a href="https://docs.python.org/3/c-api/dict.html">link</a><a href="https://docs.python.org/3/c-api/dict.html">.</a>

Your Master should attempt to maximize concurrency, but avoid duplication. In other words, don’t send the same task to different Workers until you know that the Worker who was previously assigned that task has died.

At this point, you should be able to pass test_master_3 , test_master_4 , test_worker_2 , test_integration_1 , test_integration_2 , and test_integration_3 .

<h1>Walk-through example</h1>

See a <a href="https://eecs485staff.github.io/p4-mapreduce/example.html">complete example here</a><a href="https://eecs485staff.github.io/p4-mapreduce/example.html">.</a>

<h1>Testing</h1>

We have provided a decently comprehensive public test suite for project 4. However, you should attempt to test some features of your mapreduce solution on your own. The walkthrough example above, should help you get started on testing your solution’s functionality.

We have provided a simple word count map and reduce example. You can use these executables, as well as the sample data provided, and compare your server’s output with the result obtained by running:

$ cat tests/input/<strong>*</strong> | ./tests/exec/wc_map.sh | sort | 

./tests/exec/wc_reduce.sh <strong>&gt;</strong> correct.txt

This will generate a file called <sub>correct.txt</sub> with the final answers, and they must match your server’s output, as follows:

$ cat tmp/job-<strong>{</strong>id<strong>}</strong>/reducer-output/<strong>*</strong> | sort <strong>&gt;</strong> output.txt

$ diff output.txt correct.txt

Note that these executables can be in any language – your server should not limit us to running map and reduce jobs written in python3! To help you test this, we have also provided you with a word count solution written in a shell script (see section below).

Note that the autograder will watch your worker and master only for the messages we specified above. Your code should have no other dependency besides the communication spec, and the messages sent in your system must match those listed in this spec exactly.

Run the public unit tests. Add the <sub>-vs</sub> flag to pytest to show output, which includes logging messages.

$ pwd

/Users/awdeorio/src/eecs485/p4-mapreduce

$ pytest -vs

<h2>Test for busy waiting</h2>

A solution that busy-waits may pass on your development machine and fail on the autograder due to a timeout. Your laptop is probably much more powerful than the restricted autograder <a href="https://eecs485staff.github.io/p4-mapreduce/python-processes-threads-sockets.html#busy-waiting">environment, so you might not notice the performance problem locally. See the </a><a href="https://eecs485staff.github.io/p4-mapreduce/python-processes-threads-sockets.html#busy-waiting">Processes, Threads and Sockets in Python Tutorial</a><a href="https://eecs485staff.github.io/p4-mapreduce/python-processes-threads-sockets.html#busy-waiting"> for an explanation of busy-waiting.</a>

To detect busy waiting, time a master without any workers. After a few seconds, kill it by pressing Control – <sub>C</sub> several times. Ignore any errors or exceptions. We can tell that this solution busywaits because the user time is similar to the real time.

$ time mapreduce-master 6000 INFO:root:Starting master:6000

<em>… </em>

real 0m4.475s user 0m4.429s sys 0m0.039s

This example does not busy wait. Notice that the user time is small compared to the real time.

$ time mapreduce-master 6000 INFO:root:Starting master:6000 <em>… </em>

real 0m3.530s user 0m0.275s sys 0m0.036s

<h2>Testing fault tolerance</h2>

This section will help you verify fault tolerance. The general idea is to kill a worker while it’s running an intentionally slow MapReduce job.

We have provided an intentionally slow MapReduce job in <sub>tests/exec/wc_map_slow.sh</sub> and tests/exec/wc_reduce_slow.sh . These executables use <sub>sleep</sub> statements to simulate a slow

running task. You may want to increase the sleep time.

$ grep -B1 sleep tests/exec/wc_<strong>*</strong>slow.sh

tests/exec/wc_map_slow.sh-# Simulate a long running job tests/exec/wc_map_slow.sh:sleep 3

—

tests/exec/wc_reduce_slow.sh-# Simulate a long running job tests/exec/wc_reduce_slow.sh:sleep 3

First, start one MapReduce Master and two Workers. Wait for them to start up.

$ pkill -f mapreduce-           <em># Kill any stale Master or Worker</em>

$ mapreduce-master 6000 &amp;       <em># Start Master</em>

$ mapreduce-worker 6000 6001 &amp;  <em># Start Worker 0</em>

$ mapreduce-worker 6000 6002 &amp;  <em># Start Worker 1</em>

Submit an intentionally slow MapReduce job and wait for the mappers to begin executing.

$ mapreduce-submit 

–input ./tests/input_small 

–output ./output 

–mapper ./tests/exec/wc_map_slow.sh      –reducer ./tests/exec/wc_reduce_slow.sh 

–nmappers 2 

–nreducers 2

Kill one of the workers while it is executing a map or reduce job. Quickly use <sub>pgrep</sub> to find the PID of a worker, and then use <sub>kill</sub> on its PID. You can use <sub>pgrep</sub> again to check that you actually killed the worker.

$ pgrep -af mapreduce-worker  <em># Linux</em>

$ pgrep -lf mapreduce-worker  <em># macOS</em>

77811 /usr/local/Cellar/python/3.7.2_1/Frameworks/Python.framework/Versions/3.7/Resources/

77893 /usr/local/Cellar/python/3.7.2_1/Frameworks/Python.framework/Versions/3.7/Resources/

$ kill 77811

$ pgrep -lf mapreduce-worker

77893 /usr/local/Cellar/python/3.7.2_1/Frameworks/Python.framework/Versions/3.7/Resources/

Here’s a way to kill one worker with one line.

$ pgrep -af mapreduce-worker | head -n1 | awk ‘{print $1}’ | xargs kill  <em># Linux</em>

$ pgrep -lf mapreduce-worker | head -n1 | awk ‘{print $1}’ | xargs kill  <em># macOS</em>

[2]-  Terminated: 15          mapreduce-worker 6000 6001

Finally, verify the correct behavior. Read to logging messages from your Master and Worker to consider:

How many mapping tasks should the first Worker have received?

How many mapping tasks should the second Worker have received?

How many sorting and reducing tasks should the first and the second Worker receive?

<strong>Pro-tip:</strong> Script this test, adding <sub>sleep</sub> commands in between commands to give them time for startup and TCP communication.

<h2>Code style</h2>

As in previous projects, all Python code should contain no errors or warnings from <sub>pycodestyle </sub>, pydocstyle , and <sub>pylint </sub>. Run pylint like this:

$ pylint –disable<strong>=</strong>no-value-for-parameter mapreduce

You may not use any external dependencies aside from what is provided in <sub>setup.py </sub>.

<h1>Submitting and grading</h1>

One team member should register your group on the autograder using the <em>create new invitation </em>feature.

Submit a tarball to the autograder, which is linked from <a href="https://eecs485.org/">https://eecs485.org</a><a href="https://eecs485.org/">.</a> Include the <sub>–</sub>disable-copyfile flag only on macOS.

$ tar 

–disable-copyfile 

–exclude ‘*__pycache__*’ 

–exclude ‘*tmp*’    -czvf submit.tar.gz    setup.py 

bin    mapreduce


