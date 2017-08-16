---
layout: post
title:  "Syncing org-mode TODO list from Android to Fedora, using Orgzly"
date:   2017-08-16 03:12:00 +0200
categories: emacs org-mode orgzly android todo synchronization
---

## Introduction 

I love Emacs and Org-mode. It is gradually taking over my life
with its capabilities, but this is not a post about that --- though
one such post would definitely appear in the future.

This post is
about how I'm syncing my TODO list, that I manage using
Emacs [Org-Mode](http://orgmode.org/), along my mobile phone (Android) and my
laptop. 

If you have never heard of Org-Mode, then check
the [References](#references) for some links about that.

## The problem

I want to have only one place to store the things that I need to
do. Ideally I'd note everything in my laptop, on a `tasks.org` file
that I have open. 
However this isn't always possible. In many occasions I need to note
something down when my laptop is not available: stand up
meetings, water cooler discussions, non-working hours (I am not
talking only about work-related TODO things).

At those times, I'd usually note it down on a different file on my
phone, or on my agenda, or ask the person to mail me the request. Still I'd prefer to
be able to put it on my phone and then magically have it appear on my
laptop.

## A (possible) solution

The main issue was that I couldn't find a decent way to edit org
files on my phone, and then have them synced on my laptop. Sure, there is a
port of Emacs for Android and some other apps that try to provide the
same functionalities as org mode, but they were feeling more like
hacks than appropriate solutions. 

For example an app is using a database to store the org files,
ignoring that one of the biggest advantages of org files is their plain text
format. So I stopped looking at it, until a few months ago when a
friend of mine mentioned [Orgzly](http://orgzly.com/) to me. 

Orgzly stores everything in plaintext files and has some really easy ways
to capture TODO items. Also it's able to sync to my Dropbox folder, so
it should be possible to use the same file both in my phone and in my
laptop. So it looked really promising.

## My implementation

I've implemented the sync with the following steps.

### Syncing from Orgzly

The first thing you need to do, is to enable sync with Dropbox from
the Orgzly settings. You will have to specify a directory where your
org files would be stored. For the sake of the post let's call this
directory `Orgzly`. 

Make sure that you have some notebooks in Orgzly, like a `tasks`
notebook. 

**NOTE:** Orgzly adds the `.org` extension automatically,
so you don't have to specify it at the creation of the new notebook.

After you have your files, you need to manually sync. It's the bottom
entry on the drop out menu. Now you should be able to see your files
on your Dropbox directory.

**NOTE:** Orgzly sync will fail if the directory contains non org-mode
files. That means that any file that doesn't end in `.org` (like for
example a `.org~`) will break the sync. A solution for this is
mentioned later on this post.

### Syncing to the laptop

I won't discuss the installation of the Dropbox client, since I assume
it's very easy to find
the [official documentation](https://www.dropbox.com/install). Instead
I will focus on my specific configuration that resulted from the
workflow that I'm following and the issues that appeared because of that.

#### Workflow

I want to have one `tasks.org` file that opens directly when I open
Emacs. I should be able to modify this file from my laptop or mobile
phone and the changes then be synced (even if I have to initiate the
sync manually) to both devices.

#### Automatic startup

To automatically open a file with Emacs, you just need to add the
following to your `.emacs` file  (changing of course the path to the one of your file...)

~~~ elisp
;; Open tasks.org on startup
(find-file "~/Dropbox/Orgzly/tasks.org") 
~~~

#### Issue 1

This approach created the first issue.

Emacs automatically saves the changes
that you do to files ending with `~`. As mentioned earlier, this
breaks the sync with Orgzly. In Linux systems, a solution is to
disable Dropbox sync on these kind of files. This can be done with:

~~~ bash
cd ~/Dropbox/Orgzly/
dropbox exclude add *.org~
~~~

This will stop syncing the files and sync should go back to working
again. **NOTE** if you add this exception and sync is still failing,
even though you've removed the `.org~` file, check the directory from
your dropbox website, since it might have already synced the file.


#### Archiving done tasks

Assuming we'll keep using this file and this approach, we'll end up
having a number of tasks getting done. Of course we could just delete
them. However, it would be nice to be able to go back and see what
we've been doing over time. So instead of deleting them, we'd like to
use the archive functionality of Org-Mode.

I'm not yet fully familiar with these features of org-mode, so I'd
just reference a nice answer
found [here](https://stackoverflow.com/a/6998051/5576273). This leads
us to the second issue.

**NOTE:** To have the TODO items of `tasks.org` file appear in the
agenda, you'd have to add it to the files that are "monitored" for
such entries. The default key binding to do that (while you're in the
file) is `C-c [`.

#### Issue 2

When archiving a number of TODO items, Org Mode defaults into saving
them to a file with the same name as the source, adding the ending
`_archive`. 

So we have more files that break the sync. Of course we can add
another rule as earlier. However I decided to use symbolic links
instead.

I already have an `org_files` directory, where I store all of my org
files. However I didn't want to put all of these files to Dropbox and
Orgzly, for many reasons.

So to solve Issue 2, I simply created a symbolic link in this
directory, pointing to the file in the Dropbox dir. This way, when I
archive entries, they are stored in the "local" `org_files` directory,
and not the Dropbox one. 

**NOTE:** This also solves Issue 1, but I'm leaving the entry in case
there are other files that I decide to use in that directory.

The symlink is created with:

~~~ bash
ln -s ~/Dropbox/Orgzly/tasks.org tasks.org
~~~

from your own `org_files` directory.


## Conclusion

That's all. You should now be able to add tasks from your mobile phone
and have them synced to your laptop, and vice versa.

## References

Some basic links for [Org-Mode](http://orgmode.org/).

1. [Emacs org-mode examples and cookbook](http://ehneilsen.net/notebook/orgExamples/org-examples.html)
2. Org Mode YouTube videos:
   - [Getting Started With Org Mode](https://www.youtube.com/watch?v=SzA2YODtgK4)
   - [Emacs Org-mode - a system for note-taking and project planning](https://www.youtube.com/watch?v=oJTwQvgfgMM)

