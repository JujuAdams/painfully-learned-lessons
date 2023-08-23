<h1 align="center">Painfully Learned Lessons</h1>

<p align="center"><i>A straight-forward guide to mitigate the various harms committed upon society by YoYoGames' masterwork of digital torture, <b>GameMaker</b>.</i></p>

<p align="center"><i>PRs welcome. Share your pain.</i></p>

&nbsp;

&nbsp;

### I am getting a shader compilation failure in GameMaker 2023 / 2022 / GameMaker Studio 2.
https://www.microsoft.com/en-us/download/details.aspx?id=30679

&nbsp;

### I am getting performance issues on Windows 11 and newer versions of Windows 10.
https://www.microsoft.com/en-us/download/details.aspx?id=30679

&nbsp;

### The game is crashing to desktop without an error message.

Try running the game in YYC to generate a C++ project, and then run the generated solution file through Visual Studio's debugger. This'll hopefully point you in the right direction. Even if you can't solve the problem, it'll make for a more informative bug report to YYG.

&nbsp;

### My `fps_real` is high but `fps` is low and performance is bad.
`fps_real` only measures what you're doing on the CPU. You're likely GPU-bound. Turn off graphical effects, reduce rendering resolution, draw less to the screen. Use RenderDoc to identify particular pain points.

&nbsp;

### `<insert library name here>` is taking up 50% of a Step in the profiler and I am very worried about it.
The GameMaker profiler is deceptive and the percentage measurement is nigh useless due to the way it's calculated. The bit that's actually important is the time taken to execute the function. If the execution time is less than 1ms then you're getting your knickers in a twist about nothing.

&nbsp;

### I added an object and my `fps_real` went from 5,000 down to 4,000. Help!

Don't worry about it. `fps_real` is deceptively useless and the only thing it's really good for is making people panic needlessly.

Frame rate is [inversely proportional to frame time](https://www.desmos.com/calculator/d4hvus9oys), which means that (among other things) minute differences in very small frame times will have seemingly disproportionately large effects on the `fps_real`. What you really care about is the **frame time,** or the amount of time it takes to deliver a frame. This means you have a budget of about 16.6 ms to do all of your Step and Draw event processing if you want to maintain a refresh rate of 60 frames per second. An `fps_real` value of 5000 means that your entire game is finished with its update tick in **0.2 milliseconds,** and an `fps_real` value of 4000 means that your entire game is finished with its update tick in **0.25 milliseconds.** That's a difference of **0.05 milliseconds,** which profesional game developers like to call "a vanishingly tiny proportion of your frame time budget" (it's about 0.3%).

Whatever you did to make your `fps_real` dip, you can do it another 300 times and still hit a 60FPS target.

&nbsp;

### GameMaker is lagging whenever I add instances to a room.

Turn off Feather (at least for the duration that you're doing lots of heavy room editing).

&nbsp;

### What is GameMaker's GLSL ES version?
1.00 rev 17. Older versions of GameMaker have patchy support for standard derivatives, and no current (GMS2023.6 and before) versions of GameMaker natively support vertex texture fetching outside of HTML5.

&nbsp;

### My gamepad doesn't work.

Usually a result of GameMaker missing configuration data for that particular make and model. Might be solved by using [Input](https://github.com/jujuadams/input) or by adding an updated [`gamecontrollerdb.txt`](https://github.com/JujuAdams/Input/blob/master/datafiles/sdl2.txt) to your project's Included Files.

&nbsp;

### I am getting a shader compilation failure in GameMaker Studio 1.
https://www.microsoft.com/en-gb/download/details.aspx?id=8109

&nbsp;

### I'm using GameMaker Studio 1 and my game has performance issues I can't figure out.

Try using [gmsched](https://github.com/skyfloogle/gmsched).

&nbsp;

### On consoles (especially Xbox) my rectangles and other primitives are offset for some reason.

Known issue, use this compatibility script (expand the `os_type` if case to other affected consoles yourself):

```gml
function draw_rectangle_color_f(x1, y1, x2, y2, col1, col2, col3, col4, outline) {
    if (os_type == os_xboxseriesxs) {
        // d3d12 bug
        ++x2;
        ++y2;
    }
    
    draw_rectangle_color(x1, y1, x2, y2, col1, col2, col3, col4, outline);
}

function draw_point_color_f(x1, y1, col1) {
    if (os_type == os_xboxseriesxs) {
        // d3d12 bug
        ++x1;
        ++y1;
    }
    
    draw_point_color(x1, y1, col1);
}

// implement other functions in the same fashion...
```

&nbsp;

### On Nintendo Switch the color channel order looks off, shouldn't it be the same as Linux?

Known issue, it should be, but it isn't. On the Switch it's the same as Windows even though the graphics backend is OpenGL.

(when using Scribble you might want to edit the `__SCRIBBLE_FIX_ARGB` macro to cover `os_switch` too)
