---
title: 'Peacenet Dev Update: October 26th, 2019'
date: 2019-10-26T16:39:58.362Z
description: >-
  I've been working a ton on porting and improving the game's Terminal
  mechanic.  I've also decided to start using Linux to develop the game.  I've
  made a lot of progress so far and I'm so happy to show it off in this first
  ever written The Peacenet development update.
image: /img/screenshot-at-2019-10-26-12-44-00.png
---
Above is a screenshot of the game so far.  As you can tell, there's quite a lot that's been added since the last time I've shown a build of this game outside of Twitter or Discord (_which, by the way, come_ [_join our Discord_](https://discord.me/bitphoenixsoftware) _\- I hear there's fresh muffins_), such as:

* Window management
* Panel buttons
* The shell itself
* Files and folders
* Various linux commands
* `/dev/null`
* Something that actually resembles the Peacegate Desktop.

You'll also notice that window border - that's not Windows 10... nope, my sweet reader... that'd be Ubuntu MATE 19.04 with the Ambiance Dark theme.  Do you like it? I do. :) But that's not what this article's about.

To start, the video version of this dev update can be watched right here:

<iframe src="https://youtube.com/embed/dxMvxkx1_Js" style="width: 480px; height: 270px;"></iframe>

So what's the big stuff?

## Window management

Windows are now handled by Peace Engine itself as part of the **WindowManager** service.  Windows can currentl be dragged, closed, maximized, minimized, and restored.  Windows do NOT rely on the Peacegate OS being active, as the Peace Engine handles windows on its own.  Effectively, this means that Peace Engine games can use windows and window management **from anywhere.**

You can also set a **window boundary rectangle** in your game's code, which tells Peace Engine where on-screen windows are able to render and where on-screen they can maximize to.  In the screenshot above, this is used to make the Terminal maximize to the desktop's workspace area without covering the panels at the top or bottom of the screen.

If this is disabled, Peace Engine will render and maximize windows inside the full screen area.

## Panel Buttons

Ahhh, yes.  One of the ShiftOS features that we've come to know and love that has been in The Peacenet since it was still called Project: Plex. Oh, how far we've come.

And, in this build, they're far more reliable.  Panel Buttons are only displayed for **Program Windows**, which are Peace Engine windows that are loaded by The Peacenet as in-game programs (such as the Terminal above.)  They display the program's window title, and soon the icon.

When you click a panel button, if its window is not in focus, the window will be un-minimized and brought to the front of the screen (above all other windows).  If the window is already in focus, it will be minimized.  There's not much more to say about panel buttons other than they've been completely fixed compared to their Unreal implementation. (It helps that we wrote all of the engine code that makes this possible. :P)

## The shell itself

What is The Peacenet without its in-game recreation of `bash`? Unplayable, that's what it is.  As a hacking game, a lot of the gameplay in The Peacenet will be fairly _terminal-centric_. It makes sense that I've spent most of my time in this build working on commands and the terminal.

I've even switched to using Ubuntu as my daily driver and Windows _as an emergency backup_ so that I can very easily pop into a real terminal and compare my in-game implementation of various Ubuntu and linux features to their actual behaviour.  I want things to be as close as possible (of course, taking somee liberties because this is a game.)

So far, the shell is missing UNIX piping and redirection, so it's impossible to chain commands together to run one after the other.  It's also impossible to write the output of a command to a file, or use the output of one command as the input of another command.  I'll be implementing that next week.

BUT, needless to say, I've done my best so far.  Working directories are implemented, so one can write or read files in the current working directory - also making `ls` work. I've added `cwd` which shows your current working directory as well.  All commands are also stored as files in the `/bin` directory.

That brings me to the next thing...

## Files and Folders

The filesystem is another HUGE part of the game and allows for more gameplay mechanics down the line.  So I've spent a lot of work here as well.  The API's not perfect, but I've tried to keep it somewhat close to that of `System.IO.File` and `System.IO.Directory`.  I've added support for "user files" and "special files", user files being files created by you - the player - and special files being things like programs, commands, and exploits.  User files can be written to by the player, of course, while special files are read-only.

The filesystem currently supports:

* Creating directories
* Checking if a directory exists
* Checking if a file exists
* Getting a list of all directories or files in a directory
* Deleting directories - recursively or not
* Deleting files
* Opening file streams

In the next few weeks, I'll be adding:

* Moving files
* Copying files
* Moving directories
* Copying directories
* Symlinks
* File transfers (Low level stuff for certain hacking mechanics).

### About `Stream` support...

As I've mentioned, I'm trying to keep the game's filesystem API as close to `System.IO` as I can.  Therefore, opening files gives you a `System.IO.Stream`.  You can treat this `Syream` just like a real file stream.  But there are some things you should know first.  Here's a comparison of user file streams and special file streams.

**User files**:

* Stream type: `BitPhoenix.Peacenet.Game.FS.UserFileStream`
* CanRead: `true`
* CanWrite: `true`
* CanSeek: `true`

**Special files**:

* Stream type: `BitPhoenix.Peacenet.Game.FS.ReadOnlyMemoryStream`
* CanRead: `true`
* CanWrite: `false`
* CanSeek: `true`

Also, in the case of `/dev/null`, you will be given `System.IO.Stream.Null` no matter what as this emulates the behaviour of `/dev/null` in real Linux.  Basically, be mindful of what type of stream you get when opening files as trying to write to a special file will crash the game.

## What commands are supported right now?

These are the commands I've added to the game so far:

* bash: The bash shell.
* cat: Prints the contents of a file.
* cd: Changes the current working directory.
* clear: Clears the screen.
* cwd: Prints the current working directory.
* echo: Prints the given text.
* exists: Prints whether the given file/directory exists.
* help: Lists all available commands.
* ls: Lists all directories and files in the current working directory.
* man: The UNIX manual.
* mkdir: Creates a directory.
* rm: Deletes the given file or directory.
* write: Debug command - writes the given text to the given file.

So basically your usual linux system commands.

## /dev/null

Because I want to have Peacegate behave just as much like Linux as I can, I'm even emulating some kernel stuff.  If for some reason you need to do so, you can read from or write to **the literal fucking abyss** by reading from or writing to /dev/null. It will always give you nothing when you read it. And anything you write to it will magically be gone. Fucking brilliant.

## Something that resembles the actual Peacegate desktop

...Because I'm redesigning Peacegate's layout.  There are now two panels.  The panel at the top has your current username, hostname, and IP address in the middle.  In the future, the stealth meter, mission timer, and other gameplay-related information will be in the top-right.  If you're connected to a remote desktop, the top right will also have a "disconnect" button - but this isn't implemented yet.   In the top-left will be quick access to in-game settings just like Hacknet, as well as access to mission chat and email.  This top panel isn't really part of the in-game OS, it's more of a HUD.

The bottom panel is where you'll find the system time, notifications, panel buttons, and the Peacegate Menu.  In this build, only the panel buttons are implemented.

Everything in between these panels is the workspace of whatever desktop you're connected to.  Windows will show here, so will desktop icons, which **I'm happy to say are coming back.**

## **Conclusion**

Minus a few engine bugfixes and performance enhancements, that's basically it for this update.  Things are coming along, for sure.

I've decided to do both a written development update as well as the usual video.  I hope you guys like this new way of doing dev updates.  I think I'll do the more in-depth explanation of each new feature in articles like this, with videos just being a basic run-through of the new features in each build.  Let me know what you guys think. :)
