---
title: Validation
has_children: false
nav_order: 6
---

# Validation

## Testing Approach

The whole team, including the AI assistants used during development, worked following the Test-Driven Development (TDD) approach. Every behaviour entered the codebase as a failing test (red), was then implemented with the minimum amount of code needed to make that test pass (green), and finally cleaned up without touching its behaviour (refactor).

The test suite followed a unit-first strategy with a mirrored structure. Each production module has a matching test module under `tests/`, replicating the package tree. Keeping the two trees aligned makes gaps immediately visible: if a module has no twin under `tests/`, it has not been tested. It also keeps the red-green cycle short, because a failing test points to one file.

The domain layer (i.e. the `model` package) is pure logic with no framework involved, hence it is the layer where test-first design pays the most, making it the most densely asserted part of the suite. The adapters (the CSV repository, the audio service) and the Kivy view are tested against their interfaces, so the tests describe the expected behaviour and not the internal implementation, and a change in the implementation does not break them without reason.

### Framework

The framework used for test suite is `pytest` with `pytest-cov`. It is preferred rather than the standard library `unittest` for two reasons:

- plain `assert` statements give rich failure introspection, which keeps the test code short and readable during a tight red-green loop;
- fixtures allow composable setup and easy injection of test doubles.

Headless isolation. The root `tests/conftest.py` sets `KIVY_WINDOW=headless` and `KIVY_NO_ENV_CONFIG=1` before any Kivy import. Furthermore, `tests/view/conftest.py` injects minimal Kivy stubs into `sys.modules`. Thanks to this, the presentation layer can be tested on CI runners that have no graphical environment at all.

### Quality Gate

Coverage is measured with `poe coverage`, which runs `pytest --cov=OrecchietTetris --cov-report=term-missing`. The same command runs in the CI pipeline on every push and pull request, together with the static checks, so no untested or untyped code reaches the default branch. Combined with TDD, new code normally arrives already covered by the test that motivated it. Quality of the tests is also evaluated against the ability to cover the requirements adequately.

## Automated Testing

### Test doubles

Included with the definition of the tests, test doubles were also implemented:

- Mocks, based on `unittest.mock.MagicMock`: they stand in for the collaborators of the view (model, repository, audio service, UI callbacks) and, in `test_app.py`, for every heavy dependency of the composition root. The reason is that those tests verify interactions, that is whether the right method is called with the right arguments. Additionally, a mock is cheaper to inject than the real object, since the latter would drag in game state or input and output.

- Fakes and stubs: `MockObserver` is a subclass of `Observer` that records the events it receives. In this way, the contract between model and observer can be tested with an actual subscriber and without a real Kivy view. The Kivy widget stubs in `tests/view/conftest.py` are minimal classes with `ABCMeta` as metaclass. They provide just enough behaviour for the widget constructors to run. In this way, the issue that a real graphical backend cannot be instantiated is solved.

### Unit Testing

The suite counts 468 tests, organised to mirror the production package tree. Each module is exercised in isolation, with test doubles injected in place of its collaborators.

#### Domain and Logic Layer

| Module | Tests | What it covers | Requirements |
| ------ | ----- | -------------- | ------------ |
| `test_tetris.py` | 56 | Spawning, gravity `tick`, moves and rotations accepted or rejected by collision, hard drop, hold with its one-per-drop rule, line clearing, scoring, level progression, the `tick_interval` speed curve, pause and resume, game over, and the domain events emitted at each step | F2, F3, F4, F5, F6 |
| `test_board.py` | 40 | Piece placement, `is_valid_position` on boundaries and overlaps, `clear_lines` on single and multiple rows, `is_game_over`, `reset` | F2, F3, F6 |
| `test_tetromino.py` | 9 | Shape identity and rotation matrices of the seven pieces | F2 |
| `test_bag_tetromino_factory.py` | 5 | The 7-bag randomiser: every shape appears exactly once per bag, the bag refills when exhausted, `reset` empties it | F2 |

#### Observer Infrastructure

| Module | Tests | What it covers | Requirements |
| ------ | ----- | -------------- | ------------ |
| `test_observer_subject.py` | 12 | Attach and detach, fan-out of notifications to several observers, delivery of the event payload | Since this is the communication backbone between model and view, a defect here would break every screen update at once. |

#### Leaderboard Repository

| Module | Tests | What it covers | Requirements |
| ------ | ----- | -------------- | ------------ |
| `test_csv_leaderboard_repository.py` | 14 | Save and load round trip, append semantics, sorting by score in descending order, behaviour when the CSV file does not exist yet | F7, F8, NF4 |
| `test_leaderboard_entry.py` | 5 | Fields of the value object | F7, F8 |
| `test_ileaderboard_repository.py` | 4 | The interface contract | F7, NF4 |

> During such tests, repository is always pointed at a temporary directory, so the real user data folder returned by `platformdirs` is never touched during the tests.

#### Audio Service

| Module | Tests | What it covers | Requirements |
| ------ | ----- | -------------- | ------------ |
| `test_kivy_audio_service.py` | 44 | Play, stop and toggle state, volume setting, queue advance and wrap around | F10, NF6 |

> `SoundLoader` is mocked, so the behaviour is asserted without any real playback and without depending on the audio hardware of the machine.

#### View and Presentation Layer

| Module | Tests | Requirements |
| ------ | ----- | ------------ |
| `test_game_screen.py` | 69 | F2, F3, F5, F6, F7, F10 |
| `test_menu_screen.py` | 34 | F1, F9 |
| `test_leaderboard_screen.py` | 28 | F8 |
| `test_app.py` | 16 | F1 |
| `test_block_renderer.py`, `test_i_view.py` | 21 | NF6 |
| Reusable widgets (`board_widget`, `dialog_overlay`, `leaderboard_row`, `top_player_card`, `titled_box`, `cell`, `piece_preview`, `rounded_button`, `rounded_toggle_button`) | 108 | F8, NF3, NF6 |

`test_game_screen.py` covers the mapping between keyboard input and model commands, the dispatch of domain events to the right routine, the line-clear animation, the game-over overlay and the saving of the score. The view is deliberately thin, but its wiring, that is key bindings, observer dispatch and dependency injection, is the delicate part to be tested appropriately.

#### Results

| Metric | Value |
| ------ | ----- |
| Total unit tests | 468 |
| Passing | 468 |
| Failing | 0 |
| Success rate | 100% |
| Statement coverage (`pytest-cov`) | 100%, 1884 statements out of 1884 |

All 468 tests passed and no statement of the `OrecchietTetris` package is left uncovered. The HTML coverage report is uploaded as a build artifact by the CI workflow, so the result can be inspected for any commit.

### Integration testing

Toghether with the isolated unit tests, several tests exercise pairs of collaborating components through their real interfaces, in order to verify that the contract between them holds from one side to the other. They live in the same modules as the unit tests because they were written in the same process. The following list shows the tested pairs:

- `Tetris` & `Observer`: every state-changing command is checked to emit to `MockObserver` the correct `EventType` with the correct payload.

- `Tetris` & `Board` & `BagTetrominoFactory`: model tests also work as integration tests of the whole logic layer, where collision, locking, clearing, spawning and 7-bag sequencing are exercised together exactly as they run in production.

- `GameScreen` & its collaborators: the screen is constructed with a mocked model, repository and audio service injected through its constructor. The tests assert that a key press is translated into the right model command, that an incoming domain event reaches the right routine, and that a game over followed by a confirmed name triggers a save on the repository.

- Composition root: `build()` is executed with `Tetris`, `CsvLeaderboardRepository`, `KivyAudioService` and the three screens all patched, verifying that the application instantiates every collaborator and wires the `ScreenManager` correctly. This confirms that the object graph is assembled as designed, without opening a real window.

### System testing

We did not write automated end-to-end tests were developed. The behaviour of the running application and the user experience are verified manually, as described in the next section.

What is verified automatically at system level is the environment in which the application must run, through the CI/CD workflow in `.github/workflows/deploy.yml`:

- the whole suite runs on three operating systems (Ubuntu, Windows and macOS) against four Python versions (3.11, 3.12, 3.13 and 3.14), for a total of twelve runs, which is the automated part of the evidence for NF1 and I1;
- the package is built with Poetry, published to TestPyPI and then installed again from TestPyPI in a clean runner before any release reaches the official index, which verifies NF5, that is that the game can really be installed with a single `pip install`.

## Manual Acceptance Tests

The acceptance tests were executed by hand on the assembled application following the plan below. The plan is written so that another person can repeat it exactly. Each test corresponds to one acceptance criterion of the Requirements section.

| Requirement | Steps | Expected result |
| ----------- | ----- | --------------- |
| F1 | Start the application and look at the main menu | Music starts playing and buttons are available and actionable (new game, leaderboard, settings, info, exit) |
| F2 | During a game press arrow keys, `x`, `spacebar`, `c` | The piece moves left and right, falls faster while down is held, rotates, drops instantly to the lowest valid position, and is swapped with the next one. Every reaction is immediate, with no perceptible delay. A second hold before the piece has locked is refused |
| F3 | Fill a row completely, then fill a row leaving one empty cell | The complete row disappears and the score increases; the incomplete row stays on the board |
| F4 | Clear ten lines and watch the level indicator and the fall speed | The level goes from 1 to 2 and the pieces fall slightly faster. The score awarded for the following clears grows accordingly, because it is multiplied by the level |
| F5 | Press `p` or escape during a game, wait some seconds, then resume | The piece stops falling and the timer freezes. On resume the board is exactly as it was left, with no piece having moved in the meantime |
| F6 | Stack the pieces up to the top of the board | The game ends only when a new piece cannot be placed at the spawn position, and the final board stays visible behind the overlay |
| F7 | At the game over, type a name and confirm; close the application and open it again | The name and the score appear in the leaderboard, and they are still there after the restart |
| F8 | Open the leaderboard with at least five saved scores | The first three entries are shown on the podium, visually separated from the ranked list below, and each score is attributed to the right name |
| F9 | From the main menu switch the language from Italian to English and back | All the visible text of the menus and of the game changes immediately, without restarting the application |
| F10 | Start the application and listen; then press `m`, `n` and `b` during a game | The soundtrack starts automatically, mute silences it and restores it, `n` moves to the next track and `b` to the previous one, and after the last track the queue starts again from the first |
| NF2 | Play a full session with the network interface disabled | Nothing changes: no feature asks for a connection and no error is shown |
| NF3 | Ask a person who has never seen the game to start a game and save a score, without giving explanations | The person reaches the game and saves a score without asking for help |
| NF4 | Save some scores, then quit the game normally, both from a game over and from the quit command `q` | The CSV file in the user data directory keeps all the entries, and no score is lost |
| NF5 | On a clean machine, run `pip install OrecchietTetris` and then `python -m OrecchietTetris` | The game installs and starts with two commands and no manual configuration |
| NF6 | Look at the blocks, the menu images and listen to the soundtrack | The seven tetrominoes use Apulian food images, and the music and the menus follow the same theme consistently |
| I1 | Check the declared version in `pyproject.toml` and run the game on Python 3.11 | The package requires Python 3.11 or higher and the game works correctly under that version |

> Manual Tests have been executed on MacOS and Windows, double-checking the requirement NF1 toghether with system tests executed in CI/CD workflow.
