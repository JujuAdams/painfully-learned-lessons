## I am getting a shader compilation failure in GMS2.
https://www.microsoft.com/en-us/download/details.aspx?id=30679#

## I am getting a shader compilation failure in GMS1.
https://www.microsoft.com/en-gb/download/details.aspx?id=8109

## I am getting performance issues on Windows 11 and newer versions of Windows 10.
https://www.microsoft.com/en-us/download/details.aspx?id=30679#

## `fps_real` is high but `fps` is low (performance is bad):
You're GPU-bound. Turn off graphical effects, reduce rendering resolution, draw less to the screen. Use RenderDoc to identify particular pain points.

## <library name> is taking up 50% of a Step in the profiler. WWhat do I do?
Look at the actual time taken to execute the function. If it's less than 1ms then you're getting your knickers in a twist about nothing.

## What is GameMaker's GLSL ES version?
1.00 rev 17. Older versions of GameMaker have patchy support for standard derivatives, and no current (GMS2023.6 and before) versions of GameMaker natively support vertex texture fetching outside of HTML5.
