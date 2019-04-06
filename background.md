From Issues

I originally started down the path of creating a UI framework like React. But as I used the REPL more and more, I started thinking about how it could be leveraged to do other things, and I went off in a different direction ...

I ended up creating a tool instead. A tool that allows you to create a UI in real-time, by entering commands into the REPL, and which you can then export as a standalone app. I think this last part is pretty exciting, and I'll go into it more at the end.

To run the demo you will need to use Firefox or Chrome and have the standard browser flags on for WASM and PTHREADS. It does not work with Emterpreter yet, and I'm not sure I want to spend the time figuring that out either. It tested it once, and the performance was pretty bad.

If you can, I highly suggest you use Firefox for this demo. The tool will work in Chrome, but if you use it, there is a workaround you will need to do at the end, when you run the exported app.

I also recommend you copy-paste the commands I will be providing. The tool has only been tested to work for this demo. There is probably a ton of bugs, I still haven't found, that may cause the app to freeze up if you type out something wrong.

Start by running the REPL located at http://rebol.brianotto.com/ui-builder/

Load up the UI Builder by entering this ...

`run 'ui-builder`

To give you a little background, I started this project before loading apps from a remote repository was fully working. So this was just my way of having a standard command to run an app. The run command assumes all apps are stored under the "app" sub-directory and are named after the literal word you used. It also assumes all apps have an index.reb that is used to launch them. So what that command above does, is it fetches the source code at %app/ui-builder/index.reb and runs a "do" on it.

Another modification I made was to allow an app to take over the REPL's command processing by registering it's own DSL. I thought it would be pretty powerful if an app had its own language that you could use with it. I didn't realize this feature already existed :)

How I implemented it was by declaring a global variable in the *gui.js* JavaScript.

`var dsl_parser = null`  

When an app started up it would assign a function to this variable ...

`window.dsl_parser = ui-parse`

... and if a function existed, the REPL would wrap all commands with it.

    if (dsl_parser !== null) {
        text = dsl_parser + " {" + text + "}"
    }

This is why, when you start the ui-builder, you are able to enter commands that would normally give you an error in the language. My app parses the commands and runs the proper Rebol code for them.

So let's build a simple video player ...

First we want to add a row to place our video in.

`add row`

This will create a &lt;div&gt; that acts as our first row. Since rows are normally not visible in our layout, the app adds a dotted border around it so that you can see what you've added to the canvas. The app also gives it an ID, so that you don't need to think about names while your building the UI. This can always be changed later or it can be specified by providing a parameter to the row, e.g. `add row myId`

Now let's add a second row for our video's controls.

`add row`

Oops! You can't add a row inside of another row until you've added a column, and so an error is displayed. Also, we want the row to be below this one, not inside it. Any time you add an element to the page it will automatically get selected, and so what we need to do is select the top level canvas first.

`get canvas`
`add row`

Now before we can add any content to it, we will need to add a column. However, we want to think about where the controls should be displayed. For our purposes, let's make the controls centered on the screen. To do this, we need to add 3 columns and put the controls in the center column.

Remember how elements automatically gets selected when they are added to the page? We want to prevent this from happening this time, and you can do this by providing the /more "refinement". Any time you provide this, you are telling the app that you need to add more elements to the same parent. This allows you to quickly add a group of elements to a parent.

`add col/more`
`add col/more`
`add col`

We want to add the buttons in the middle column and so we select it by id.

`get col2`

Let's add a button to Stop the video now.

`add button`

Let's change the text.

`add text Stop`

This is an `add` command, not a `set` command, because we are adding an actual HTML element to the page (i.e. &lt;span&gt;) and not just setting an attribute or style.

Let's add an icon. The names come from the [Font Awesome](https://fontawesome.com/icons?d=gallery&s=brands,solid&m=free) class names.

`add icon stop`

Let's add a Play button now.

`get col2`
`add button`
`add text Play`
`add icon play`

Those buttons are too close together so let's move the Play button over a bit. Also, as you might notice, the button is still selected, even though we added a text and icon element to it. This is because those elements can't have any children and so the parent stays selected.

`set margin-left 10px`

Let's center those buttons too.

`get col2`
`set text-align center`

Let's add the video.

`get row1`
`add col`
`add video http://www.youtube.com/embed/cSp1dM2Vj48`

Doh, wait, our controls are not hooked up yet! I really wanted to use this [video](https://www.youtube.com/watch?v=dQw4w9WgXcQ) for the demo, because then you would be forced to watch it, with no way of stopping it, for a while :D ... but unfortunately YouTube does not allow you to play certain videos when a HTML page is launched from file:/// (which we will touch on later), and so I had to choose a movie Trailer instead. They appear to have less restrictions in place. I still need to look into why.

Anyway, let's hook up this player to some Rebol code!

`get button1`
`set onclick ui-video-stop`
`get button2`
`set onclick ui-video-play`

Try it out. You can now Stop and Play the video. The `ui-video-stop` and  `ui-video-play` values we entered are the names of functions I wrote in Rebol. These functions are pretty simple and just run some JavaScript code to control the video, but they don't have to be this way! You could use them to process some business logic in Rebol and then update the UI with your results.

Also, I have hard-coded these functions as part of the app code, for the purposes of this demo, but they don't have to be this way either ...

I plan on eventually adding a button that will let you open up a mini Rebol console, very similar to how replpad starts. It will let you play around with code until you've built what you need and then you can save it to your browser's local storage. This can then pushed through the console and be available as functions for your app!

Also, another alternative is we could provide a button that opens a list of 3rd party APIs. Do you need a full-featured video API for your app? Click here to download it into your local storage!

This is not only a rapid prototyping tool, but a way to create full featured apps. Well, at least full featured within the security context of a browser.

Okay, so now you have a working video player. Let's hide those layout borders, and it should look something like this.

`hide layout`

![ui-builder-screen](https://user-images.githubusercontent.com/3478333/54901652-5f269080-4e94-11e9-9dd5-67d3fa733224.png)

Additionally, the app has commands for a few other things I didn't cover. You can move elements around to different parents, hide/show the ids of all elements on the page (in case you've forgotten what they're named), and eventually the Styles and Classes button will open a sidebar that will let you manually update all the styles at once, or add custom classes.

Hopefully you've made it this far, because now things get interesting ...

The final command I will cover is `export`. It allows you to export your design into a HTML file that embeds the Rebol interpreter **with no outside dependencies**. None whatsoever, there is not even any external JavaScript files required. You can distribute your Rebol-powered app in a single HTML file!

If your default browser is Firefox then try it out. Run the `export` command and save the file to your computer. Then double click it and the app will open in your browser and display the same as before, and the controls, which use Rebol code, will continue to work.

If your default browser is Chrome then you will need to give it access to local files by passing in this flag during startup `--allow-file-access-from-files`. On Windows, you do it like this.

`start "" chrome --allow-file-access-from-files`

So, the Chrome thing sucks. I wish the default access allowed this. However, there may be a way around this, but let me explain how I got this to work in the first place ...

At a high level, what I'm doing is base64 encoding all the JavaScript required to get the REPL running and writing this to JavaScript variables, in the HTML file, on export. When the exported HTML file runs, it saves these "files" to an [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) storage location, which is accessible to web workers. I then modified the web worker slightly to load this JavaScript from storage, instead of directing importing the scripts... and volia, an embedded Rebol interpreter running on JavaScript blobs.

The problem with Chrome is that is sees the URI to these blobs as blob:file:///MYDATA, whereas Firefox sees them as blob:nul/MYDATA. So these aren't local files according to Firefox, but they are to Chrome and so it locks down access. I *think* there may be a way around this. This local access only becomes an issue when I push these blobs into a dynamically created script tag. I might be able to write out the JavaScript directly, instead of doing this, and get around this problem. But we'll see, it needs some research ...

Well, that's it for now. I was going to post my thoughts on use cases, next steps and some other experiments we could try, but this post got a little long and I have a busy day today. So I will follow up with that tomorrow.

Cheers!

Brian Otto
