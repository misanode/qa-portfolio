## TC-0001: Player overlay remains centered when media items are removed

Priority: Low

Preconditions: Squidd is launched and the player overlay is visible

Steps:
1. Launch Squidd
2. Start playback so the player overlay is visible
3. Trigger a state change where one item is removed
4. Observe the overlay during the transition

Expected Result: The player overlay remains visually centered as items appear or disappear

Related defects: [misanode/Squidd#6](https://github.com/misanode/Squidd/issues/6)
