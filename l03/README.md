Lab 3
=====

Submit the solution to [PipeWatch](#pipewatch) task according to the
[Submitting instructions](#submitting) before Tuesday Oct 17 23:59:59.



PipeWatch
---------

Implement a (single threaded) program that watches multiple named pipes for
data: whenever data is available, it reads it from the pipes and writes to
stdout according to the output format below.

## Implementation details

In the following text, to "exit with an error" means to print an error to the
standard error and exit with a non-zero exit status.

Your program will receive names of the pipes to watch as arguments:
- If some of the arguments don't exist, your program must create them (using
  `mkfifo`).
- If some of the arguments refer to existing files that are not pipes, your
  program must exit with an error.
- If opening some of the pipes fails (due to permissions or other problems),
  your program must exit with an error.
- When another program finishes writing to a pipe and closes it, the reader will
  see an `EOF` event (read will return 0). Your program should "watch" the pipes
  indefinitely and thus must reopen the pipe (or use other tricks to keep
  reading from the pipe even when the writers close the pipe / exit).

Whenever there is data available to be read from a pipe, the program must read
it and print the following message to standard output

    <PIPE NAME>: <SIZE> bytes\n

where `<PIPE NAME>` is the name of the pipe as given in the arguments, `<SIZE>`
is the number of bytes that was read and `\n` is the newline character. The size
should be the total number of bytes that was read before the pipe was completely
exhausted, i.e. if your program did two reads of 1024 bytes and then final read
of 10, your program should print a single line with a size of 2058.

If the data read contains one of the following special strings, your program
must execute the associated action:

- `quit`: exit your program.
- `time`: print current local time on standard output in the format returned by
  the `ctime`/`ctime_r` functions.

These "commands" should be handled before the normal message containing the
number of read bytes is output.

Note that these strings can "span" multiple reads, i.e. you can receive `q` as
the last byte of one read and then `uit` as the first three bytes of a next read
(but you don't have to handle them across `select` calls).


You can use either C or C++ to implement your program, but you must use the
"raw" functions (`read`,`write`,`open` etc) to interact with the pipes and
`select` to handle the non-blocking aspect. You can use any C/C++ way to write
the messages to standard output and error.

```sh
man 7 fifo
man 7 pipe
man 3 mkfifo
man 2 stat
man 2 select
man 2 time
man 3 ctime
```

### Select

    man 2 select

`select` allows programs to wait for events on files descriptors: when data
becomes available for reading, when space becomes available for writing and when
special "exceptions" occur.

`select` operates on *file descriptor sets*. There are three different sets we
can ask `select` to monitor: the read, write and exception sets. Any of these
can be passed to `select` as `NULL` meaning we are not interested in monitoring
any file descriptors for this type of activity.

File descriptor sets are defined as variables of type `fd_set` (`sys/types.h`)
and manipulated through various macros: `FD_ZERO` should be used to "clear" the
whole set, `FD_SET` to set a particular file descriptor, `FD_ISSET` to test
whether a file descriptor is set (see the manpage for more details and also for
an example at the end).

A timeout parameter can also be given to `select`, in which case the call will
end when the timeout expires even if no activity is detected, otherwise `select`
would wait indefinitely.

When `select` returns, it will return the number of file descriptors that have
events pending and will modify the fd sets so that only those file descriptors
will be set. These can be used to check which fds need reading or writing.

Note that select notifies us only when new data really comes in or when more
space becomes available in a buffer (for write fds). This means that if 100
bytes come in, we will get a notification from select. However if we read only
50 of those and call select again, it will block until more data actually comes
in, even though there are still 50 bytes that could be read immediately. The
best strategy with select is therefore to read all input until we get an EAGAIN
error (though it might not always be possible to use it, i.e. when copying data
and the output buffer might fill up).


```c++
fd_set readFds; // define the set of fds that will be watched for reading
int maxFd = 0; // select needs to know the highest fd we use, see manual

FD_ZERO(&readFds);
for (/* fd in fds we are interested in reading from */) {
	if (fd + 1 > maxfd) maxfd = fd + 1;
	FD_SET(fd, &readFds); // set the fd
}

struct timeval tv; // a timeout value for select
tv.tv_sec = 10;    // set for 10 seconds
tv.tv_usec = 0;

int ret = select(maxFd, &rfds, NULL, NULL, &tv);  // wait...

if (ret == -1) {
	// error happened...
}
else if (ret == 0) {
	// timeout expired without any activity...
	// maybe we wanted to update a clock, some output ?
}
else {
	// we actually have some data to read
	for (/* fd in fds we are interested in reading from */) {
		if (FD_ISSET(fd, &readFds) {
			// fd actually has something to read
			// we should read as much as possible...
		}
	}
}
```

Submitting
----------

Submit your solution by committing required files (at least `pipewatch.cpp` or `pipewatch.c`)
under the directory `l03` and creating a pull request against the `l03` branch.

A correctly created pull request should appear in the
[list of PRs for `l03`](https://github.com/pulls?utf8=%E2%9C%93&q=is%3Aopen+is%3Apr+user%3AFMFI-UK-2-AIN-118+base%3Al03).
