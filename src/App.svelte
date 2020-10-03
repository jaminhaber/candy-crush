<script lang="ts">
  import { sample, sampleSize, times } from "lodash";

  // TYPES
  type Bubble = { color: string };
  type Coord = [number, number];

  // CONSTANTS
  const WIDTH = 10;
  const HEIGHT = 10;
  const NUM_MOVES = 10;
  const NUM_COLORS = 4;
  // prettier-ignore
  const COLORS = [ "#ffadad", "#ffd6a5", "#fdffb6", "#caffbf", "#9bf6ff", "#a0c4ff", "#bdb2ff", "#ffc6ff"];

  // VARIABLES
  let colors = sampleSize(COLORS, NUM_COLORS);
  let grid: Bubble[][] = [];
  let score = 0;
  let movesLeft = NUM_MOVES;

  // FUNCTIONS
  const onClickSquare = (coord: Coord) => {
    const nodes = dfs(coord);
    movesLeft -= 1;
    let num = 0;
    for (const [i, j] of nodes) {
      num += 1;
      score += num;
      grid[i][j].color = sample(colors);
    }
  };

  const serializeCoord = ([i, j]: Coord) => `${i}.${j}`;

  const dfs = ([i, j]: Coord): Coord[] => {
    const visited = new Set<string>();
    const { color } = grid[i][j];
    const stack: Coord[] = [[i, j]];

    const acc = [];
    while (stack.length !== 0) {
      const node = stack.pop();
      if (!visited.has(serializeCoord(node))) {
        visited.add(serializeCoord(node));
        const [row, col] = node;
        if (grid[row + 1]?.[col]?.color === color) stack.push([row + 1, col]);
        if (grid[row - 1]?.[col]?.color === color) stack.push([row - 1, col]);
        if (grid[row]?.[col + 1]?.color === color) stack.push([row, col + 1]);
        if (grid[row]?.[col - 1]?.color === color) stack.push([row, col - 1]);

        acc.push([row, col]);
      }
    }

    return acc;
  };

  const newGame = () => {
    score = 0;
    movesLeft = NUM_MOVES;
    colors = sampleSize(COLORS, NUM_COLORS);
    grid = times(HEIGHT, () => times(WIDTH, () => ({ color: sample(colors) })));
  };

  onMount: newGame();
</script>

<style>
  .grid {
    display: grid;
  }
  .cell {
    width: 2rem;
    height: 2rem;
  }

  main {
    padding: 2rem;
  }
</style>

<button class="reset" on:click={newGame}>New Game</button>

{#if movesLeft > 0}
  <main>
    <h2>score {score}</h2>
    <h4>moves left {movesLeft}</h4>

    {#each grid as row, i}
      <div
        class="grid"
        style={`grid-template-columns: repeat(${WIDTH}, 2rem);`}>
        {#each row as col, j}
          <div
            style={` background-color: ${col.color};`}
            on:click={() => onClickSquare([i, j])}
            class={`cell w2 h2 bg-${col.color} ba b--white`} />
        {/each}
      </div>
    {/each}
  </main>
{:else}
  <main>
    <h2>Game Over</h2>
    <h2>score {score}</h2>
  </main>
{/if}
