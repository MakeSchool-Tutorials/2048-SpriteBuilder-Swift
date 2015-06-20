---
title: Finishing the core gameplay
slug: finishing-game
---

#Finishing the Game

In the second part of this tutorial we have added user interaction, tile movement and tile merging. We implemented all the basic mechanics of the game. In this part of the tutorial we will add a win and a lose condition to the game, keep track of scores and add some finishing touches to our very own version of *2048!*

We will start with adding some scores to our game.

## Keep track of scores

In *2048* the player's score increases when two tiles merge. We are not yet keeping track of any scores in our version of *2048.* Let's change that right away!

We will start by displaying the score of the current game. Later on we will also keep track of the player's overall highscore.

### Keeping track of current game's score

First of all we need a new variable to store the current score of the game. It could make sense to add an entirely new game class to our program that would encapsulate all the information about our game. For a *2048* game however, there is very little information to be stored. To not add unnecessary complexity we will store the current game score as a property of the *Grid*, which is absolutely fine for this type of game.

> [action]
> Add this variable to *Grid*:
>
>       var score: Int = 0

We will use this new integer variable to store the score of the game.

As mentioned above, the score of the game increases when two tiles merge. The score increases by the combined value of both tiles. For example a merge between a "4" tile and another "4" tile will result in 8 points.

The best place to add this functionality is the method where we perform the merge between to tiles: *mergeTilesAtindex.*

> [action]
> Add a line to increase the score to the beginning of *mergeTilesAtindex* in *Grid*:
>
>       var mergedTile = gridArray[x][y]!
>       var otherTile = gridArray[otherX][otherY]!
>       score += mergedTile.value + otherTile.value
>
> **Attention:** you should only add the line that increases `score`!

Now we are successfully keeping track of the current score of the game.

At the moment the player will not see that we are keeping track of the score. We are updating a variable but we are never displaying the new score in the game. You maybe remember that we set up a score and a highscore label in one of the first steps of this tutorial. Both labels are part of the *MainScene*. We will have to add the code that updates the score label to *MainScene*.

<!--consider updating to KVO in MainScene on score in Grid -->

### Displaying the new score

The change we need to make is relatively simple. `Grid` should always have a reference to `MainScene` through `parent`. `parent` gets set when `Grid` is added as a child of `MainScene`. All we need to do is create a `didSet` property observer for `score` so we can update `scoreLabel`.

> [action]
> Change the following line in `Grid.swift`:
>
>       var score: Int = 0
>
> to:
>
>       var score: Int = 0 {
>           didSet {
>               var mainScene = parent as! MainScene
>               mainScene.scoreLabel.string = "\(score)"
>           }
>       }

 **Well done!** Now you can play the game and should see the score increasing:

![](./score.gif)

Now let's determine when a game is over so that we can store a highscore!

## Add a win and a lose condition

A player loses in *2048* when they cannot perform any further move. This situation occurs when the grid is full and none of the existing tiles can be merged. We need to detect this situation so that we can end the game. The player wins the game if they reach the "2048" tiles.

The best place to detect if the losing condition occurred in the game is the *nextRound* method. In the *nextRound* method we spawn a new random tile. After we have spawned a tile we can check if the grid is full and if any further moves are possible. If no moves are possible we end the game.

The best place to check the winning condition is in the *mergeTilesAtindex* method. That is the method where we actually perform the merge between two tiles and determine the new value of the merged tile. If the new value is *2048* we know that the player has won the game.

Let's get started by implementing the win condition.

### Implementing the win condition

Let's start by setting up a constant for the value of the final tile.

> [action]
> Add this constant to the other constants in the `Grid` class:
>
>       let winTile = 8

For debugging purposes we will set the `winTile` to be `8`. This way it will be a lot easier to test if the win condition works, reaching "2048" can take quite a lot of time ;)

Now we will need to check if this win condition occurs.

> [action]
> Add the following line the *mergeTilesAtindex* method in `Grid` in the same place you define your other `CCAction`s:
>
>		var checkWin = CCActionCallBlock.actionWithBlock { () -> Void in
      	if otherTile.value == self.winTile {self.win()}
      	} as! CCActionCallBlock
>
> Now let's add this `CCAction` to the `CCActionSequence` we call on *mergedTile*. Change:
>
> 		var sequence = CCActionSequence.actionWithArray([moveTo, mergeTile, remove]) as! CCActionSequence
>
>  To:
>
> 		var sequence = CCActionSequence.actionWithArray([moveTo, mergeTile, checkWin, remove]) as! CCActionSequence

Once the value of the merged tile reaches the value of the *WIN_TILE* we call the *win* method! You can see that we have to check the value only after we *mergeTile*, as that's when we assign *otherTile* its new value.

> [action]
> Add the *win* method to *Grid*:
>
>       func win() {
>           endGameWithMessage("You win!")
>       }


All we do in the *win* method is calling the *endGameWithMessage* method. In our game, most tasks that need to be performed to reset a game will be the same for won and lost games (reset game, store new highscore, etc.). Therefore it makes sense to extract this functionality into the *endGameWithMessage* method instead of duplicating the code.

We simply pass a different text to method for lost or won games.

> [action]
> Add the *endGameWithMessage* method to *Grid*:
>
>       func endGameWithMessage(message: String) {
>           println(message)
>       }

For now, all we are doing in this method is logging to the console for debugging purposes. Now you are ready to test this new feature. Run the game. Merge tiles until you reach the "8" tile, then you should see **"You win!"** appear in the Xcode console.

Well done! Now let's implement the losing condition.

### Implementing the losing condition

Detecting the losing situation is a little more complex then a win situation. A losing situation occurs when the entire grid is filled with tiles and no merges between these tiles are possible. Then there is no possible move left in the game. We will have to add code to detect such a situation in our game. On a high level with have do to the following: in the *nextRound* method we need to check if the player is able to perform a move or not. If the player cannot move the tiles in any direction we need to end the game.

> [action]
> Add the following lines to the end of the *nextRound* method in *Grid*:
>
>       if !movePossible() {
>           lose()
>       }

So far, so simple. If no move is possible, the player loses. We now need to add the *movePossible* and *lose* methods.

Let's start with the *movePossible* method. The *movePossible* method reads the entire grid to determine if any moves are possible. It returns a boolean value that defines if moves are possible or not.

> [action]
> Add the *movePossible* method to *Grid*:
>
>       func movePossible() -> Bool {
>           for i in 0..<gridSize {
>               for j in 0..<gridSize {
>                   if let tile = gridArray[i][j] {
>                       var topNeighbor = tileForIndex(i, y: j+1)
>                       var bottomNeighbor = tileForIndex(i, y: j-1)
>                       var leftNeighbor = tileForIndex(i-1, y: j)
>                       var rightNeighbor = tileForIndex(i+1, y: j)
>                       var neighbors = [topNeighbor, bottomNeighbor, leftNeighbor, rightNeighbor]
>                       for neighbor in neighbors {
>                           if let neighborTile = neighbor {
>                               if neighborTile.value == tile.value {
>                                   return true
>                               }
>                           }
>                       }
>                   } else { // empty space on the grid
>                       return true
>                   }
>               }
>           }
>           return false
>       }

This method iterates over the entire grid and selects each index. For each index the loop checks if the position on the grid is free. If the position is free that means the player can definitely perform a move, so we immediately return *true*. If the index is not empty, we check all the neighbors of the tiles and see if they have the same value as the tile at the current index; if that is the case, the tiles could be merged, so we can return *true* because there is a possible move.

If *every* index is occupied and none of the neighbours of a tile has the same value, we will complete the iteration through the grid and return *false* at the end. This means that the player has no possible move left and will lose the game.

To access a tile at an index we use a utility method which we need to add to our program: *tileForIndex*. The *tileForIndex* method simply takes an index and returns the tile at that grid position.

> [action]
> Add the *tileForIndex* method to the *Grid* class:
>
>       func tileForIndex(x: Int, y: Int) -> Tile? {
>           return indexValid(x, y: y) ? gridArray[x][y] : noTile
>       }

This method either returns a *noTile* in case an invalid index was provided. Otherwise it picks the relevant tile from the *gridArray*.

Now there's one last method which we need to add to actually test the lose condition:the *lose* method. We are already calling the *lose* method from the *nextRound* method.

> [action]
> Add the method to *Grid*:
>
>       func lose() {
>           endGameWithMessage("You lose!")
>       }

Now we have put the parts together. In the *nextRound* method we check wether a move is possible or not, using the *movePossible* method. If no move is possible we call the *lose* method that uses the *endGameWithMessage* to end the game and display a lose message.

As you may remember the current implementation of *endGameWithMessage* just logs a message to the console.

We have one little issue left; currently, it is really difficult to lose in the game. It will take many moves and result in a very long debugging cycle.

> [action]
> To make losing easier open *Tile.swift* and change the line in the *didLoadFromCCB* method that generates a random number to:
>
>           value = Int(CCRANDOM_MINUS1_1() * 201) + 201


Now the tile numbers will be so widely spread that it is very easy to lose. **Run the new version of the game.** After a couple of moves your grid should look like this:

![](./SimulatorWow.png)

Additionally you should see a log message "**You lose!"** in the Xcode console. We now can detect if a player wins or loses the game!

Now that we have tested the new functionality we can reset the values that we chose for debugging.

> [action]
> Change the line that sets the value in the *didLoadFromCCB* method of *Tile.swift* back to:
>
>       value = Int(CCRANDOM_MINUS1_1() + 2) * 2
>
> Additionally change the *winTile* constant in *Grid* to:
>
>       let winTile = 2048

Great! Another step toward completing the game. Next we are going to take care of storing players' highscores.

## Keep track of highscores

The score of the current game only needs to be stored while the game is going on. That's why we can use a simple variable to store the *score*. However, the highscore should be persistent. If a player restarts the app, the highest score from any previous game should be available.

In iOS, the easiest way to store simple information persistently is using *NSUserDefaults*. *NSUserDefaults* provides a very simple interface to store key-value information.

A good place to update the highscore is when the game ends.

Add these lines to the end of *endGameWithMessage*:
>
    let defaults = NSUserDefaults.standardUserDefaults()
    var highscore = defaults.integerForKey("highscore")
    if score > highscore {
      defaults.setInteger(score, forKey: "highscore")
    }

What are we doing in these couple of lines? We are reading the current highscore from *NSUserDefaults.* If the score of the current game is a new highscore, we store the new value in *NSUserDefauts*.

Now we are storing the highscore but we are not updating the label that displays the highscore yet. You might remember that we had a similar problem when displaying the score.

*MainScene* is the class that takes care of displaying the labels. That is where we need to add the code to update the highscore label.

> [action]
> Let's first add a method to *MainScene* that takes care of reading the newest highscore and updating the label to display it:
>
>       func updateHighscore() {
>           var newHighscore = NSUserDefaults.standardUserDefaults().integerForKey("highscore")
>           highscoreLabel.string = "\(newHighscore)"
>       }

We need to call this method in two situations:

*   When the app starts - to display the latest highscore
*   When the highscore gets updated

> [action]
> Add the following lines to *didLoadFromCCB* in *MainScene*:
>
>       NSUserDefaults.standardUserDefaults().addObserver(self, forKeyPath: "highscore", options: .allZeros, context: nil)
>       updateHighscore()

We are doing two things here. We observe the highscore, just as we observe the score value of the *Grid*. This means whenever the highscore stored in the *NSUserDefaults* changes this class will be notified (the *observeValueForKeyPath* will be called).

<!--TODO: explain KVO-->

The second thing we do is call the *updateHighscore* method directly from *didLoadFromCCB* to display the latest highscore once the app starts.

> [action]
> Now that we are observing a variable we need to add an *observeValueForKeyPath* method in *MainScene*:
>
>       override func observeValueForKeyPath(keyPath: String, ofObject object: AnyObject, change: [NSObject : AnyObject], context: UnsafeMutablePointer<Void>) {
>           if keyPath == "highscore" {
>               updateHighscore()
>           }
>       }

You can see that we are now reacting to changes of `score` and `highscore`. If the highscore changes we call *updateHighscore* and refresh the displayed highscore in the game.

Now you can run the new version of the game and see how the highscore is stored and displayed in the game:

![](./highscore.png)
