# 教程

本教程将引导您完成一个简单的井字棋游戏。

?> 我们将从终端运行命令并使用 Node.js/npm。如果你之前没有做过这些，您可能需要先阅读[命令行入门指南][cmd]并按照[如何安装 Node 的说明操作][node]。你还需要一个文本编辑器来编写代码，比如 [VS Code][vsc] 或 [Atom][atom]。

[node]: https://nodejs.dev/learn/how-to-install-nodejs
[cmd]: https://tutorial.djangogirls.org/en/intro_to_command_line/
[vsc]: https://code.visualstudio.com/
[atom]: https://atom.io/



## 设置

我们将使用 ES2015 的特性，比如模块的[导入](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)和[对象展开](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)语法，因此我们需要使用某种构建系统来为浏览器编译我们的代码。

本教程展示了两种不同的方法: 一种是使用 [React](https://reactjs.org/)，另一种是使用基本的浏览器 API，并通过 [Parcel](https://parceljs.org/) 编译我们的应用。
你可以选择您感觉更舒服的方式。

<!-- tabs:start -->

### **纯 JS**

让我们从命令行创建一个新的 Node 项目:

```
mkdir bgio-tutorial
cd bgio-tutorial
npm init --yes
```

?> 这些命令将创建一个名为 `bgio-tutorial` 的新目录，切换到该目录，并初始化一个新的 Node 包。[在 Node 包管理器文档中了解更多。][pkgjson]

[pkgjson]: https://docs.npmjs.com/creating-a-package-json-file#creating-a-default-packagejson-file

我们将添加 boardgame.io 和 Parcel 来帮助我们构建应用：

```
npm install boardgame.io
npm install --save-dev parcel-bundler
```


现在，让我们创建项目所需的基本结构:


1. 一个用于我们 Web 应用的 JavaScript 文件位于 `src/App.js`。


2. 一个用于我们游戏定义的 JavaScript 文件位于 `src/Game.js`。


3. 一个基本的 HTML 页面，用于加载我们的应用位于 `index.html`：

    ```html
    <!DOCTYPE html>
    <html>
    <head>
      <title>boardgame.io 教程</title>
      <meta charset="utf-8" />
    </head>
    <body>
      <div id="app"></div>
      <script src="./src/App.js"></script>
    </body>
    </html>
    ```

你的项目目录现在应该如下所示:

    bgio-tutorial/
    ├── index.html
    ├── node_modules/
    ├── package-lock.json
    ├── package.json
    └── src/
        ├── App.js
        └── Game.js

看起来不错？好的，让我们开始吧！🚀

?> 你可以查看本教程的完整代码，并在 CodeSandbox 上试玩：<br/><br/>
[![Edit bgio-plain-js-tutorial](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/bgio-plain-js-tutorial-ewyyt?fontsize=14&hidenavigation=1&module=%2Fsrc%2FApp.js&theme=dark)

### **React**

我们将使用 [create-react-app](https://create-react-app.dev/) 命令行工具来初始化我们的 React 应用程序, 然后将 boardgame.io 添加到其中。

```
npx create-react-app bgio-tutorial
cd bgio-tutorial
npm install boardgame.io
```

既然我们在这里，让我们为游戏代码也创建一个空的 JavaScript 文件：

```
touch src/Game.js
```

?> 您可以查看本教程的完整代码，并在 CodeSandbox 上试玩：<br/><br/>
[![Edit boardgame.io](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/boardgameio-wlvi2)

<!-- tabs:end -->



## 定义游戏

我们通过创建一个对象来定义游戏，该对象的内容告诉 boardgame.io 你的游戏是如何运行的。或多或少一切都是可选的，所以我们可以从简单开始，逐步增加复杂性。首先，我们将添加一个 `setup` 函数，该函数将设置游戏状态 `G` 的初始值，以及一个包含构成游戏动作的 `moves(移动)` 对象。

移动是将 `G` 更新到所需新状态的函数。它接收一个包含多个字段的对象作为第一个参数。这个对象包括游戏状态 `G` 和 `ctx` ——一个由 boardgame.io 管理的对象，包含游戏元数据。它还包括 `playerID`，用于识别做出移动的玩家。在包含 `G` 和 `ctx` 的对象之后，moves 可以接收你在移动时传入的任意参数。

在井字棋中，我们只有一种移动类型，我们将其命名为 `clickCell`。它将获取被点击的单元格的 ID，并用点击该单元格的玩家 ID 更新该单元格。

让我们把这些放在 `src/Game.js` 文件中，开始定义我们的游戏：

```js
export const TicTacToe = {
  setup: () => ({ cells: Array(9).fill(null) }),

  moves: {
    clickCell: ({ G, playerID }, id) => {
      G.cells[id] = playerID;
    },
  },
};
```

?> `setup` 函数同样可以接收一个对象作为其第一个参数，比如移动。如果你需要根据 `ctx` 中的某个字段定制初始状态——例如玩家数量——但井字棋不需要这些。


## 创建客户端

<!-- tabs:start -->

### **纯 JS**

我们将首先在 `src/App.js` 中创建一个类来管理我们的 web 应用的逻辑。

在类的构造函数中，我们将创建一个 boardgame.io 客户端，并调用其 `start` 方法来运行它。

```js
import { Client } from 'boardgame.io/client';
import { TicTacToe } from './Game';

class TicTacToeClient {
  constructor() {
    this.client = Client({ game: TicTacToe });
    this.client.start();
  }
}

const app = new TicTacToeClient();
```

Let’s also add a script to `package.json` to make serving the web app simpler
and a [browserslist string](https://github.com/browserslist/browserslist) to
indicate the browsers we want to support:

```json
{
  "scripts": {
    "start": "parcel index.html --open"
  },
  "browserslist": "defaults and supports async-functions"
}
```
?> By dropping support for browsers that don’t support async functions, we don’t
   need to worry about including the `regenerator-runtime` polyfill. If you need to
   support older browsers, you can skip adding `browserslist`, but may need to
   include the polyfill manually.

You can now serve the app from the command line by running:

```
npm start
```

### **React**

Replace the contents of `src/App.js` with

```js
import { Client } from 'boardgame.io/react';
import { TicTacToe } from './Game';

const App = Client({ game: TicTacToe });

export default App;
```

You can now serve the app from the command line by running:

```
npm start
```

<!-- tabs:end -->

Although we haven’t built any UI yet, boardgame.io renders a Debug Panel.
This panel means we can already play our Tic-Tac-Toe game!

You can make a move by clicking on `clickCell` on the
Debug Panel, entering a number between `0` and `8`, and pressing
**Enter**. The current player will make a move on the chosen
cell. The number you enter is the `id` passed to the `clickCell` function as
the first argument after `G` and `ctx`. Notice how the
`cells` array on the Debug Panel updates as you make moves. You
can end the turn by clicking `endTurn` and pressing **Enter**. The next call to
`clickCell` will result in a “1” in the chosen cell instead of a “0”.

```react
<iframe class='react' src='snippets/example-1' height='760' scrolling='no' title='example' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'></iframe>
```


?> You can turn off the Debug Panel by passing `debug: false`
in the `Client` config.



## Game Improvements

### Validating Moves

So far, if a player calls `clickCell` for a cell that is already filled,
it will be overwritten. Let’s prevent that by updating `clickCell`
to let us know that a move is invalid if the selected cell isn’t `null`.

Moves can let the framework know they are invalid by returning a
special constant which we import into `src/Game.js`:

```js
import { INVALID_MOVE } from 'boardgame.io/core';
```

Now we can return `INVALID_MOVE` from `clickCell`:

```js
clickCell: ({ G, playerID }, id) => {
  if (G.cells[id] !== null) {
    return INVALID_MOVE;
  }
  G.cells[id] = playerID;
}
```

### Managing Turns

In the Debug Panel we clicked `endTurn` to pass the turn
to the next player after making a move. We could do this from our
client code too: make a move, then end the turn. This could be flexible
because a player could choose when to end their turn, but in
Tic-Tac-Toe we know that the turn should always end when a move is made.

There are several different ways to manage turns in boardgame.io.
We’ll use the `maxMoves` option in our game definition to tell
the framework to automatically end a player’s turn after a single
move has been made, as well as the `minMoves` option, so players
*have* to make a move and can't just `endTurn`.

```js
export const TicTacToe = {
  setup: () => { /* ... */ },

  turn: {
    minMoves: 1,
    maxMoves: 1,
  },

  moves: { /* ... */ },
}
```

?> You can learn more in the [Turn Order](turn-order.md)
    and [Events](events.md) guides.

### Victory Condition

The Tic-Tac-Toe game we have so far doesn't really ever end.
Let's keep track of a winner in case one player wins the game.

First, let’s declare two helper functions in `src/Game.js`
to test the `cells` array with:

```js
// Return true if `cells` is in a winning configuration.
function IsVictory(cells) {
  const positions = [
    [0, 1, 2], [3, 4, 5], [6, 7, 8], [0, 3, 6],
    [1, 4, 7], [2, 5, 8], [0, 4, 8], [2, 4, 6]
  ];

  const isRowComplete = row => {
    const symbols = row.map(i => cells[i]);
    return symbols.every(i => i !== null && i === symbols[0]);
  };

  return positions.map(isRowComplete).some(i => i === true);
}

// Return true if all `cells` are occupied.
function IsDraw(cells) {
  return cells.filter(c => c === null).length === 0;
}
```

Now, we add an `endIf` method to our game.
This method will be called each time our state updates to
check if the game is over.

```js
export const TicTacToe = {
  // setup, moves, etc.

  endIf: ({ G, ctx }) => {
    if (IsVictory(G.cells)) {
      return { winner: ctx.currentPlayer };
    }
    if (IsDraw(G.cells)) {
      return { draw: true };
    }
  },
};
```

?> `endIf` takes a function that determines if
the game is over. If it returns anything at all, the game ends and
the return value is available at `ctx.gameover`.



## Building a Board

<!-- tabs:start -->

### **Plain JS**

You can build your game board with your preferred UI tools.
This example will use basic JavaScript, but you should be able
to adapt this approach to many other frameworks.

To start with, let’s add a `createBoard` method to our
`TicTacToeClient` and call it in the constructor. This will inject
the required DOM structure for our board into the web page.
To know where to insert our board UI, we’ll pass in an
element when instantiating the class.

We’ll also add an `attachListeners` method. This will
set up our board cells so that they trigger the `clickCell`
move when they are clicked.

```js
class TicTacToeClient {
  constructor(rootElement) {
    this.client = Client({ game: TicTacToe });
    this.client.start();
    this.rootElement = rootElement;
    this.createBoard();
    this.attachListeners();
  }

  createBoard() {
    // Create cells in rows for the Tic-Tac-Toe board.
    const rows = [];
    for (let i = 0; i < 3; i++) {
      const cells = [];
      for (let j = 0; j < 3; j++) {
        const id = 3 * i + j;
        cells.push(`<td class="cell" data-id="${id}"></td>`);
      }
      rows.push(`<tr>${cells.join('')}</tr>`);
    }

    // Add the HTML to our app <div>.
    // We’ll use the empty <p> to display the game winner later.
    this.rootElement.innerHTML = `
      <table>${rows.join('')}</table>
      <p class="winner"></p>
    `;
  }

  attachListeners() {
    // This event handler will read the cell id from a cell’s
    // `data-id` attribute and make the `clickCell` move.
    const handleCellClick = event => {
      const id = parseInt(event.target.dataset.id);
      this.client.moves.clickCell(id);
    };
    // Attach the event listener to each of the board cells.
    const cells = this.rootElement.querySelectorAll('.cell');
    cells.forEach(cell => {
      cell.onclick = handleCellClick;
    });
  }
}

const appElement = document.getElementById('app');
const app = new TicTacToeClient(appElement);
```

You probably won’t see anything just yet, because all the cells are empty.
Let’s fix that by adding a style for the cells to `index.html`:

```html
<style>
  .cell {
    border: 1px solid #555;
    width: 50px;
    height: 50px;
    line-height: 50px;
    text-align: center;
  }
</style>
```

Now you should see an empty Tic-Tac-Toe board!
But there’s still one thing missing. If you click
on the board cells, you should see `G.cells` update
in the Debug Panel, but the board itself doesn’t change.
We need to add a way to refresh the board every time
boardgame.io’s state changes.

Let’s do that by writing an `update` method for our `TicTacToeClient`
class and subscribing to the boardgame.io state:

```js
class TicTacToeClient {
  constructor() {
    // As before, but we also subscribe to the client:
    this.client.subscribe(state => this.update(state));
  }

  createBoard() { /* ... */ }

  attachListeners() { /* ... */ }

  update(state) {
    // Get all the board cells.
    const cells = this.rootElement.querySelectorAll('.cell');
    // Update cells to display the values in game state.
    cells.forEach(cell => {
      const cellId = parseInt(cell.dataset.id);
      const cellValue = state.G.cells[cellId];
      cell.textContent = cellValue !== null ? cellValue : '';
    });
    // Get the gameover message element.
    const messageEl = this.rootElement.querySelector('.winner');
    // Update the element to show a winner if any.
    if (state.ctx.gameover) {
      messageEl.textContent =
        state.ctx.gameover.winner !== undefined
          ? 'Winner: ' + state.ctx.gameover.winner
          : 'Draw!';
    } else {
      messageEl.textContent = '';
    }
  }
}
```

Here are the key things to remember:

- You can trigger the moves defined in your game definition
  by calling `client.moves['moveName']`.


- You can register callbacks for every state change using `client.subscribe`.

### **React**

React can be a good fit for board games because
it provides a declarative API to translate objects
to UI elements. To create a board we need to translate
the game state `G` into actual cells that are clickable.

Let’s create a new file at `src/Board.js`:

```js
import React from 'react';

export function TicTacToeBoard({ ctx, G, moves }) {
  const onClick = (id) => moves.clickCell(id);

  let winner = '';
  if (ctx.gameover) {
    winner =
      ctx.gameover.winner !== undefined ? (
        <div id="winner">Winner: {ctx.gameover.winner}</div>
      ) : (
        <div id="winner">Draw!</div>
      );
  }

  const cellStyle = {
    border: '1px solid #555',
    width: '50px',
    height: '50px',
    lineHeight: '50px',
    textAlign: 'center',
  };

  let tbody = [];
  for (let i = 0; i < 3; i++) {
    let cells = [];
    for (let j = 0; j < 3; j++) {
      const id = 3 * i + j;
      cells.push(
        <td key={id}>
          {G.cells[id] ? (
            <div style={cellStyle}>{G.cells[id]}</div>
          ) : (
            <button style={cellStyle} onClick={() => onClick(id)} />
          )}
        </td>
      );
    }
    tbody.push(<tr key={i}>{cells}</tr>);
  }

  return (
    <div>
      <table id="board">
        <tbody>{tbody}</tbody>
      </table>
      {winner}
    </div>
  );
}
```

The important bit to pay attention to is about how to
dispatch moves. We have the following code in our click
handler:

```js
moves.clickCell(id);
```

- `moves` is passed in your component’s props by the framework and
  contains functions to dispatch your game’s moves. `props.moves.clickCell`
  dispatches the `clickCell` move, and any data passed in is made
  available in the move handler.


Now, we pass the board component to our `Client` in `src/App.js`:

```js
import { TicTacToeBoard } from './Board';

const App = Client({
  game: TicTacToe,
  board: TicTacToeBoard,
});

export default App;
```

<!-- tabs:end -->

And there you have it. A basic tic-tac-toe game!

```react
<iframe class='react' src='snippets/example-2' height='760' scrolling='no' title='example' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'></iframe>
```

?> You can press <kbd>1</kbd> (or click on the button next to “reset”)
   to reset the state of the game and start over.



## Bots

In this section we will show you how to add a bot that is
capable of playing your game. We need to tell the
bot what moves are allowed in the game, and it will find moves that
tend to produce winning results.

To do this, add an `ai` section to the game definition.
The `enumerate` function should return an array of possible
moves, so in our case it returns a `clickCell` move for every
empty cell.

```js
export const TicTacToe = {
  // setup, turn, moves, endIf ...

  ai: {
    enumerate: (G, ctx) => {
      let moves = [];
      for (let i = 0; i < 9; i++) {
        if (G.cells[i] === null) {
          moves.push({ move: 'clickCell', args: [i] });
        }
      }
      return moves;
    },
  },
};
```

That's it! Now that you can visit the AI section of the Debug Panel:

- `play` causes the bot to calculate and make a single move
  (shortcut: <kbd>2</kbd>)

- `simulate` causes the bot to play the entire game by itself
  (shortcut: <kbd>3</kbd>)

`play` helps you combine moves that you make yourself
and bot moves. For example, you can make
some manual moves to get two in a row and then verify that
the bot makes a block.

```react
<iframe class='react' src='snippets/example-3' height='760' scrolling='no' title='example' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'></iframe>
```

?> The bot uses [MCTS](https://nicolodavis.com/blog/tic-tac-toe/) under the
hood to explore the game tree and find good moves. The default uses
1000 iterations per move.  This can be configured to adjust the
bot's playing strength.

The framework will come bundled with a few different bot algorithms, and an advanced
version of MCTS that will allow you to specify a set of objectives to optimize for.
For example, at any given point in the game you can tell the bot to gather resources
in the short term and wage wars in the late stages. You just tell the bot what to do
and it will figure out the right combination of moves to make it happen!

Detailed documentation about all this is coming soon. Adding bots to games for actual
networked play (as opposed to merely simulating moves) is also in the works.
