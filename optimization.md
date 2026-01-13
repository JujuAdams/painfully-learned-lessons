There's a lot that's been said about making GameMaker games run fast over the years. Some of it's good advice, some of it was good advice a long time ago but isn't anymore, and some of it never was good advice but people keep repeating it anyway because it looks good in a tweet. Today we'll be sorting through GameMaker optimization advice using the internet's favorite organizational data structure, the tier list.

# Why should you listen to me?

Hi, I'm Michael. Depending on who you ask I'm either a community legend or a community menace. I've been around here for a long time and broken GameMaker in a lot of different ways. I've been making YouTube videos about GameMaker either since 2013 or 2019 (it's a long story), with topics including but not limited to weird 3D stuff and, uhh, optimization. Over the years, I've created a few different setups to test the performance of GameMaker games over the years. The one that's probably most widely useful is GMBenchmark, which is a little GUI that lets you test two pieces of code head-to-head to see who wins. We'll be making use of this and a few other things today.

Oh and I also made this run on a Raspberry Pi.

If the appeal to authority fallacy isn't doing it for you, I'm also going to be posting an experiment or some other citation to go with pretty much everything on this list.

# Why is GameMaker like this?

GameMaker is simultaneously slower than people think and faster than people think. It was much slower in the past, and while performance has improved over time in various ways, it's still nowhere near as fast as an equivalent application written in a lower-level language compiled natively. GameMaker's default mode of operation runs through a virtual machine, which you can think of as a generic CPU emulator built from instructions to do things like set variables, call functions, branch, and so on. There are several reasons that this is "slow" compared to a native application, with probably the biggest one being that even basic operations such as addition or multiplication that would seem atomic actually has to go through a whole virtual process of fetching, decoding, and executing each instruction in software instead of doing it natively in hardware. Remember how Nintendo DS and 3DS and PS3 emulators took like half a decade to become playable, even when running on much more powerful desktop computers? That's why.

Having said that, it's reasonable to ask why GameMaker was designed like this in the first place. There are two main benefits: building your game to VM bytecode is insanely fast, even in projects with hundreds of thousands of lines, and it (usually) makes games more portable because much of the work in ensuring compatibility between different platforms is done by the engine developer, and the user doesn't have to worry about any platform-specific compiler weirdness.

The Yoyo Compiler is an alternate GameMaker runtime which first translates your GML code into the most crusty C++ you've ever seen in your life. This speeds up execution of most GameMaker code by a reasonable amount, but does not speed up the engine internals like collision detection, built-in pathfinding, or automatic object Step events. It's important to note that this is still not as fast as an equivalent application written in native C++, mostly owing to GameMaker's dynamic type system and the fact that even things as simple as variables and numeric arithmetic carry a fair amount of overhead. On average the YYC will speed up your code by about 2-3x, but likely will still be 5-10x slower than sane, native C++. There's also at least one know case where YYC is SLOWER than VM (though it's probably not something you run into regularly). In most of the benchmark screenshots in the following post, I'll generally be showing results in YYC, mainly because people who really care about performance will probably be using that anyway. Screenshots involving the profiler will instead be using the VM, since the debug builds don't work in YYC.

Combined with the fact that GameMaker is (mostly) single-threaded, this means that GameMaker games will make the computer's CPU choke faster than its GPU in a vast majority of cases. It's certainly possible for your game to run into a GPU limit before a CPU limit, but most of the time you have to try pretty hard (or mess up pretty badly) to make that happen. There will be a few GPU things in this thread, but they're only going to be items that someone using GameMaker in a normal way can be expected to run into, rather than the stuff that weirdos like me deal with.

Lastly, everything I'm going to write here applies to the current GameMaker runtime. In the upcoming GMRT the entire runtime and build system will be rewritten from scratch. The goal is that it should be faster than Current Runtime overall, but in reality it's unknown what the performance situation is going to look like, and it's likely still a long way off. I might or might not rewrite this whole thing when the time comes, but will otherwise I'll just make occasional speculative notes about things that might change in GMRT and ignore it otherwise.

# Context

There aren't any absolute rules in optimization. There will be optimizations that are more relevant on some platforms than others, or optimizations that apply everywhere except on HTML5 because the HTML5 runner is based on JavaScript, which has a different set of constraints than the regular GameMaker runner. If you do this for long enough you'll eventually find code that looks bad but doesn't really make a difference because you only run it once during Game Start, and you'll eventually find code that doesn't look bad but still adds up because you just tried to simulate the entire observable universe at the same time (whoops). You have to sit back and think about what's happening in your specific game: if you go into your code and start chopping bits off with reckless abandon, without thinking about what you're doing, just because some weirdo on the Internet told you to, bad things are going to happen.

It's important to remember that not all optimizations are created equal. You might find one piece of code that you can make 10% faster, and one piece of code that you can make 50% faster, but the code that you make 10% faster takes 10 ms of frame time while the code that you make 50% faster only takes 0.25 ms. By all means do both if you can, but development time is almost always a limited resource and on some level you have to triage what's most important to work on: in this case, a 10% improvement on a routine that's 10 ms will actually save much more time than a 50% improvement on a routine that takes 0.25 ms.

That brings us to...

# S+ tier: Profile your game

I lied in the last section, there's one absolute rule. This isn't actually an optimization, but I'm giving it its own tier because it should always be the first thing you do if you think your game is running slowly. The profiler will actually tell you what in your game is running slowly. If you don't do this, you'll be guessing randomly.

If you do this for long enough you develop some intuition for what causes games to slow down. I can probably correctly guess how something will perform about 75% of the time, but that's still not an excuse to not look at the profiler to confirm or unconfirm. You can, and people have, waste days of your life by making assumptions instead of just checking the profiler.

If you're experiencing issues on a specific platform (namely, HTML5 or console), use the profiler that comes with that platform. If you're experiencing GPU issues on Windows or Linux, use RenderDoc.

# S tier: Optimizations that actually make a difference

I might not call these "universal," but they're pretty close. Most of them will also continue to be relevant into GMRT, as they're fundamentally part of "things that make computers slow."

## Avoid n² (or worse) algorithms

Let's start off by making this feel like homework.

Time complexity is one of those things you probably heard a lot about if you went to school for computer science, and something you probably haven't if you didn't. In most situations, there are more than one viable way to compute a result. The time complexity of an algorithm describes how many steps it takes to complete for different input sizes.

A classic example is sorting a list. If you have a list of random values, you have the option to:

    Compare each value in the list in sequence, and if they're out of order, swap them and restart from the beginning
    Break the list into smaller pieces, sort the pieces, and stitch them back together
    Randomly shuffle the list until it sorts itself

Some of these choices have a better time complexity than others. We won't spend too much time on definitions here because this is a long post and I want to sleep eventually, but you should be mindful of how quickly the amount of work the game has to do if you have to evaluate 10, or 20, or 50, or 100, or more pieces of data at a time. An example of an n² routine in a game is an object that checks some property (for example, the distance to, or how much HP or level they have compared to you, or something else) against every other object of its type in the game. If you have 4 objects there will be 12 individual checks, 10 objects will be 90 checks, 50 objects will be 2450 checks, and this will continue to scale with the square of the number of objects.

Yes, this is technically n(n-1) instead of n² operations. It's still considered an n² time complexity because that’s where the function slots into the fast-growing hierarchy.

As a side note, sorting a list isn't something you should ever have to do in GameMaker. array_sort is implemented as Quicksort. There was a point in history where it was one of the slower sorts and implementing Quick or Merge Sort yourself could yield performance gains, but this hasn't been the case for several years.

### Verdict

Rewriting expensive logic using more efficient algorithms can yield substantial performance wins, but isn't always possible. (This is actually one of the biggest unsolved problems in computer science, although most games don't really deal with NP-hard problems very often.) Most of the time you should just design your game in a way that doesn't need this stuff in the first place.

## Avoid abstractions that don't do anything

I see this in the wild more often than I'd like. Way, way, way more often than I'd like.

When GameMaker's 2.3 update came out in 2020 and it suddenly seemed like anything was possible, a lot of people - including me - spent a good deal of time doing things like "rewriting arrays or strings so that they could be used in an object-oriented way." I'm sure you've seen something like this before.

```gml
function Array() constructor {
    data = [];

    static Push = function(element) {
        array_push(data, element);
    };

    static Get = function(index) {
        return data[index];
    };

    static Delete = function(index) {
        array_delete(data, index, 1);
    }
}
```

It's been done with pretty much everything in GameMaker over the years: arrays, strings, other data structures, vector types, generic math, collisions, audio, sprites, and so on down the line.

Admittedly, it does look nice. However, it doesn't actually add any new functionality that didn't already exist in GameMaker arrays. Meanwhile, each individual function call has a very small amount of performance overhead. I've clocked the amount of time it takes to invoke an individual function call to be in the ballpark of some tens of nanoseconds (yes, nanoseconds) on my Ryzen 9 9950X, which isn't a lot: this is the amount of time it takes a photon of light to travel about ten or twenty meters. However, if you expend that additional tens of nanoseconds every single time you call an array, perhaps thousands or tens of thousands of times per step, every single time you access an array, that can add up.

Excessive abstraction is also a major part of why much software written today runs slower than software written twenty years ago for computers with a fraction of the power.

### Verdict

Abstraction is good in a lot of cases, and from a code maintenance standpoint using too few abstractions is much worse than using too many. However, like many things in life, a responsible software developer still needs to think about the abstraction they've written and decide if it actually contributes to their code in a meaningful way.

## Avoid significant and recurring memory allocations

Moving on from arrays, let's talk about arrays. In a lot of ways arrays are the most fundamental data structure that exists, and they're useful for a lot of things. As with most other things on this list, the cost of allocating a single array of elements isn't too bad, but if you do it excessively - such as in the Step event of every active instance in the game - it can start to become a lot. It's also an easy thing to not think about, since usually when people experience performance issues they look at other things first (probably because they're not using the profiler).

This is almost entirely irrelevant to GameMaker because you can't create data structures on the stack and the whole runtime is so slow it doesn't really matter anyway, but if you want to impress your friends, there's a whole rabbit hole on stack memory vs heap memory and why one is slower than the other.

There are also a few functions which return structs or arrays that you want to be careful of. Matrix functions are especially guilty of this, although matrix_multiply and other matrix functions recently gained the ability to output the results into an existing array instead of returning a new one, which is usually the way to go.

Arrays are the most common structures that run afoul of this, but it also goes for other things, like buffers, ds_whatever, and surfaces. Surfaces can be especially bad because it's easy to create large ones without thinking about it.

### Verdict

If your code does depend on a lot of array allocations, it can be difficult to get rid of them without changing functionality or introducing bugs. In some circumstances, you can ease the burden by declaring an array as a static variable so that it's only allocated once and the same array persists every time you call a function - but this can be dangerous if you're not careful, as it means you can accidentally carry over data from one call of the function to another, which can create errors that are very hard to debug.

## If you're using matrix functions, use the output arrays whenever possible.

Be careful with Blur, Bloom, and other full-screen smear-y filters/effects

This is one of the rare times I'm going to talk about GPU stuff in this thread, and the reason for it is because it's probably the only GPU mess that it's really easy to create without doing anything weird on purpose. Bloom, glow, blur, and other post-processing effects that smear out pixels are an inherently costly operation. Since these effects are computed on every pixel, they scale especially poorly at high resolutions. Remember our discussion of time complexity? The cost of post-processing effects scales with the number of pixels in the output image, and when your resolution grows on both the horizontal and vertical axis - and I assume yours does, because a 1D game would be weird - this means the time complexity is n²-ish.

GameMaker's effect layers have "quality" settings, but most of them noticeably change how the result looks in a way that people usually don't like. If you're writing a shader to do this yourself, you actually can compute the effect at a lower-than-native resolution for pretty noticeable performance improvement and a minimal hit to image quality. I put in a feature request for an option like this in the preset effect layers, but they haven't done it yet (view that thread for performance stats and visual comparisons), and it unfortunately looks like they probably won't before GMRT. Maybe if enough of you people ask nicely...

### Verdict

If you need a bloom or glow effect and this becomes a problem, it may be worth writing your own and computing it at low resolution.

# A tier: Optimizations that are sometimes useful but...

These are all good optimizations, but they won't be applicable to all types of games. Hopefully whatever you're working on can benefit from at least one or two of them.

## Deactivating instances

We've all read the Forager article by now. If you haven't, this is the Forager article. Also, how did you make it this long without running into the Forager article? That’s honestly kind of impressive, it’s like living in the United States without knowing what the Superbowl is. Anyway, there are a few things in the Forager article, and it's generally good advice. We'll get to some of the other things in it later, but for now the part that we (and most people in general) are interested in is most of the second half, where lazyeye talks about deactivating instances.

GameMaker instances are pretty fast, but if you have a lot of them their frame-time cost can add up over time. This includes not only the actual code you've written in the step and draw events, but also things like updating their built-in variables like x, y, speed, direction, friction, and gravity. Over the years, a lot of energy has been spent by a lot of people trying to mitigate this problem. Remember this - this is going to be a recurring theme in this list.

Instance deactivation circumvents this by removing instance(s) from the game loop, but keeping them around in memory. It's as if they've been deleted, but sent to the Recycle Bin where they can still be brought back later. They'll be skipped over in the Step, Draw, and other events, they won't respond to with() summons, and they'll be deleted for real if you go to another room, but while you're still in the same room they can be accessed via a reference and re-activated when needed.

A popular strategy in GameMaker is to deactivate instances that are offscreen, and re-activate the ones that you can actually see. You can do this with something that looks like

```gml
var cam = view_get_camera(0);
var left = camera_get_view_x(cam);
var top = camera_get_view_y(cam);
var width = camera_get_view_width(cam);
var height = camera_get_view_height(cam);

instance_activate_object(obj_grass);
instance_deactivate_region(left, top, width, height, false, true);
```

On the face of it, it works fairly well. Here's an example I made using this grass that sways when the player runs past it project.

The cost of deactivating/reactivating instances is more or less fixed (given a constant number of instances), while the cost of an instance's step or draw event can vary greatly. If most of the instances you're dealing with are simplistic things like these grass objects - here each individual grass object takes about 800 nanoseconds of frame time, and managing the deactivation system takes about 50 nanoseconds for each - the benefit of deactivating them won't be as interesting. If the objects you're dealing with are much more complicated, perhaps running some kind of pathfinding AI or a more involved Draw event, the deactivation system will still take about 50 nanoseconds for each but the cost of having them active will be much higher.

### Verdict

Deactivating instances is a real optimization that has been used in a lot of games, but it only works if the net cost of deactivation/reactivation is substantially lower than the net cost of the instances themselves. It works best in very large maps, where only a small fraction of the instances in the game will be on screen at once. Don't do this if your maps are all small and confined.

Options to improve this include not doing this every step, or staggering the regions that are activated/deactivated to spread out the load over multiple frames. It may only be worth doing it for instances with an expensive step event, such as someone who runs a complicated AI decision tree or pathfinding.

If you use the additional instance-tracking enhancements mentioned in the Forager article, you'll hit the threshold where deactivation is more expensive than just running the instances faster.

# Avoid large numbers of batch breaks and texture swaps

Your GPU is happiest when you give it a lot of work to do and don't interrupt it. Anything that changes the GPU state will do this: setting a shader or uniform, most functions that start with gpu_set_ except for gpu_set_depth, setting any kind of matrix, changing surface targets, and some other things will all do this.

Most of the time, the cost is pretty small. Here's the difference between drawing 20,000 sprites in one batch vs drawing 20,000 sprites in 20,000 batches.

Most of the time the cost of a batch break is pretty small and different sources have a more or less equivalent cost, but there are a few exceptions. Setting any of the rendering matrices (world, view, or projection) requires a little more math behind the scenes, and matrix_sets usually add up a little more quickly. Setting or resetting the surface target is a little more expensive. Your GML code and rendering on the GPU happen asynchronously, which means GameMaker dumps a pile of work on the GPU's desk and the GPU will get to it when it gets to it. When you change the surface target, GameMaker has to actually stop what it's doing and wait for the GPU to finish. This still doesn't take a ton of time in the grand scheme of things - it's not uncommon for GameMaker games to go through a stack of three or four different surfaces in postprocessing effects - but you should avoid using them for small, repeated tasks. If you need to clip graphics to a region on screen, for example, you can use the scissor region or stencil testing instead.

Texture swaps are a little more interesting. GameMaker puts all of your graphics on composite texture atlases, and for ideal results you would draw objects in your game grouped together based on what texture atlas they belong to instead of jumping between texture atlases frequently. Since referencing a new texture page implicitly interrupts the GPU state, all texture swaps are also batch breaks.

In theory, a texture swap is heavier than a batch break, because it could require data to be loaded and unloaded, which is much more expensive than simply stopping one batch and starting a new one. In practice, however, the graphics driver won't actually unload anything from VRAM unless it needs the space for something else, and if your game runs out of VRAM and has to resort to the shared GPU pool in main memory you have much bigger problems to worry about.

We'll talk a little more about GPU memory in the next two items.

### Verdict

It's important to not let this get out of hand, but as long as things don't get out of hand it's far from the most important thing. It's slightly more important on low-powered devices like laptops, and it's more important still or mobile phones. Be a little more judicious with surfaces. Don't run out of VRAM or bad things happen.

Here's a more detailed video I made on it a while ago.

# Disable surface depth if you don't need it

By default, all GameMaker surfaces come with a depth and stencil buffer, which you may or may not actually need. It won't automatically make a difference in performance unless you start running out of VRAM, and while AAA games may occasionally run into problems with this [citation needed] if we're being real with ourselves, your GameMaker game probably won't.

By default, surfaces come with a 32-bit depth and stencil buffer. This amounts to an extra (width * height * 4) bytes allocated per surface. If you never use depth testing anywhere (the gpu_set_ztestenable and gpu_set_zwriteenable functions) or make use of stencil shenanigans, this is pretty much useless to you and you can turn it off with surface_depth_disable.

A 1280 x 720 surface will reserve about 3.5 mb of VRAM for depth and stencil, while a 1920 x 1080 surface will use about 8 mb. It's not a life changing amount of VRAM - the sprites and other textures in your game almost certainly already use more - but if you don't need it, turning it off is really easy and basically free.

Reducing your VRAM usage like this won't automatically make your game faster (though I suppose allocating a surface with no depth will be faster), but if you're already using a lot of VRAM, this will give you more space for other stuff.

If you do need depth or stencil, you can also enable/disable surface depth on demand. The surface depth state will be taken into account when you create a new surface, or resize an existing one.

### Verdict

Feel free to turn off depth if you're not using it. If you use it in some places, make sure to turn it on when creating or resizing the surfaces that do need it, otherwise you can get weird depth sorting errors. Don't try to circumvent needing the depth buffer if you do need it.

# Texture compression

Another VRAM one.

On disk, all of your sprites and everything are usually stored in PNG format or something similar. Image files stored like this are compressed so that they take up less space on the disk, but when your game is actually running they need to be decompressed into what amounts to a large 2D array of pixel data, and will take up exactly width * height * 4 bytes of memory. Among other reasons, this means you can access any pixel in an image in constant time.

Texture compression on the GPU has now become widespread in games. This differs from the kind of image compression you're used to because of the requirement that any pixel should be able to be accessed in constant time, which isn't possible using algorithms like PNG or JPEG. Instead we have things like ASTC and S3TC. These compress all images by a constant factor, preserving the constant-time access property. However, unlike lossless algorithms like PNG, GPU texture compression achieves this by discarding certain information about blocks of pixels. This has the effect of producing artefacts similar to (but technically distinct from) JPEG artefacts, which are usually minimal in high-definition artwork, but can mess up pixel art pretty badly.

This speeds up your game in two point five ways.

1. Textures take up less space in VRAM, which means you use less of it. Using less VRAM doesn't make much of a difference in and of itself if you aren't running out of it, but as discussed previously, you don't want to run out of it. This is more of a problem for games with high-resolution art than it is for pixel art.
2. Textures take up less space in VRAM, which means they're faster to access. Like CPUs, GPUs have on-die memory caches. When you draw an image, texture data is streamed through the cache so that it can be accessed faster because it takes less time to access data that's physically on the processor die than it is to access data in a VRAM module some distance away on the PCB. If an image of the same size takes up less data in VRAM, more of it can fit into the texture cache at any given time, meaning that sampling from it is overall faster even if the GPU has to perform some extra logic to decompress it. This is the main reason compressed textures speed up your game.
2.5. When your game loads, textures are loaded from the disk into main memory and then transferred from main memory to VRAM. Loading and decompressing less data from the disk means your game may boot up or complete loading screens slightly faster.

Texture compression isn't a built-in feature of GameMaker itself, but it's available as an official extension maintained by Yoyo Games which you can use if you need it.

See the video for the kind of performance benefits and rendering artefacts you can expect from texture compression.

### Verdict

If you're making a game with HD art, you can usually get away with some amount of texture compression with minimal artefacts (basically all AAA games have been using this for years). If you're making a game that uses pixel art, absolutely do not do this. The savings will be small and your pixel art will come out looking like someone added jpeg noise to every sprite.

Visual novels would be an especially good example for when to use this, since they often involve a lot of large, high-resolution artwork.

# Be mindful how often you query the collision system

This was more relevant pre-GMS1 before the collision system was as robust as it is now, but it still makes a small difference. Imagine you have a player who takes damage when they collide with enemies. Logically, it doesn't matter if the collision-checking code for this belongs to the player and checks for enemies or if the code belongs to the enemies and checks for the player. However, if the code belongs to the enemy and there are a large number of enemies on screen, the collision check will run once for every enemy active in the game. If the code belongs to the player, assuming there's only one instance of the player object in the game, the collision check will run once. (Most of the important collision-checking functions have a \_list version which can be used to handle multiple hit results, rather than having to deal with them one at a time.)

The difference isn't huge, and honestly I don't expect most people will notice unless they're making a Vampire Survivors-like game with hundreds of enemies coming at you at once, or perhaps a bullet hell with hundreds or thousands of bullets that can all damage you. I thought about putting this in the B tier instead, but a lot of people would probably say it's good organizational practice to do this because it reduces the need for code duplication, so I'm giving it extra credit.

This also goes for the Collision events, but I don't think many people use those.

### Verdict

This can be a minor benefit in some types of games, but it's not going to save you if you're experiencing performance issues. However, even if it's not a major optimization, it's probably a preferable organizational strategy and I would generally encourage it for that reason.

It's worth pointing out that a lot of the here is in calling the collision function 6,000 times, and not the logic inside the collision function itself.

# B tier: Optimizations that technically work but you should only do this after you've gone through everything else in your game

These are all optimization tricks that are technically true, but won't save you if your game is running poorly. You can also make things worse instead of better if you employ them in a place where they're not appropriate.

Avoid functions in the update clauses of loops

When you write a for loop, it usually looks something like this.

```gml
for (var i = 0; i < 10; i++) {
    whatever(i);
}
```

And if you need to iterate over an array, you might do it like this.

```gml
for (var i = 0; i < array_length(array); i++) {
    var element = array[i];
    whatever(element);
}
```

This isn't that bad, but you are calling the array_length function every time you go through the loop. The array_length function doesn't do much, but unless you're modifying the array with each iteration (and you're not, right?) it's going to be the same value every time you go 'round. You can pull it out to a local variable before you run the loop for a mild performance boost.

```gml
var n = array_length(array);
for (var i = 0; i < n; i++) {
    var element = array[i];
    whatever(element);
}
```

If that looks too awkward you're also allowed to define multiple locals in the init clause (which is what I usually do).

```gml
for (var i = 0, n = array_length(array); i < n; i++) {
    var element = array[i];
    whatever(element);
}
```

There are a bunch of other structures in GameMaker you might loop over: ds_lists, buffers, strings, instances, etc. These should all get similar treatment.

## Using repeat loops instead of for loops (also known as, when to not trust pretty graphs)

There are a bunch of different ways to do loops in GameMaker. The classic for loop is ol' reliable, but the update condition and statement are executed in GML, and as we've seen GML isn't the fleetest afoot.

```gml
for (var i = 0; i < array_length(array); i++) {
    var element = array[i];
    whatever(element);
}
```

That's pretty basic usage of a loop in programming.

GameMaker also has a little control structure called a "repeat" loop. Instead of giving it an update condition and update statement, you just tell it to run a certain finite number of times. This moves some of the aforementioned work into the GameMaker runtime, which is faster. If you wanted to, you can change this to a repeat loop that looks something like this.

```gml
repeat (array_length(array)) {
    var element = array[i];
    whatever(element);
}
```

Oops, that won't work, because we don't have a loop counter. We still need the loop counter.

```gml
var i = 0;
repeat (array_length(array)) {
    var element = array[i];
    whatever(element);
    i++;
}
```

It's a little weird-looking, but it'll get the job done. Let's have a look at how that performs.

The repeat loop is about 40% faster than the ol' reliable. So if you use a repeat loop instead of a for loop, your loops will be 40% faster, right? Sounds great, right?

Wait, why did I put this in the B tier again?

This is where synthetic benchmarks can differ dramatically from the real world. In the real world, most of the cost of running the loop will be the code that you put in the loop, and since you just changed how the loop was written you're not doing any less of that. If the loop itself in your code takes 1 ms, and the content of the loop takes 10 ms, the total time that it takes to go through the loop will be 11 ms. If you speed up the loop by 40%, the loop itself now takes 0.6 ms, the content of the loop still takes 10 ms, and you've reduced the total time down to 10.6 ms. That's a 4% improvement.

This is one of the many examples of where looking at an individual piece of code instead of the broader context that it lives in can cause major problems.

### Verdict

Feel free to write your loops like this going forward, but I would recommend not going back and changing all of the ones you've already written. If your game is running slowly, this isn't going to help, unless your game is somehow nothing but empty for loops.