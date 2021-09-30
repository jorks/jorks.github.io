---
title:	Introducing JamfLog for Sublime Text
date:	2021-09-30 12:03:00 +1000
excerpt_separator: "<!--more-->"
tags:
  - Jamf
  - Sublime Text
---
Logs suck! Actually they're great, what I really mean is they're not the easiest to read and often most useful when the pressure is on. 

**[JamfLog](http://github.com/jorks/sublime-jamflog){:target="_blank"}** is a [Sublime Text](http://www.sublimetext.com/){:target="_blank"} syntax highlighter for [Jamf Pro](http://jamf.com){:target="_blank"} server logs. It makes Jamf Logs easier for humans.

<!--more-->

![no-alignment]({{ site.url }}{{ site.baseurl }}/assets/posts/2021-08-jamflog/JamfLog.gif )

## How to install Jamf Log

JamfLog is available via Sublime Text’s built-in _App Store_ aka [Package Control](https://packagecontrol.io/){:target="_blank"}.

**Video Instructions:** 📺 Watch [JamfLog for Sublime Text - Install Instructions and Usage](https://www.youtube.com/watch?v=iqHyu3vG48w){:target="_blank"} on YouTube. 
{: .notice--info}

1. If you have never used package control, in the menu go to Tools > Command Palette and search `Install Package Control`
2. Open the Tools > Command Palette and search `Package Control: Install Package`
3. Search for `JamfLog` and install the latest version. 
4. Activate the default settings. Open the Tools > Command Palette and search `Preferences: JamfLog Settings`
5. The prefilled default settings file will open - **SAVE THIS FILE** - Done!

## Usage Tips

Sublime Text's Command Palette is your best friend - familiarise yourself with the keyboard shortcut: 

- Mac: `Cmd+Shift+P`
- Win: `Ctrl+Shift+P`

Open a `.log` file and use the Command Palette to:

- `Set Syntax: JamfLog`
- `Word Wrap: Toggle`
- `Code Folding: Fold/Unfold All` (hides all the stack traces)

## How did this come about

Do you know when you look a little too deep into something and end up falling in? Well, that's how this project started. I was looking at another Sublime Text log package and thought, I could edit this for Jamf Logs. Turns out I couldn't, but I was too invested and decided to start from scratch. After a lot of trial and error, here we are. 

Where to now? Sublime Text has a powerful Python-based API. I would be open to expanding this package to manipulate the log files and add commands (jump forward one hour?).

Feel free to get in touch if you would like help customising something or want to get involved. 

If you encounter a problem with JamfLog, please submit an Issue on GitHub.