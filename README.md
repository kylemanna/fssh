Forward SSH
-----------

The goal of this project is to provide a wrapper around ssh to allow a user to execute commands on their local host from a remote host.  This assumes that the user has ssh key forwarding via an ssh agent.  Key forwarding is essential for the prompt-less login back to the local host machine.

Use Case
--------

You connect to a remote server (referred to as remote host) and run tmux on it from Linux or OS X (referred to as local host).  Occasionally you want to be able access the copy-paste buffer of your local host machine.  How do you do that?  Right now you probably rely on copy/pasting things from your terminal.  This falls short when you want to copy/paste large amounts of data or have tmux panes side by side that get in the way.

There are tools like [pbcopy/pbpaste](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/pbcopy.1.html) for Mac OS X and [xclip](http://xclip.sourceforge.net/) for Linux/X11 to modify the copy-paste buffer.  However, the problem is that the remote host doesn't know have any access to the local host.

Read on...

Goal of fssh
------------

The goal of fssh is to leverage ssh port forwarding to setup a reverse ssh connection and passes the essential parameters though environmental variables.  The environmental variables are critical to avoid port collisions with multiple sessions should the port numbers be hard coded.

	---------------------                   ------------------
	| local host        | -- ssh + fssh --> | remote host    |
	|                   |                   |                |
	| X11/Mac OS X GUI  | <-- ui_copy   --  | shell or tmux  |
	|                   | --  ui_paste  --> |                |
	---------------------                   ------------------

Example
-------

No Documentation is complete without an example.  Assume "laptop" is a machine running Mac OS X and "server" is running Ubuntu Linux.

	user@laptop:~$ fssh server
	>> Attempting reverse SSH forwarding via server:26336 to user@laptop
	user@server:~$ md5sum /etc/services
	22e9e61234a27da688971ba9377ec731  /etc/services
	user@server:~$ cat /etc/services | ui_copy
	user@server:~$ ui_paste | md5sum
	22e9e61234a27da688971ba9377ec731  -
	user@server:~$ exit
	user@laptop:~$ ui_paste | md5sum
	22e9e61234a27da688971ba9377ec731  -

The entire /etc/services file from the server is read in to the laptop's copy-paste buffer by feeding it to <code>ui_copy</code>.  The buffer is then read back from the laptop to the server and the hash is verified.  The user then logs out and invokes <code>ui_paste</code> locally on the laptop to verify that the copy-paste buffer still works locally.  The real use is that user running these terminal sessions can then just do <code>Cmd+v</code> on OS X or wheel click on Linux to paste the entire /etc/services file.

The opposite direction can be exercised as well

	<User copies a long URL via <code>Cmd+v</code>
	user@laptop:~$ fssh server
	>> Attempting reverse SSH forwarding via server:16292 to user@laptop
	user@server:~$ wget $(ui_paste)
	<Download happens>

This is incredibly handy for things like:
* Copying a semi-large text file (too big to fit on a terminal or broken up by tmux) from a remote machine or server so that it can be pasted in to a webpage (ie gist.github.com) or in to chat session for sharing.
* The opposite direction like copying a large gist file or chat output on a workstation and fetching it from a server.
* Copying text files from remotes to remotes by placing the text file in the workstation's copy-paste buffer on one server (ie <code>ui_copy</code>) and then fetching it from the workstation's copy-paste buffer on another connected server (ie <code>ui_paste</code>).


Utilities
---------
* <code>fssh</code> - create a forwarding ssh connection for use with ui_copy and ui_paste.  In addition to creating the port forward, the ssh operation should operate transparently.
* <code>ui_copy</code> - copy data from stdin to the local or remote GUI copy-paste buffer
* <code>ui_paste</code> - copy data from the local or remote GUI copy-paste buffer to stdout
