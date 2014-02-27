# How to make a Flappy Bird in HTML5 with Phaser

Flappy Bird is a nice little game with easy to understand mechanics, and I
thought it would be a perfect fit for an HTML5 game tutorial. So in this 
tutorial we are going to make a simplified version of Flappy Bird, in only 65 
lines of Javascript.

[Click here][1] to test the game we are going to build.

![][2]

Note 1: you need to know some Javascript in order to understand this tutorial
.

Note 2: If you want to learn more about the Phaser framework and how to set up
your environment, you should read my short[getting started tutorial][3] first
. 


## Set up

You should download [this basic template][4] that I made for this tutorial. In
it you will find:

*   phaser.min.js, the minified Phaser framework v1.1.5.
*   index.html, where the game will be displayed.
*   main.js, a file where we will write all the code. 
*   assets/, a directory with 2 images (bird.png and pipe.png).

Put all of these files on your webserver. Open the index.html file in your
browser, and open the main.js file in your text editor.

In the main.js file you should see the basic structure of a Phaser project that
we discussed in the previous tutorial.

    // Initialize Phaser, and creates a 400x490px game
    var game = new Phaser.Game(400, 490, Phaser.AUTO, 'game_div');  
    var game_state = {};
    
    // Creates a new 'main' state that will contain the game
    game_state.main = function() { };  
    game_state.main.prototype = {
    
        preload: function() { 
            // Function called first to load all the assets
        },
    
        create: function() { 
            // Fuction called after 'preload' to setup the game    
        },
    
        update: function() {
            // Function called 60 times per second
        },
    };
    
    // Add and start the 'main' state to start the game
    game.state.add('main', game_state.main);  
    game.state.start('main');  
    

We are going to fill in the `preload()`, `create()` and `update()` functions,
and add some new functions to make the game work.


## The bird

Okay, so now you are ready to code! Let's first focus on adding a bird to the
game that can be controlled with the spacebar key.

Everything is quite simple and well commented, so you should be able to
understand easily the code below. For better readability, I removed the Phaser 
initialization and states management code that you can see above.

First we update the `preload()`, `create()` and `update()` functions.

    preload: function() {  
        // Change the background color of the game
        this.game.stage.backgroundColor = '#71c5cf';
    
        // Load the bird sprite
        this.game.load.image('bird', 'assets/bird.png'); 
    },
    
    create: function() {  
        // Display the bird on the screen
        this.bird = this.game.add.sprite(100, 245, 'bird');
    
        // Add gravity to the bird to make it fall
        this.bird.body.gravity.y = 1000;  
    
        // Call the 'jump' function when the spacekey is hit
        var space_key = this.game.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);
        space_key.onDown.add(this.jump, this);     
    },
    
    update: function() {  
        // If the bird is out of the world (too high or too low), call the 'restart_game' function
        if (this.bird.inWorld == false)
            this.restart_game();
    },
    

And just below this, add these two new functions.

    // Make the bird jump 
    jump: function() {  
        // Add a vertical velocity to the bird
        this.bird.body.velocity.y = -350;
    },
    
    // Restart the game
    restart_game: function() {  
        // Start the 'main' state, which restarts the game
        this.game.state.start('main');
    },
    

Save the main.js file with the new code, and reload the index.html file. You
should have a bird jumping when you press the spacebar key.


## The pipes

A Flappy Bird game without pipes is not really interesting, so let's change
that.

First, load the pipe sprite in the `preload()` function.

    this.game.load.image('pipe', 'assets/pipe.png');  
    

Then, create a group of pipes in the `create()` function. Since we are going to
handle a lot of pipes in the game, it's easier to use a group of sprites. The 
group will contain 20 sprites that we will be able to use as we want.

    this.pipes = game.add.group();  
    this.pipes.createMultiple(20, 'pipe');  
    

Now we need a new function to add a pipe in the game. By default, all the pipes
contained in the group are dead and not displayed. So we pick a dead pipe, 
display it, and make sure to automatically kill it when it's no longer visible. 
This way we never run out of pipes.

    add_one_pipe: function(x, y) {  
        // Get the first dead pipe of our group
        var pipe = this.pipes.getFirstDead();
    
        // Set the new position of the pipe
        pipe.reset(x, y);
    
        // Add velocity to the pipe to make it move left
        pipe.body.velocity.x = -200; 
    
        // Kill the pipe when it's no longer visible 
        pipe.outOfBoundsKill = true;
    },
    

The previous function displays one pipe, but we need to display 6 pipes in a
row with a hole somewhere in the middle. So let's create a new function that 
does just that.

    add_row_of_pipes: function() {  
        var hole = Math.floor(Math.random()*5)+1;
    
        for (var i = 0; i < 8; i++)
            if (i != hole && i != hole +1) 
                this.add_one_pipe(400, i*60+10);   
    },
    

To actually add pipes in the game, we need to call the `add_row_of_pipes()`
function every 1.5 seconds. We can do this by adding a timer in the`create()`
function.

    this.timer = this.game.time.events.loop(1500, this.add_row_of_pipes, this);  
    

And finally add this line of code a the beginning of the `restart_game()`
function to stop the timer when we restart the game.

    this.game.time.events.remove(this.timer);  
    

Now you can save your file and test the code. This is slowly starting to look
like a real game.


## Scoring and collisions

The last thing we need to finish the game is adding a score and handling
collisions. And this is quite easy to do.

Add this in the `create()` function to display a score label in the top left.

    this.score = 0;  
    var style = { font: "30px Arial", fill: "#ffffff" };  
    this.label_score = this.game.add.text(20, 20, "0", style);  
    

Put this in the `add_row_of_pipes()`, to increase the score by 1 each time new
pipes are created.

    this.score += 1;  
    this.label_score.content = this.score;  
    

And add this in the `update()` function to call `restart_game()` each time the
bird collides with a pipe.

    this.game.physics.overlap(this.bird, this.pipes, this.restart_game, null, this);  
    

And we are done! Congratulation, you now have an HTML5 Flappy Bird clone. You
can download the full source code[here][5]. 


## What's next?

The game is working, but it's a little bit boring. In the next tutorial we'll
see how we can make it better by adding sounds, animations, menus, etc. So if 
you haven't already, make sure to subscribe to the newsletter below to be 
notified when the next tutorial is out.

Enjoyed this article? Then please consider sharing it [on Twitter][6]. And you
can also follow me:[@lessmilk_][7].

And you can have a look at my "one HTML5 game per week" challenge on 
[lessmilk.com][8]



 [1]: http://www.lessmilk.com/flappy_bird/
 [2]: img/FB-1.png
 [3]: http://blog.lessmilk.com/make-html5-games-with-phaser-1/

 [4]: https://github.com/lessmilk/phaser-tutorials/blob/master/2-flappy_bird/basic_template.zip?raw=true

 [5]: https://github.com/lessmilk/phaser-tutorials/blob/master/2-flappy_bird/flappy_bird.zip?raw=true

 [6]: http://twitter.com/share?text=I%27m+reading+%22How+to+make+a+Flappy+Bird+clone+in+HTML5+with+Phaser%22+on&via=lessmilk_&url=http://blog.lessmilk.com/how-to-make-flappy-bird-in-html5-1/
 [7]: https://twitter.com/LessMilk_
 [8]: http://www.lessmilk.com
