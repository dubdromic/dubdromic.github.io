---
layout: post
title: 'Notes: Remote projects with Emacs'
date: 2015-05-02
---

In the near future my development setup will shift to using a
development server for heavy lifting while my local machine acts as a
thin client. This means working out a way to edit the code hosted on
the development server in a performant, seamless way.

It turns out Emacs' tramp mode handles this use case quite well. With
no configuration at all I can call up a remote file (`M-x M-f` then
`/ssh:hostname:/path/to/file`) and edit it as if it were local. Magit,
Projectile, and (surprisingly) ag.el will recognize the remote repository
and act on it transparently.

There are a few performance improvements possible with a bit of configuration.

First, make sure and enable SSH's `ControlMaster` function. It's considerably
faster to share connections.

In `~/.ssh/config`:

{% highlight bash %}
Host *
  ControlMaster auto
  ControlPath ~/.ssh/master-%r@%h:%p
{% endhighlight %}

I use it for all hosts but it can be configured per-host if needed.

Second, I set Tramp's default mode to SSH. This isn't so much a performance improvement
as a stability improvement. Anecdotally, it's less prone to timeouts.

{% highlight lisp %}
(setq tramp-default-method "ssh")
{% endhighlight %}

<strong>Edit:</strong> I've had great success with the rsync method as well.
It's more or less identical to SSH other than when saving files it just sends
the diff. Which is nice.

Finally, Projectile tries to establish the name of the repo for the
modeline when you open a file. This can be slow (because remote
directory traversals are slow). Here's how to disable it.

{% highlight lisp %}
(add-hook 'find-file-hook
          (lambda ()
            (when (file-remote-p default-directory)
              (setq-local projectile-mode-line "Projectile"))))
{% endhighlight %}

<strong>Edit:</strong> I've encountered a few issues with rspec-mode and Tramp.

First, Tramp doesn't pick up on remote rbenv binaries. This can be rectified
by adding those locations to `tramp-remote-path`. Like so:

{% highlight lisp %}
(require 'tramp)

(add-to-list 'tramp-remote-path "~/.rbenv/shims")
(add-to-list 'tramp-remote-path "~/.rbenv/bin")
{% endhighlight %}

Second, rspec-mode is ubiquitous in its use of `(buffer-file-name)`.
For Tramp buffers this is something like `/rsync:host:/path/to/file`.
The test runner is more or less given the buffer names directly. Obviously,
it can't find those files. I have a patch in the works for rspec-mode
but in the mean time I've set up a symlink on the remote machine so those
paths become "valid".

{% highlight bash %}
ln -s / /rsync:host:
{% endhighlight %}

This is ugly, and I hate myself for doing it. But it does work.