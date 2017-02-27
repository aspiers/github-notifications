github-notification
===================

This utility prevents your notifications inbox on github.com from
filling up if you read GitHub's email-based notifications from a
text-based mail client such as [mutt](http://mutt.org/).

Background and motivation
-------------------------

Like many [F/OSS](http://en.wikipedia.org/wiki/Free_and_open-source_software)
developers, I'm a [heavy user](https://github.com/aspiers)
of [GitHub](https://github.com/).  This means I interact with other
developers via GitHub multiple times a day.  GitHub has a very
nice [notifications system](https://help.github.com/articles/about-notifications/)
which lets me know when there has been some activity on a project I'm
collaborating on.

I'm a fan of David Allen's
[GTD ("Getting Things Done") system](http://gettingthingsdone.com/),
and in my experience I get the best results by minimising the number
of inboxes I have to look at every day.  So I use another great
feature of GitHub, which is the ability to have
[notification emails delivered directly to your email inbox](https://help.github.com/articles/configuring-notification-emails/).
This means I don't have to keep checking
https://github.com/notifications in addition to my email inbox.

However, this means that I receive GitHub notifications in two places.
Wouldn't it be nice if when I read them in my email inbox, GitHub
could somehow realise and mark them read at
https://github.com/notifications too, so that when I look there, I
don't end up getting reminded about notifications I've already seen in
my inbox?  Happily the folks at GitHub already thought of this too,
and came up with a solution, which used to be explained
at the bottom of
[GitHub's article on notification emails](https://help.github.com/articles/configuring-notification-emails/):

> Shared read state
> -----------------
>
> If you read a notification email, it'll automatically be marked as
> read in the Notifications section. An invisible image is embedded
> in each mail message to enable this, which means that you must
> allow viewing images from notifications@github.com in order for
> this feature to work.

GitHub has since removed this info from that page, but it's still
true.

But there's a catch!  Like many Linux geeks, I
use [mutt](http://www.mutt.org/) for reading and writing email.  (In
fact, I've been using it since 1997 and I'm still waiting for
another [MUA](http://en.wikipedia.org/wiki/Email_client) to appear
which is more powerful and lets me crunch through email faster.)
However mutt is primarily text-based, which means by default it
doesn't download images when displaying HTML-based email.  Of course,
it *can*.  But do I want it to automatically open a new tab in my
browser every time I encounter an HTML attachment?  No!  That would
slow me down horribly.  Even launching a terminal-based HTML viewer
such as [w3m](http://w3m.sourceforge.net/)
or [links](http://links.twibright.com/)
or [lynx](http://lynx.browser.org/) would be too slow.

That means my GitHub account's notification tray keeps filling up
forever and does not accurately reflect which notifications I've
read via my mail client.

Solution
--------

mutt has a nice `message-hook` feature where you can execute mutt
functions for messages matching specific criteria.  So we can use that
to pipe the whole email to this script whenever a message is being
read for the first time:

    message-hook "(~N|~O) ~f notifications@github.com" \
                 "push '<pipe-message>github-notification\n'"

This script reads the email on STDIN (or from a filename argument),
extracts the URL of the 1-pixel read notification beacon `<img>`, and
sends an HTTP request for that image, so that GitHub knows the
notification has been read.

### Disconnected operation

The flaw with the above approach is that it does not work if you are
reading email on a laptop whilst disconnected from the internet.  To
address this, `github-notifications` supports two extra options:

- `--push` means that instead of immediately requesting the URL, it
  push it into a filesystem-backed queue under
  `~/.config/github-notifications`.

- `--daemon` forks a background process which periodically polls the
  queue and sends HTTP requests for any URLs found in it, but only
  when internet connectivity is detected (by
  pinging
  [Google's Public DNS at `8.8.8.8`](https://en.wikipedia.org/wiki/Google_Public_DNS)).

So to use `github-notifications` with support for disconnected operation
enabled, simply use the `--push` option from your MUA, e.g. from mutt:

    message-hook "(~N|~O) ~f notifications@github.com" \
                 "push '<pipe-message>github-notification --push\n'"

and then launch the daemon via `github-notifications --daemon`.

Installation
------------

- Either `git clone` this repository or directly download
  [the `github-notifications` script](https://raw.githubusercontent.com/aspiers/github-notifications/master/github-notifications).
- Symlink or copy the `github-notifications` script to somewhere on
  your `$PATH`, such as `~/bin`.
- Configure your mail client so that it will pipe github notification
  emails to the script according to the principles explained in the
  above section.
- If using `--push` for disconnected operation, additionally configure
  your system to automatically launch the utility in `--daemon` mode.

Usage
-----

If everything is configured correctly, it will all happen automatically.
The daemon will log to `~/.log/github-notifications.log` so that you can
monitor its behaviour.

Development / support / feedback
--------------------------------

Please see [the `CONTRIBUTING` file](CONTRIBUTING.md).

History
-------

Here is [the original blog post I wrote about this tool](https://blog.adamspiers.org/2014/10/05/managing-your-github-notifications-inbox-with-mutt/) when I first announced it in 2014.

License
-------

See [`LICENSE.txt`](LICENSE.txt).
