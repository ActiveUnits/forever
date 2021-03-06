# forever

A simple CLI tool for ensuring that a given script runs continuously (i.e. forever).

## Installation

### Installing npm (node package manager)
<pre>
  curl http://npmjs.org/install.sh | sh
</pre>

### Installing forever
<pre>
  [sudo] npm install forever
</pre>

## Usage 
There are two distinct ways to use forever: through the command line interface, or by requiring the forever module in your own code.  

### Using forever from the command line
You can use forever to run any kind of script continuously (whether it is written in node.js or not). The usage options are simple:

<pre>
  usage: forever [start | stop | stopall | list] [options] SCRIPT [script options]

  options:
    start          start SCRIPT as a daemon
    stop           stop the daemon SCRIPT
    stopall        stop all running forever scripts
    list           list all running forever scripts

    -m MAX         Only run the specified script MAX times
    -l  LOGFILE    Logs the forever output to LOGFILE
    -o  OUTFILE    Logs stdout from child script to OUTFILE
    -e  ERRFILE    Logs stderr from child script to ERRFILE
    -p  PATH       Base path for all forever related files (pid files, etc.)
    -s, --silent   Run the child script silencing stdout and stderr
    -h, --help     You're staring at it

  [Long Running Process]
    The forever process will continue to run outputting log messages to the console.
    ex. forever -o out.log -e err.log my-script.js

  [Daemon]
    The forever process will run as a daemon which will make the target process start
    in the background. This is extremely useful for remote starting simple node.js scripts
    without using nohup. It is recommended to run start with -o -l, & -e.
    ex. forever start -l forever.log -o out.log -e err.log my-daemon.js
        forever stop my-daemon.js
</pre>

There are several samples designed to test the fault tolerance of forever. Here's a simple example: 

<pre>
  forever samples/error-on-timer.js -m 5
</pre>

### Using an instance of Forever from node.js 
You can also use forever from inside your own node.js code. 

<pre>
  var forever = require('forever');
  
  var child = new (forever.Forever)('your-filename.js'), {
    max: 3,
    silent: true,
    options: []
  });
  
  child.on('exit', this.callback);
  child.start();
</pre>

### Options available when using Forever in node.js
There are several options that you should be aware of when using forever:

<pre>
  {
    'max': 10,                  // Sets the maximum number of times a given script should run
    'forever': true,            // Indicates that this script should run forever 
    'silent': true,             // Silences the output from stdout and stderr in the parent process
    'logFile': 'path/to/file',  // Path to log output from forever process (when in daemon)
    'pidFile': 'path/to/file',  // Path to put pid information for the process(es) started
    'outFile': 'path/to/file',  // Path to log output from child stdout
    'errFile': 'path/to/file',  // Path to log output from child stderr
  }
</pre>

### Events available when using an instance of Forever in node.js
Each forever object is an instance of the node.js core EventEmitter. There are several core events that you can listen for:

* error   [err]:          Raised when an error occurs
* stop    [process]:      Raised when the target script is stopped by the user
* restart [err, forever]: Raised each time the target script is restarted
* exit    [err, forever]: Raised when the call to forever.run() completes
* stdout  [err, data]:    Raised when data is received from the child process' stdout
* stderr  [err, data]:    Raised when data is received from the child process' stderr

## Using forever module from node.js
In addition to using a Forever object, the forever module also exposes some useful methods. Each method returns an instance of an EventEmitter which emits when complete. See the [forever binary script][1] for sample usage.

### forever.load (config, callback)
Sets the specified configuration (config) for the forever module. In addition to the callback, this method also returns an event emitter which will raise the 'load' event when complete. There are two important options:

* root:    Directory to put all default forever log files 
* pidPath: Directory to put all forever *.pid files

### forever.stop (index)
Stops the forever daemon script at the specified index. These indices are the same as those returned by forever.list(). This method returns an EventEmitter that raises the 'stop' event when complete.

### forever.stopAll (format)
Stops all forever scripts currently running. This method returns an EventEmitter that raises the 'stopAll' event when complete.

### forever.list (format, procs)
Returns a list of metadata objects about each process that is being run using forever. This method is synchronous and will return the list of metadata as such.

### forever.cleanup ()
Cleans up any extraneous forever *.pid or *.fvr files that are on the target system. This method returns an EventEmitter that raises the 'cleanUp' event when complete.

## Run Tests
The test coverage for 0.3.0 is currently lacking, but will be improved in 0.3.1.
<pre>
  vows test/*-test.js --spec
</pre>

#### Author: [Charlie Robbins](http://www.charlierobbins.com)
#### Contributors: [Fedor Indutny](http://github.com/donnerjack13589)

[0]: http://nodejitsu.com
[1]: https://github.com/indexzero/forever/blob/master/bin/forever