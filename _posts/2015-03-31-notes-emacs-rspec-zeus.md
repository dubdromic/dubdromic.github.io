---
layout: post
title: 'Notes: Emacs, Rails, Rspec'
---

Running Ruby tests using Emacs' rspec-mode and the Zeus Rails preloader.

Use cases:

1. Switch between a Ruby file and its test.
2. Run the test of the current file (or the test itself).

A few plugins will do this out of the box (rinari?) but this was the simplest.

#### Installation ####

- Emacs, duh.
- [rbenv](https://github.com/sstephenson/rbenv#installation) and some [version of Ruby](https://github.com/sstephenson/ruby-build#installation).
- Zeus and Rspec (`gem install zeus rspec`)
- rspec-mode (`M-x package-install`)

#### Configuration ####

Emacs needs to know where the rbenv shims are located. Add this to Emacs initialization.

{% highlight lisp %}
(setenv "PATH"
      (concat
       (getenv "HOME") "/.rbenv/shims:"
       (getenv "HOME") "/.rbenv/bin:"
       (getenv "PATH")))
(setq exec-path
    (cons (concat
           (getenv "HOME") "/.rbenv/shims")
           (cons (concat (getenv "HOME") "/.rbenv/bin") exec-path)))
{% endhighlight %}

Also, a simpler rspec-mode prefix makes things easier to use.

{% highlight lisp %}
(custom-set-variables
 ;; ... other stuff ...
 `(rspec-key-command-prefix (kbd "M-s")))
{% endhighlight %}

#### Usage ####

Start up a Zeus instance and you're good to go.

- Switch to (or from) test: `M-s t`
- Run test: `M-s v`
- Run suite: `M-s a`

#### Bonus Points ####

Some Rails installations use engines, many of which include separate
spec folders for each engine.

When rspec-mode runs a test it looks for a Zeus socket to use. The
socket (`.zeus.sock`) typically appears in the Rails root. rspec-mode
shortcuts knowing where that is and just uses `../#{spec_root}`. In an
engine-specific test, `../#{spec_root}` is actually the engine root so
there won't be a socket available.

The "fix" is to symlink the socket to wherever you need it. Quick and dirty.

{% highlight bash %}
cd rails_root/
ln -s .zeus.sock engines/frontend/.zeus.sock
ln -s .zeus.sock engines/backend/.zeus.sock
# ... etc ...
{% endhighlight %}

<small><strong>Edits</strong><br /> 2015-04-14: Fixed wrong ruby-build URL</small>