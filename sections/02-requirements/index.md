---
title: Requirements
has_children: false
nav_order: 3
---
# Requirements
## User stories
- As a **player**, I want to start a new game with a single click, so that I can begin playing immediately without setup.
- As a **player**, I want simple, responsive controls (move, rotate, hard drop, hold), so that the game feels immediate and skill-based rather than fiddly.
- As a **player**, I want the game to gradually get faster as I clear more lines, so that the challenge scales with my skill and the game stays engaging over time.
- As a **player**, I want to pause the game at any time, so that I can step away without losing my progress or being penalized.
- As a **returning player**, I want to save my score with my name after a game over, so that I can see how I rank against my own and others' previous attempts.
- As a **returning player**, I want to view a leaderboard with the best scores, so that I have a reason to keep coming back and improving.
- As a **player**, I want to switch the game's language between Italian and English, so that I can play in the language I'm most comfortable with.
- As a **player**, I want background music themed around Apulian culture, with the ability to mute it or skip tracks, so that I can control my own audio experience while enjoying the game's theme.
- As a **player**, I want to install and run the game with a single command, so that getting started requires no technical expertise.

## Requirements analysis
- Type of requirements:
    - Functional:
        - F1: the user must be able to reach a main menu offering three actions: start a new game, change language, and view the leaderboard
        - F2: the user must be able to control a falling tetromino: move it left/right, rotate it, soft-drop it, hard-drop it (teleport straight to the bottom), and swap it with the next piece (hold)
        - F3: the system must detect and clear completed lines (rows fully filled with blocks, no gaps) and award the player points for each clear
        - F4: the system must increase the game's difficulty (fall speed) as the player progresses, in discrete levels
        - F5: the user must be able to pause and resume an ongoing game at any time
        - F6: the system must detect the game-over condition (a new tetromino cannot be placed because the stack has reached the top) and end the session
        - F7: upon game over, the user must be able to enter a name and have their score saved to a persistent leaderboard
        - F8: the user must be able to view the leaderboard, with the top three scores highlighted separately (e.g. on a podium) from the rest of the ranked list
        - F9: the user must be able to switch the application's language between Italian and English at any time from the main menu
        - F10: the system must play a looping background soundtrack automatically when the application starts, and let the user mute it, skip to the next track, or return to the previous track
    - Non-functional:
        - NF1: the game must run smoothly, targeting at least 30 FPS, ideally 60 FPS
        - NF2: the game must run identically on Windows, macOS, and Linux
        - NF3: the game must be usable without any network connection or external backend service
        - NF4: the game's controls and menus must be simple enough to be understood without instructions (pick-up-and-play)
        - NF5: saved leaderboard data must persist across application restarts and must not be lost on a normal game-over or quit
        - NF6: the game must be installable and launchable through a minimal number of steps (ideally one command)
        - NF7: the visual and audio theming must consistently reflect Apulian culture across blocks, music, and menus
    - Implementation:
        - I1: the software must be developed in Python (>= 3.11), as mandated by the course requirements for the project work
        - All other technical choices (Kivy for the GUI, Poetry for dependency management, distribution via PyPI, the Observer-based architecture, etc.) are **design decisions** made to satisfy the requirements above, not externally imposed constraints — they are discussed and justified in the Design section instead
- Glossary:
    - Tetromino: a geometric piece made of four connected blocks, in one of seven shapes (I, O, T, S, Z, J, L), themed with an Apulian food image
    - Board: the 10x20 grid on which tetrominoes fall and stack
    - Line clear: the event where a row of the board is completely filled with blocks and is removed, awarding points
    - Soft drop: making the current tetromino fall faster than normal while a key is held
    - Hard drop: instantly moving the current tetromino to the lowest valid position ("teleporting" it down)
    - Hold: swapping the current tetromino with the next one in the queue, storing the current one for later use
    - Level: a discrete difficulty stage; increasing the level increases the tetromino fall speed
    - Leaderboard: the persisted, ranked list of player names and scores
    - Podium: the visual highlight given to the top three leaderboard entries
    - Locale: the active display language of the game (Italian or English)
    - Apulian theming: visual (block art) and audio (soundtrack) elements drawn from the culture and cuisine of Apulia (Puglia), Italy
- Acceptance Criteria for the requirements
    - F1: from the main menu, the user can navigate to a new game, the language selector, and the leaderboard, each via a single, clearly labeled action
    - F2: each control (move, rotate, soft drop, hard drop, hold) produces the expected, immediate change in the tetromino's position, orientation, or identity, with no perceptible input lag
    - F3: a row with no empty cells is removed from the board and the player's score increases; a row with any empty cell is left untouched
    - F4: the level indicator increases as the player clears lines, and the fall speed measurably increases at each new level
    - F5: pausing halts the falling piece, the timer, and the fall speed timer; resuming continues the game exactly from the paused state, with no loss of board state
    - F6: the game ends only when a new tetromino cannot be placed at the spawn position; the final state of the board is shown to the player
    - F7: after entering a name and confirming, the score and name appear in the leaderboard and persist after the application is closed and reopened
    - F8: the leaderboard view visually distinguishes the top three scores (podium) from the remaining ranked entries, all attributed to the correct saved names
    - F9: selecting the other language immediately updates all visible menu and in-game text, without requiring a restart
    - F10: music starts automatically on launch, loops after the last track, and mute/next/previous controls each produce the expected audio change within one interaction
    - NF1-NF7: if achieved, this will be verified through manual playtesting and, where applicable, automated tests measuring frame timing and cross-platform runs
    - I1: the published package and source code declare and require Python >= 3.11, and the game runs correctly under that version


## Use Case Diagram

![use case diagram](../pictures/use_case_diagram.png)

