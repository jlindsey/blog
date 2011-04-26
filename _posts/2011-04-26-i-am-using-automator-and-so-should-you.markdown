---
layout: post
date: 2011-04-26
time: 03:23:59 PM
title: I am using Automator, and so should you.
tags:
  - automation
  - os x
  - apple
  - ruby
---

<img src="/img/automator/automator-icon.png" alt="Automator" title="Automator" class="left" style="display: inline;">
[Automator](http://www.macosxautomation.com/automator/) is an aptly-named app that has shipped as part of Mac OS X since Tiger way back in 2005.
It received a much-needed overhaul in Leopard, and had some new actions added in Snow Leopard. Snow Leopard also overhauled the way the often-overlooked
Services menu worked, which is a kind of tangential upgrade to Automator. Most people I know who use OS X as their primary work machine (mostly web developers)
don't think much of Automator, which is a shame because it can do some really useful stuff for you.

<img src="/img/automator/services-menu-empty.png" alt="Empty Services Menu" class="right post-image">
If you use OS X, you've probably noticed the Services menu. You may have even opened it once or twice and been greeted with the screenshot you can see here. That
"No Services Apply" message is a product of the Snow Leopard changes mentioned a second ago. In SL, Services were changed to only show up in the menu if they are
applicable to the current app and situation. This screenshot is from Chrome, with just a tab open and no text selected. If we select some text, however, you can
see that the menu populates with lots of useful things.

We're getting a bit ahead of ourselves, though. I should start at the beginning.

What is a Service?
------------------
<img src="/img/automator/services-menu-populated.png" alt="Populated Services Menu" class="left post-image"> 
A Service in OS X is a function or workflow you can access globally from any application and which interacts with that application in specific ways.

Notice the screenshot here. These Services show up in Chrome on my laptop when I have some text selected. The text you have selected is passed along 
to the Service which processes it, passes it along to some other application, or perhaps uses it in a web request. Really, a Service can be created 
to do anything you want.

Services are not all Automator can make, though. _Workflows_ are a series of actions that you run from within Automator itself. In fact, Services can be
thought of as app-sepecific workflows. _Applications_ are similar to Workflows, but are self-contained and can be run by opening them just like any other 
application. _Folder Actions_ are Workflows that are attached to folders on the filesystem and triggered when the contents of the folder are changed. There
are other things to make as well: the full range of Automator products can be seen by simply starting Automator.app itself.

I decided to delve into Automator and create a Service for myself after about the tenth time I was stymied in my attempt to paste a simple Skype chat transcript
into an email. I won't derail into Skype-bashing here; the [lamentations](http://www.google.com/search?sourceid=chrome&ie=UTF-8&q=new+skype+sucks) of Mac Skype 
users are well documented elsewhere. In short, simply selecting the portion of the log you want to share and copying it gives you this jumbled mess:

<center><img src="/img/automator/skype-log-awful.png" alt="Awful Skype Log"></center>

Which is fairly incomprehensible. It gets much worse when trying to make sense of a long converstation between multiple parties.

Creating a Service
------------------
The first thing to do is start up Automator, of course. It will immediately assume you are trying to make something new (which you are, so that's nice)
and will ask you which new thing you are trying to make. Select Service (the gear icon) and hit the Choose button. You will be presented with a blank Automator
workflow window. To the left is the action library. You can drag actions from here to the right, into the main workspace editor area. The bottom of this area
is your debug log, and the top is a bar that selects your inputs and sources.

<center><img src="/img/automator/automator-service-selects.png" style="border: 1px solid black"></center>

You can see that I've selected to recieve **text** from **Skype**. You can select any single application, or opt to show this Service on all applications, but the
workflow I'm writing will expect text to be formatted as shown above (ie. from a Skype transcript) so I restrict the Service to only Skype.

After I looked over some of the text actions, I realized that none of them would do what I needed. What I needed was essentially a text mill -- something that is
trivial to write in, say, Ruby. Thankfully, Automator lets you do just that. I found the **Run Shell Script** action in the Utilities Library, and after some trial
and error I realized that I could not simply write `ruby -ne 'puts $_ unless...'`. I discovered that I could change the **Shell** dropdown to `/usr/bin/ruby`, which
is much better as it lets you write out an entire script.

{% highlight ruby %}
  out = ''

  STDIN.each_line do |line| 
    out << "\n" if line =~ /^[\D]+.*[\d]+:[\d]+\s[AM|PM]/
    out << line unless line =~ /^[\d]+:[\d]+\s[AM|PM]/
  end

  puts out.strip
{% endhighlight %}

Probably not the best Ruby or regexes ever written, but they get the job done. Then it's a simple matter of dragging over the **Copy to Clipboard** action
underneath the shell script, saving, and done! You can see the final state of the Automator window [here](/img/automator/automator-window-final.png), not
embedded in the post as it's rather large.

Now I can try it out. Go to Skype, select the same chat transcript as I did before, go up to the Services menu and hit my new **Copy Chat Log** service and...
Much better!

<center><img src="/img/automator/skype-log-clean.png" alt="Clean Skype Log"></center>

Keyboard Shortcuts
------------------
So it's awesome that now we can copy and paste a chat log without having to format it manually. That will save me a ton of time in the long run, but that is a lot
of menu to mouse through to do it. Keyboard shortcuts are a programmer's best friend, and Apple has us covered here too.

Open up the Keyboard System Prefrences pane. Go to the Keyboard Shortcuts tab, and select Application Shortcuts from the list on the left. Add a new shortcut with
the plus button, which will make an options sheet come down. Select Skype from the Application dropdown, type in "Copy Chat Log" in the Menu Title field, and select
a keyboard shortcut combination -- I chose `command+option+c` here.

And that's it. This is just scratching the surface of the tasks that you can automate with this tool. One of the big "promises" of Automator is its
integration with the Cocoa API. Application devs can create and expose custom actions in their apps that one could take advantage of using Automator Workflows or 
Services. Unfortunately, not many app developers take the time to implement these features, but there is still a ton of useful functionality you could wring out
of just the stock Automator actions.

