# Various Spine runtime problems and some workarounds

&nbsp;

### What's up with the animation update event? It's not behaving how I would expect.
The documentation is incorrect. This event does not run once per step right before draw, it runs every time ``draw_self()`` is called. It does NOT run if you draw the sprite using one of the draw_sprite functions. This means it can run 0 times or multiple times depending on how your drawing is handled. It also cannot be cleanly implemented with state machines since it's a separate event. Bone data will be updated between step and end step, so you can run code you would put in animation update in end step instead. I personally run my state machine in end step for every object that uses a spine sprite, and avoid using the animation update event entirely.

&nbsp;

### My event frame never triggers, even though I'm definitely going past that frame, but only for some events!
`skeleton_animation_get_frame` gets slowly get more and more inaccurate over time. The first time you run an animation, it might give you 5.00 for your event frame, and your `skeleton_animation_get_frame(channel) == skeleton_animation_get_event_frames("My Cool Animation", "My Cool Event")[0]` check will go through. However, on the 10th time, it could be giving you 5.05 and causing equivalency checks to fail. So, you need to instead check if you're close to the event frame instead of exactly on it. You could also forcibly set the animation frame back to 0 on each loop to prevent excessive deviation, but the problem will still happen if the event frame is later in the animation and the animation is long enough. Note that the slow drift of the frame number can be positive or negative, so simply flooring the frame will not always work, depending on the animation.
```
function spine_event(event_name, channel = 0, event_num = 0, deviation_range = 0.5) {
    var frame = skeleton_animation_get_event_frames(skeleton_animation_get_ext(channel), event_name);
    if abs(skeleton_animation_get_frame(channel) - frame[event_num]) < deviation_range {
        return true;
    }
    return false;
}
```

&nbsp;

### I need to draw some of my slots separately, so I can apply a shader, or draw them at different depths, or something else, but I don't see any function to draw specific slots.
Indeed, there is no function for drawing slots individually - it's all or nothing. However, I came up with a hacky workaround for this. skeleton_slot_color_set can set the alpha of individual slots, so I set the alpha of every slot I don't want to draw to 0, call draw_self() to draw them, and then either reset everything back to regular alpha, or do the reverse and swap which slots have alphas of 0 and 1 to draw the other slots. I included the functions I use to make it easier below.
```
/**
 * Function Takes an array of slot names and hides all slots not listed in that array, so only the listed slots will be drawn.
 * @param {array} slotsToShow An array containing the string names of the slots you want to show.
 */
function spine_slot_isolate(slotsToShow) {
    var list = ds_list_create();
    skeleton_slot_list(sprite_index, list);
    
    var hideSlot = true;
    for (var i = 0, size = ds_list_size(list); i < size; ++i) {
        for (var j = 0, len = array_length(slotsToShow); j < len; ++j) {
            if list[| i] == slotsToShow[j] {
                hideSlot = false;
                break;
            }
        }
        if hideSlot == true {
            skeleton_slot_color_set(list[| i], c_white, 0);
        }
        else {
            skeleton_slot_color_set(list[| i], c_white, 1);
            hideSlot = true;
        }
    }
    ds_list_destroy(list);
}

/**
 * Function Resets slot alpha, returning all slots to the default alpha of 1.
 */
function spine_slot_reset() {
    var list = ds_list_create();
    skeleton_slot_list(sprite_index, list);
    for (var i = 0, size = ds_list_size(list); i < size; ++i) {
        skeleton_slot_color_set(list[| i], c_white, 1);
    }
    ds_list_destroy(list);
}
```

&nbsp;

### How do I play different animations on the same skeleton at different speeds? There doesn't appear to be any functions to change the speed of different tracks individually.
You will need to progress the frames of each animation manually. Set image_speed to 0 in the create event to disable the automatic animation, and then manually count up the frame using `skeleton_animation_set_frame`.
