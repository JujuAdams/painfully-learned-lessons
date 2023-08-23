<h1 align="center">Painfully Learned Lessons</h1>

<p align="center"><i>A straight-forward guide to mitigate the various harms committed upon society by YoYoGames' masterwork of digital torture, <b>GameMaker</b></i></p>

&nbsp;

### I am getting a shader compilation failure in GMS2.
https://www.microsoft.com/en-us/download/details.aspx?id=30679

&nbsp;

### I am getting a shader compilation failure in GameMaker Studio 1.
https://www.microsoft.com/en-gb/download/details.aspx?id=8109

&nbsp;

### I am getting performance issues on Windows 11 and newer versions of Windows 10.
https://www.microsoft.com/en-us/download/details.aspx?id=30679

&nbsp;

### `fps_real` is high but `fps` is low (performance is bad):
You're GPU-bound. Turn off graphical effects, reduce rendering resolution, draw less to the screen. Use RenderDoc to identify particular pain points.

&nbsp;

### <library name> is taking up 50% of a Step in the profiler.
Look at the actual time taken to execute the function. If it's less than 1ms then you're getting your knickers in a twist about nothing.

&nbsp;

### GameMaker is lagging whenever I add instances to a room.

Turn off Feather (at least for the duration that you're doing lots of heavy room editing).

&nbsp;

### What is GameMaker's GLSL ES version?
1.00 rev 17. Older versions of GameMaker have patchy support for standard derivatives, and no current (GMS2023.6 and before) versions of GameMaker natively support vertex texture fetching outside of HTML5.
