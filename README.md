Forward SSH
-----------

The goal of this project is to provide a wrapper around ssh to allow a user to execute commands on their local host from a remote host.  This assumes that the user has ssh key forwarding via an ssh agent.  Key forwarding is essential for the prompt-less login back to the local host machine.

Example Use Case
----------------

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

Utilities
---------
* <code>fssh</code> - create a forwarding ssh connection for use with ui_copy and ui_paste.  In addition to creating the port forward, the ssh operation should operate transparently.
* <code>ui_copy</code> - copy data from stdin to the local or remote GUI copy-paste buffer
* <code>ui_paste</code> - copy data from the local or remote GUI copy-paste buffer to stdout
