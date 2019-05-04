---
layout: post
title: "Ember: Conway's Game of Life"
date:   2015-09-22 08:13:39
categories: web breakable-toy ember
excerpt: "Life's a game with Ember!"
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/ember-conways-game-of-life)*

During my [apprenticeship], we were encouraged to keep a [breakable toy] for
trying out new concepts or developing a deeper understanding for the
framework/tools you are using. The basic idea is to have a project you don't
have to ship so you can spend time experimenting.

[apprenticeship]: http://www.apprentice.io/
[breakable toy]: http://redsquirrel.com/dave/work/a2j/patterns/BreakableToys.html

Recently I implemented [Conway's Game of Life] for the first time using Ember.
This exposed me to parts of Ember that I think are interesting to talk about.

Disclaimer: As this is a breakable toy, before I start I like to choose a
concept or addon that I haven't used before to see how it feels. For this I
chose [ember-computed-decorators] to try out.

[Conway's Game of Life]: https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life
[ember-computed-decorators]: https://github.com/rwjblue/ember-computed-decorators

## The game

Conway's Game of Life is a zero player game, meaning that it requires no input
from any players and the result can be determined from the initial setup. The
game is played on an infinite two-dimensional board made up of cells. A cell is
in one of two states, alive or dead. On a turn, each cell has its state change
depending on the states of its surrounding eight neighbours. All cells change
state at the same time. The next state is determined from the following rules:

1. If the cell is *alive* and has less than two living neighbours then the cell
   dies.
2. If the cell is *alive* and has two or three living neighbours then it remains
   alive.
3. If the cell is *alive* and has more than three living neighbours then the
   cell dies.
4. If the cell is *dead* and has exactly three live neighbours then the cell
   will come to life.

## The board

Let's get the most obvious problem out of the way first. How should we model a
infinite board? I decided to model my board as a [torus], which means the top
and bottom edges are considered to be attached as are the left and right sides.
This means that we have an infinite, 2-dimensional board that repeats every
certain number of cells.

[torus]: https://en.wikipedia.org/wiki/Torus#Flat_torus

We'll need a way to generate the board given a width and height at which the
universe repeats, and then tell each cell who its eight neighbours are.
Let's start with a `game-board` component:

```js
// app/components/game-board.js
import Ember from 'ember';
import computed from 'ember-computed-decorators';
import generateBoard from 'game-of-life/utils/board-generator';
import registerNeighbours from 'game-of-life/utils/register-neighbours';

export default Ember.Component.extend(Ember.Evented, {
  width: 50,
  height: 50,
  density: 0.2,

  @computed('width', 'height', 'density')
  board(width, height, density) {
    const board = generateBoard(width, height, density);
    registerNeighbours(board);

    return board;
  },

  actions: {
    step() {
      const board = this.get('board');
      board.forEach(function(row) {
        row.invoke('step');
      });
    },
  },
});
```

So I've started here with code I wish I had. My board is initialized with the
`generateBoard` function and then the neighbours for each cell needs to be
registered with that cell. Let's start by generating our board:

```js
// app/utils/board-generator.js
import Cell from 'game-of-life/models/cell';

export default function boardGenerator(width, height, density) {
  const board = [];
  for (let i = 0; i < height; i++) {
    const row = [];
    for (let j = 0; j < width; j++) {
      const alive = Math.random() < density;
      row.pushObject(Cell.create({alive}));
    }
    board.pushObject(row);
  }

  return board;
}
```

The above code loops over the height and width to create a matrix of cells.
We'll need a `Cell` model to contain the logic for working out the next state
according to the rules, but we'll come back to that later. Let's move on to
registering neighbours:

```js
// app/utils/register-neighbours.js
export default function registerNeighbours(board) {
  return board.map((row, rowIndex, board) => {
    return row.map((cell, cellIndex) => {
      const left = wrapAround(cellIndex - 1, row.length);
      const right = wrapAround(cellIndex + 1, row.length);
      const up = wrapAround(rowIndex - 1, row.length);
      const down = wrapAround(rowIndex + 1, row.length);

      const neighbours = [
        board[up][left],
        board[up][cellIndex],
        board[up][right],
        board[rowIndex][left],
        board[rowIndex][right],
        board[down][left],
        board[down][cellIndex],
        board[down][right]
      ];

      cell.set('neighbours', neighbours);
      return cell;
    });
  });
}

function wrapAround(index, length) {
  if (index === -1) {
    return length - 1;
  } else if (index === length) {
    return 0;
  } else {
    return index;
  }
}
```

This simply iterates over every cell on the board and finds the eight adjacent
cells. We'll use the `wrapAround` function to handle connecting our edges
together giving us our Torus.

This checks if we're trying to access a cell position that is out of bounds. If
so, we move to the beginning or end of the row/column where appropriate.

Finally, we have the template for our `game-board` component:

```hbs
{% raw %}
<!-- app/templates/components/game-board.hbs -->
{{#each board as |row|}}
  <div class='row'>
    {{#each row key="@index" as |cell|}}
      {{game-cell cell=cell}}
    {{/each}}
  </div>
{{/each}}

<p>
  <button {{action 'step'}}>STEP</button>
</p>
{% endraw %}
```

## The cells

Now that we have a board set up, let's build our `game-cell` component:

```js
// app/components/game-cell.js
import Ember from 'ember';
const {computed} = Ember;

export default Ember.Component.extend({
  cell: null,
  alive: computed.alias('cell.alive'),
});
```

Next we'll need a `cell` model. This should be aware of what its current state
is and what its next state should be based on the state of its neighbours:

```js
// app/models/cell.js
import Ember from 'ember';
import computed from 'ember-computed-decorators';

export default Ember.Object.extend(Ember.Evented, {
  alive: false,
  neighbours: [],

  @computed('neighbours.@each.alive')
  aliveNeighboursCount(neighbours) {
    return neighbours.filter(alive => alive).get('length');
  },

  @computed('alive', 'aliveNeighboursCount')
  nextState(alive, aliveNeighboursCount) {
    return (alive && aliveNeighboursCount === 2) ||
      (aliveNeighboursCount === 3);
  },

  step() {
    this.set('alive', this.get('nextState'));
  },
});
```

Here we use computed properties to keep a count of living neighbours, then we
use that to determine what the `nextState` should be. Finally, we've added a
`step` function that changes the alive property to the `nextState`.

Let's see how this runs:

![broken-game-of-life](https://images.thoughtbot.com/ember-conways-game-of-life/VXVOZUkS1G5gQ2B1vPYD_broken-gol.gif)

Something isn't right here.

## Ember run loop

Turns out, we've forgotten that all cells change state at the same time. Our
code that runs `row.invoke('step')` will iterate through each cell in the row
and tell it to move to the next state. This causes the state to change and
affect the neighbouring cells when they should be interested in the previous
state. To fix this we need to look at Ember's [run loop].

[run loop]: https://guides.emberjs.com/v1.10.0/understanding-ember/run-loop/

The run loop is used to schedule work into queues. Each iteration of the run
loop runs each queue in a particular order to ensure that operations are done in
the most efficient way possible. For example, there is a `routerTransitions`
queue which contains router transition jobs. This come just before the `render`
queue which is responsible for updating the DOM with the current state of the
application. If these queues were reversed, then it would take two iterations
of the run loop to move between pages instead of one.

The default queues are as follows: `sync`, `actions`, `routerTransitions`,
`render`, `afterRender` and finally `destroy`. We can put tasks into a specific
queue using [`Ember.run.schedule`]. Armed with this knowledge let's edit our
step function:

[`Ember.run.schedule`]: http://emberjs.com/api/classes/Ember.run.html#method_schedule

```js
// app/models/cell.js
...
  step() {
    const nextState = this.get('nextState');
    Ember.run.schedule('sync', this, function() {
      this.set('alive', nextState);
    });
  },
...
```

And now our applications should look like this:

![awesome-game-of-life](https://images.thoughtbot.com/ember-conways-game-of-life/YSSJmSBS9uAX92W2c6qQ_awesome-gol.gif)

That's better!

## Code

You can find the [repository here] and [here's the app in action].

[repository here]: https://github.com/MattMSumner/gol
[here's the app in action]: http://appallingfarrago.com/gol/
