<h1 align="center">Painfully Learned Lessons</h1>

<p align="center"><i>HTML5 is so janky that it gets its own page.</i></p>

&nbsp;

&nbsp;

1. Download open-sourced HTML5 runtime from https://github.com/YoYoGames/GameMaker-HTML5. This will make runtime code not obfuscated and give actually meaningful stacktraces in case of exceptions. You can choose exact versions of runtime if needed by cloning the repo and checking-out exact commit, they are sometimes tagged with runtime version.

2. Set the open-source runtime in "Preferences > Platform Settings > HTML5". Paste the path to "scripts" subfolder to "Path to HTML5 Runner"

3. Here you need to also make sure that Obfuscate is un-checked (will make your game code not-obfuscated too), and Pretty Print and Verbose is checked (will generate more readable code and more meaningful console output).

4. After running the project click "More Tools > Developer Tools" or press Ctrl+Shift+I, this will open debug dev tools (at least on Chrome, may be different on other browsers).

5. On the "Console" tab will be your console output and error messages.

6. On the "Sources" tab you may want to enable "Pause on uncaught/caught exceptions" on the right panel for automatic breakpoints on errors. That may lead to to pausing on runner inner exceptions which you can ignore sometimes as they are not fatal.

7. "Watch" panel above is the same as "Watches" in GameMaker debugger but you'll need to write JavaScript code instead with relevant HTML5 variable names.

8. On the left panel you will see "html5game" and "scripts" folder. "scripts" is the GameMaker inner runtime code, "html5game > GameName.js?cachebust=<numbers>" is your game's code. You can open these js files and place breakpoints by pressing on a line number.

9. Breakpoints will be reflected on a "Breakpoints" panel on the right (where pauses on exceptions were), where you can disable them.

10. After the game will pause on Breakpoint "Scope" and "Call Stack" will populate with data. "Script" in "Scope" will show you currently visible in scope variables, like "Locals" in GameMaker debugger, "Global" is like "Globals" in GM. "Call Stack" is pretty the same as in GM, you can jump on the callstack by pressing stack entries.

11. You can continue, step in, step out, etc. on a breakpoint with the control panel on the top right.

12. Hovering the variables on in the source code and in the console will popup its inner fields for complex types like arrays and objects/structs.

13. You can execute JavaScript code by writing it directly to Console. May be useful for getting variables or running some functions.
