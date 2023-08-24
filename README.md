<h1 align="center">Painfully Learned Lessons</h1>

<p align="center"><i>A straight-forward guide to mitigating the various harms committed upon society by YoYoGames' masterwork of digital torture, <b>GameMaker</b>.</i></p>

<p align="center"><i>PRs welcome. Share your pain.</i></p>

&nbsp;

&nbsp;

## Stability

### The game is crashing to desktop without an error message.

Try running the game in YYC to generate a C++ project, and then run the generated solution file through Visual Studio's debugger. This'll hopefully point you in the right direction. Even if you can't solve the problem, it'll make for a more informative bug report to YYG.

&nbsp;

### Man, HTML5 is cursed.

Sure is. Strap on the proton pack and take a look at [some tips on busting HTML5 ghosts](https://github.com/JujuAdams/painfully-learned-lessons/blob/main/HTML5.md).

&nbsp;

### I have an active subscription and/or a permanent license. Why don't I see any export options?

To see your exports, you need to be logged into your Opera account. If you're logged in, but still don't see them, log out and log back in via the blue account icon in the top-right of the GameMaker IDE. This is a bug and will hopefully be fixed soon (issue has been present since version 2022.2.0.71, 2023-03-03).

&nbsp;

### My gamepad doesn't work.

Usually a result of GameMaker missing configuration data for that particular make and model. Might be solved by using [Input](https://github.com/jujuadams/input) or by adding an updated [`gamecontrollerdb.txt`](https://github.com/JujuAdams/Input/blob/master/datafiles/sdl2.txt) to your project's Included Files.

&nbsp;

### My game was running fine on YYC, but I changed one line and now my game doesn't compile.

The YYC module should detect changed files and only recompile those needed but it occasionally loses track, especially after adding scripts and objects. To solve this, hit the Clean Cache button (the little brush next to the debug/play/stop buttons in the IDE) to force GameMaker to recompile from scratch. You can also delete specific files from the cache to force a smaller and faster partial recompile but this is often more trouble than it's worth.

&nbsp;

&nbsp;

## Performance

### I am getting performance issues on Windows 11 and newer versions of Windows 10.
Also often caused by a broken/missing/outdated Visual C++ redistributable. [Try installing this](https://www.microsoft.com/en-us/download/details.aspx?id=30679) (Visual C++ Redistributable for Visual Studio 2012 Update 4).

&nbsp;

### My `fps_real` is high but `fps` is low and performance is bad.
`fps_real` only measures what you're doing on the CPU. You're likely GPU-bound. Turn off graphical effects, turn off shaders/effect layers, reduce particle counts, reduce rendering resolution, draw less to the screen. Use RenderDoc to identify particular pain points.

&nbsp;

### `<insert library name here>` is taking up 50% of a Step in the profiler and I am very worried about it.
The GameMaker profiler is deceptive and the percentage measurement is nigh useless due to the way it's calculated. The bit that's actually important is the time taken to execute the function. If the execution time is less than 1ms then you're getting your knickers in a twist about nothing.

&nbsp;

### I added an object and my `fps_real` went from 5000 down to 4000. Help!

Don't worry about it. `fps_real` is deceptively useless and the only thing it's really good for is making people panic needlessly.

Frame rate is [inversely proportional to frame time](https://www.desmos.com/calculator/d4hvus9oys), which means that (among other things) minute differences in very small frame times will have seemingly disproportionately large effects on the `fps_real`. What you really care about is the **frame time,** or the amount of time it takes to deliver a frame. This means you have a budget of about 16.6 ms to do all of your Step and Draw event processing if you want to maintain a refresh rate of 60 frames per second. An `fps_real` value of 5000 means that your entire game is finished with its update tick in **0.2 milliseconds,** and an `fps_real` value of 4000 means that your entire game is finished with its update tick in **0.25 milliseconds.** That's a difference of **0.05 milliseconds,** which profesional game developers like to call "a vanishingly tiny proportion of your frame time budget" (it's about 0.3%).

Whatever you did to make your `fps_real` dip, you can do it another 300 times and still hit a 60FPS target.

&nbsp;

### GameMaker is lagging whenever I add instances to a room.

Turn off Feather (at least for the duration that you're doing lots of heavy room editing).

&nbsp;

### I'm using GameMaker Studio 1 and my game has performance issues I can't figure out.

Try using [gmsched](https://github.com/skyfloogle/gmsched).

&nbsp;

&nbsp;

## Graphics

### I am getting a shader compilation failure in GameMaker 2023 / 2022 / GameMaker Studio 2.
Often caused by a broken/missing/outdated Visual C++ redistributable. [Try installing this](https://www.microsoft.com/en-us/download/details.aspx?id=30679) (Visual C++ Redistributable for Visual Studio 2012 Update 4).

&nbsp;

### I am getting a shader compilation failure in GameMaker Studio 1.
Often caused by a broken/missing DirectX 9c runtime. [Try installing this](https://www.microsoft.com/en-gb/download/details.aspx?id=8109) (DirectX 9c via DirectX End-User Runtimes (June 2010)).

&nbsp;

### On consoles (especially Xbox) my rectangles and other primitives are offset for some reason.

This is a known issue and maybe it'll get fixed some time. In the meanwhile, you'll need to manually adjust the rendering position of primitives yourself. On Xbox, for example, the top-left corner of `draw_rectangle()` will need to be moved right 1 pixel and down 1 pixel. This impacts many primitive-drawing functions cross-platform - happy hunting!

&nbsp;

### On Nintendo Switch the colour channel order looks off when I'm using vertex buffer functions, shouldn't it be the same as other OpenGL platforms like Linux?

It should be but it isn't. On Switch, the colour order is the same as Windows even though the graphics backend is OpenGL. (When using [Scribble](https://github.com/jujuadams/scribble) you might want to edit the `__SCRIBBLE_FIX_ARGB` macro to cover `os_switch` too.)

&nbsp;

### My font sizes in-game are not the same as in the graphics mockup.

If you've created a mockup in Photoshop, Figma, GIMP, etc. and then tried to replicate it in GameMaker, you may notice that the font in GameMaker is rendered somewhat bigger than in the mockup despite font point size being the same. This is happening because of inconsistent DPI settings: image editors often have their default DPI at 72 while GameMaker's is 96. To fix this issue you, multiply the image editor's font values by 0.75 (so 48pt in Photoshop becomes 36pt in GameMaker Font Editor) or change the DPI in the image editor to 96 pixels per inch (without resampling).

&nbsp;

### What is GameMaker's GLSL ES version?
1.00 rev 17. Older versions of GameMaker have patchy support for standard derivatives, and no current (GMS2023.6 and before) versions of GameMaker natively support vertex texture fetching outside of HTML5.

&nbsp;

### Nothing is drawing and I see "Draw failed due to invalid input layout" in the Output log.
This happens when something you are drawing does not provide the attribute data that your shader needs. For example, if you apply a shader that expects texture coordinates, onto a draw_rectangle, it will fail. A common workaround is to have different versions for you shaders depending on what you're drawing (e.g. shd_basic, shd_textured, shd_textured_normals, etc)
