# Turing Game
Turing Game a game where there are multiple challenges based on the turing machine for the player to complete. This programme was written by Ng Ri Chi (1004143, CC06) for the DW Final Assignment.

## [Introductory Video](https://sutdapac-my.sharepoint.com/:v:/g/personal/richi_ng_mymail_sutd_edu_sg/ESbYQFLCugpChpmpE98HbtMBW905CgmdIixHe0zc4CNpSg?e=9oyzNF)
There are slight cosmetic changes, and gameConf.txt is now fully configured.

## Prerequisite
This python game requires [libdw](https://people.sutd.edu.sg/~oka_kurniawan/10_009/doc/html/) and [kivy](https://kivy.org/#home). Use the package manager [pip](https://pip.pypa.io/en/stable/) to install them. For best playing experience, ensure your environment supports ANSI Select Graphic Rendition. If you are using Windows please consider running in [VS Code](https://code.visualstudio.com/) (this game's coding environment) or the [Windows Terminal](https://github.com/microsoft/terminal).

## Medium
Please note that this game is predominately text based, and GUI is only used to set up the dictionary that configures the turing machine (`confDict`). The game also loads the text files `confDict.txt` and `gameConf.txt` which allows for the player to save and manipulate confDict outside the game and modify the challenges respectively.

## Game Play
The game play is as follows:
1. Player loads the game by starting the `main.py` file and selects a challenge from the main menu
2. Game loads the challenge and presents the description.
3. The player will be prompted to enter a tape. If the challenge provides a tape the player can choose to use the tape by just pressing ENTER instead. See explanation for `class Game` function `getTape` below for more details.
4. If number os state is not given, player will be prompted for input.
5. The game will then prompt the player to use the GUI template to input the `confDict`, or use `confDict.txt`. See explanation for `class Game` function `getDict` below for more details.
6. The game will ask the player if `confDict.txt` should be re-written. This is useful if the player wants to edit the `confDict` generated by the GUI template and use it for later plays.
7. The player runs the tape. See explanation for `class Turing` below for more details.
8. The resulting Tape, tape before transformation and "Model Tape" (if the challenge provides a model `confDict`) is presented. Player is congragulated if the model and output tape are the same, and encouraged to try again if it is not.
9. The game returns to the main menu, where the player can play another round or exit the game

## Code
Here you can find a class by class breakdown of the code. Modules here are written to ensure player has a smooth experience, and will not run into any errors even though this is a text based game. As much as possible classes are put into separate file for neatness. For greater clarity it is reccomended to read this portion together with the code.

### `class Turing` in `Turing.py`
`class Turing` is a  `libdw.SM` subclass. It describes a turing machine that has a head that can read and write symbols (S (read only), X (clear), 0 and 1)(It supports more states but the definition in KivyUI limits us to just these) and can move the tape left or right by up to one cell. The tape has a finite length and loops at the end if the machine is not halted by the S state or S symbol.
#### `__init__`
Initialises variables in the class when first initiated, and also `__len` if tape is input. It is also possible to change the starting cursor of the machine but in the game the default is position 0 (at the start).
#### `get_next_values`
Gets the next state, output and movement direction according to `confDict`from the input from the tape and the state.
#### `nextCursor`
Outputs the position of the head after a input. This is separated from get_next_valuesbecause of the need to reference the tape. Also ensures tape loops at the end so it does not go out of range.
#### `step`
Updates the tape, state, count, cursorNow and cursorThen when the head moves a step. Also ensures tape stops if it reads S or has timed out (defined if the count goes more than 10 times the length of tape).
#### `__str__`
Outputs a really nicely formatted string that shows the player the tape, read and write position and status, count number and state.
#### `run`
Runs the tape from start till it runs out and prints out the result for each step. Used in `class Game` to show the player how `confDict` processes tape
#### `quietRun`
Same as run but has no printout. Used to calculate model tape output.
#### `getDict`
Used to display all variables for troubleshooting and debugging
#### `test`
Used in `class Game` function `getDict` to ensure `confDict` that is input will run without problems. Works by running all states `confDict` into all the possible inputs and detecting if there are any errors.
#### `tape`
Outputs tape

### `KivyUI.py`
This file contains a few Kivy subclasses to help make the `confDict` creation really easy. It contains a table, where the rows represent a state in the turing machine and the columns the input and its respective output state, print out and movement direction. In fact, when making new games, it is recommended that the model `confDict` to be made using this tool.
#### `Config`
Not a class, but it makes the window have a fixed size to ensure alignment of lables and spinners are accurate.
#### `class DropMenu`
A spinner subclass to ensure appearance of spinner is uniform
#### `class StateBox`
Forms the rows for each state. Inherits DropMenu class.
#### `class ConfDictMaker`
Makes the table wutg `StateBox class`, and number of states (rows) depends on the input of the player (pretty cool right?). This is possible because the wigets are linked to a dictionary, and the dictionary allows for flexible addition of states, as well as the access of values. Lastly, the `confDict` will be output regardless whether the "Submit" button is pressed of the window is closed. This is to reduce the chance of the game hanging.

### `main.py`
Holds the "operational" part of the code.
#### `os`
Ensures working directory is same as files so that `confDict.txt` and `confGame.txt` can be accessed
#### `reset` function
After every game Kivy needs a reset so it can start again.
#### `__init__` function in `class Game`
Initialised using kwargs so that dictionary from the text file can be directly used as input.
#### `getTape` function in `class Game`
Gets tape input from player by cycling the following statuses:
1. `GetInput` sub-function: Makes all alphabets upper case
2. `MainCheck`: Checks tape only contains X, 0, 1 or S
3. Modify `sCheck` variable in `MainCheck`: Ensures S only appears at the end. This is done by throwing an input error if there is a new character if S is already detected.
4. `Fail`: If the checks fails, explains possible error and asks player for a new input
5. `Pass`: Confirms checks are passed outputs the tape
#### `getDict` function in `class Game`
1. Gets `confDict` using GUI template or file.
2. If its GUI template, `class ConfDictMaker` in `KivyUI.py` is launched.
3. If from file, checks if it passes the Turing.test() so that game runs without errors. If there is problem, it automatically switches to GUI template. Also indicates whether Kivy was use for reset later.
4. Outputs `confDict`
#### `run` function in `class Game`
Run encodes the Game Play described earlier. Clear instructions are provided so that players are not lost.
#### Last few lines of code
The last few lines of code forms the back bone of this programme.
1. Imports `confGame.txt` and splits it up to individual games, and further splits up each game to the title and game configuration
2. Forms menuItems from imported games and adds an exit option at the end
3. Prints welcome message
4. Enters while loop that regulates interaction between main menu and game
#### the While Loop
1. `Menu`: Prints out menu, and ensures player inputs valid options. Prompt again if input is incorrect
2. `Game`:Runs game and resets kivy if kivy is ran
3. `Exit`: Exits the While loop, where the programme exits

## Bundled challenges
0. Make your Mark
1. Turing's first
2. Over the Fence
3. Unary Addition
4. Unary Subtraction
5. Copy
6. Free Play

## Possible future features
1. Encryption of `confGame.txt` so player cannot see solution.
2. Enable player to start in selected cursor other than 0.

## Donation
If you have enjoyed the game or find it helpful, consider donating a token sum via [payNow](https://www.abs.org.sg/consumer-banking/pay-now). I can send you my number via my email. Thanks!