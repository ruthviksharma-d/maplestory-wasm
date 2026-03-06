
# Issue #23: Game Freezes When Tab is in Background

## Problem
When the browser tab becomes inactive, browsers throttle or pause `requestAnimationFrame`. When the tab becomes active again, the timer returns a very large elapsed time, causing the accumulator loop to execute thousands of `update()` calls in a single frame, resulting in a freeze (spiral of death).

## Solution
Cap the maximum elapsed time processed in one frame to prevent the spiral-of-death.

## Changes Made

### src/client/Journey.cpp

Added a constant to cap elapsed time and applied the clamp in the game loop:

```cpp
// Maximum elapsed time to prevent spiral-of-death when returning from background.
// 250000 microseconds = 250ms, which is ~4 frames at 60fps
static constexpr int64_t MAX_FRAME_TIME = 250000;

void main_tick()
{
    // ...
    int64_t elapsed = Timer::get().stop();

    // Cap elapsed time to prevent spiral-of-death when tab returns from background
    if (elapsed > MAX_FRAME_TIME)
    {
        elapsed = MAX_FRAME_TIME;
    }

    accumulator += elapsed;

    for (; accumulator >= timestep; accumulator -= timestep)
        update();
    // ...
}
```

## How It Works
- Even if the tab is backgrounded for hours, the game only processes at most 250ms of game time per frame when it returns
- This prevents thousands of update() calls from executing in a single frame
- The game resumes normally instead of freezing

## No Changes Required
- `web/index.html` remains unchanged
- No visibility change handlers needed
- No JavaScript modifications required

