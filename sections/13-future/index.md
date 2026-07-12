---
title: Future work
has_children: false
nav_order: 14
---

# Known issues and future work

Despite the software product meets all the criteria identified and is considered by the authors as an enjoyable game, OrecchietTetris can be improved with bug fixing and new feature implementation. Here is a short list of issues or feature already identified for the new version of the game:

- Line-clear animation does not pause gravity: when the clearing animation starts, the falling-piece clock is not stopped. As a consequence, the next piece has already dropped a little as soon as the animation finishes, instead of starting its descent from the top.
- Audio is handled only inside the game screen: music and sound control currently live in `GameScreen` alone. For example, it is not possible to skip a track from the menu screen.
- Controls are not fluid enough: movement is bound to discrete key presses: for instance, holding a key down is not enough to keep moving a piece sideways, so the player has to tap the key repeatedly.
- Smoother gameplay and input handling: the overall responsiveness of the game and its controls could be improved, so that input feels immediate and the motion of the pieces is more continuous.
- Sound effects for animations: at the moment the animations (line clear in particular) have no dedicated sound.
- Responsive layout: the interface is not fully responsive yet, since it does not adapt to different window sizes or aspect ratios.
- Standalone, installable application: the game currently requires a working Python environment to run. A valuable next step would be to deliver it as an installable application, so that end users can install and launch it without having Python on their machine.
