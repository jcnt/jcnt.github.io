---
layout: post
title:  "Keep app window sizes with multiple monitors on Mac OS X"
date:   2015-02-08 19:11:42+0100
categories: 
tags: 
- Apple
- Mac
- OSX
---
I’m using my Mac with different screen sizes (internal and external monitor). I’ve been looking for a solution for quite some time now, especially since the difference between the external and the internal is growing. Currently I run a 11” MacBook Air with a Full HD external screen. The only thing I want is to have those apps I run all the time appear on the same place when I switch the output. 

Obviously one option is to use a specific software for this task, e.g. “Stay”. However, it’s absolutely possible with the builtin tools and a few easy steps. 

<!--more-->

**1./ Get the app window size and location**

In Apple Script this action is called ‘get bounds’. Basically you need to run the following short script to get the size and location: 

    try
        tell application “Safari”
            activate
            get the bounds of the front window
        end tell
    end try

You can use the “Script Editor” for this task. The app will get focus and your apple script editor will receive its bounds. e.g.: {18, 35, 1344, 751}

**2./ Create a service in Automator**

Start automator, create new and select service. Once it’s opened I changed the text as “no input”. This makes the newly created service show up in any application’s “Services”menu. Try to create a unique name for your script, it will help at the end to set up your hotkey. 

**3./ Create the script for the service**

Now you can use the numbers from the first step to move the apps into the specific location. 

    on run {input, parameters}
    
        try
            tell application “Safari”
                activate
                set the bounds of the first window to {18, 35, 1344, 751}
            end tell
        end try

When you run this service Safari will be resized based on the input. There are other tricks, like minimising windows: 

    tell application “System Events” to tell “Safari” to set visible to false

Apple Script is quite easy to read and write. 

**4./ Setup a hotkey for your service**

Last one is to create a hotkey. If you created the service as ‘no input’ and in “any application” then you’ll be able to use the hotkey in anywhere the system. Setting up the hotkey is the same as for any apps, you’ll find your script under “services”.


