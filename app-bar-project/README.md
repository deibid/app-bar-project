
Ok. So the idea is feasible.

I can manually make Automator Quick Actions and assign them to a specific keyboard shortcut from System Preferences.

But, how can I give the functionality to change the desired apps to be open?

The way I see it, I have to declare 6 template Quick Actions (QAs), bind them to a specific keyboard shortcut in the System Prefs app and simply make a utility MacOS app that changes the QAs .plist file.

I figured out a way of doing this manually. First, find the /Libraries/Services/ folder and open the QAs in a text editor. There are 2 files and a folder there.

First, find the "info.plist" file and change the app you want to open:

Under 
> <dict><array><dict><dict><string>.....

> <string> Action Name .. (i.e Open Spotify)</string>


Then, open the document.wflow file and find line #55.

> <string> /Applications/"name of the app.app"</string>


By doing this, the app that gets opened changes and there is no need to reconfigure anything else. The keyboard commands stay the same, and everything works ok.


The obvious question is, what is the component that will write changes to this file? Do I need to make a MacOS app to do this? How crazy is this?

How will I bind the external app bar to trigger the file changes?

Is there an Automator QA to write files? Hmm...

I also need a way of visualizing which apps are mapped in the bar. Does my MacOS app show that? Should I draw an overlay onscreen? Is it better to tie it to the notification center?

How about one of those mini apps that stick to the menu bar at the top?

I need to test this as soon as possible. I'll start with a proof-of-concept MacOS menu app. I have no experience developing for MacOS or Swift at all, so let's hope that my days as a freelance Android developer take me where I need to.

I'll follow this tutorial to learn how to make a simple menu app for macOS.
(https://www.raywenderlich.com/450-menus-and-popovers-in-menu-bar-apps-for-macos)


After some time trying to figure out how to make this app, it is evident that its not the best way to go for me. I want to be fast doing this, and while it'd be nice to learn Swift, I'm gonna switch to Electron and code it in Javascript, something I'm more confortable in..


The docs are great. After a few minutes I have a window with the size and position I want. THis is pretty great...

Now, how do you get a list of all the installed apps???

I can run 
> $ ls /Applications 
to get it, but that is in the terminal. Do I run this from the Electrons app? How do I get it back as a list?????

[This link](https://stackoverflow.com/questions/44719196/using-electron-node-js-how-can-i-detect-all-installed-browsers-on-macos) tells me to use a child_process. What is thaaaat????

A Node child_process is simply a module that lets you run bash commands. It exposes a few cool event methods to handle data as it comes.

I tried calling the $ ls /Applications command from it but it didn't work for some reason. i'm sure that there is a simple fix but I simply couldn't find it.

I solved this by creating a list.sh file that contained "ls /Applications" and I ran that shell file from the child process. Works like a charm.

I had to specify the econding to utf-8. That returned a long string containing all the app names. After that, I split that string with the \n separator and stored everything into an array, that way I have an array with individual app names that I can use. This is fun.

Now, for another proof-of-concept test, I'll add a button to my window that updates the Automator files that I described earlier.


Quickly learned that the main process is different than the window views. I have to find a way of communicating between them so I can pass the list of applications to the dropdown.

There is something called ipcRenderer and ipcMain. These allow you to communicate between a Main process and a Renderer process.

I didn't use them at the end, but ended up using a sharedGlobal object.

:insert_pic

Ok. I can get my installed apps from the front end. That's huge.
Lets make a button say 'hi' next.

Done.

Next, dropdown with application list.
Done. Very excited and moved, actually.

Getting picky here but.... could I get the app icon somehow? Lets spend max 10 minutes on this.

After exploration, I see that app icons are stored in the app/Content/Resources directory. The thing is, not all icons are named the same. I'd have to make another shell script that traversed all app folders and extracted all icon files by looking for the .icns extension. 
Or, loading the info.plist file for every app and extract the icon name, pull that from the resources folder into the project folder and link those assets to the app front end.

This would be SO nice design wise, but it's going to consume a lot of time that I don't have and doesn't add any functionality. So I'll leave that for later :(


Next test, writing a file to the Service folder. I have to admit I'm very nervous because everything has te be possible. If its not, nothing can work. Lets keep going

Im going to change the first keyboard shortcut from Spotify to something else.

I need to be able to call a method that lives in the main process from a rendered view. Tried using the globalObject but seems to be having issues when passing arguments to it. Will include ipcMain Module for this.

I am successfully passing the selected app back to the main process. The next step is to update the file tied to the shortcut to see if I cant substitute the app to open.

I cant seem to be able to read files. I tried a fix that I saw in a stackoverflow answer and my whole project stoped running..
Ill create a new one and port all the files


I can read the file from the Node app. Lets try and change it....

Everything works great. As long as you dont change the file name (the info.plist), the apps can get remapped.
After I tested the functionality, I tweaked the css a bit to make it nicer.

Also, added the keyboard shortcut to show and hide the electron window. That same command will be used by the secret sensor we'll add.

Whenever you close the app the select boxes get filled with fake data but the underlying shortcuts stay valid. This makes sense since the in-memory sample data gets reset when the app closes. To fix this, I added a NeDB node database where I write an object with the app settings. 

:insert_pic

There is an issue with the Calculator app. Since it doesn't expose any services, the Automator keyboard commands dont get triggered. However, the Electron commands do. So.. maybe I'll bind the app-bar to electron and call the commands from there.

>usr/bin/automator path/to/workflow 

does the trick.


Very funy... by adding this change, I can now change the file name of the short-cut, and its much quicker to jump between apps
Before wrapping up the electron-app, lets get rid of the .app prefix on the button.



































