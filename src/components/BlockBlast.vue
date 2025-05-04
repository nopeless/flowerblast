<template>

  <div
    class="blockblast-gameover-splash"
    v-if="gameOver"
  >
    <span>Game Over</span>
    <span>{{ score }}</span>
    <button
      class="blockblast-restart"
      @click="() => {
        gameOver = false;
        score = 0;
        resetGrid();
        regenerateSelection();
      }"
    >
      ↺
    </button>
  </div>

  <div class="blockblast-top-bar">
    <span>{{ score }}</span>
    <button
      class="blockblast-restart"
      @click="() => {
        score = 0;
        resetGrid();
        regenerateSelection();
      }"
    >
      ↺
    </button>
  </div>

  <div
    class="blockblast-grid"
    ref="gridRef"
    >
    <div
      v-for="(row, rowIndex) in grid"
      :key="rowIndex"
      class="blockblast-grid-row"
    >
      <BlockBlastTile
        v-for="(tile, tileIndex) in row"
        :key="tileIndex"
        :tileType="tile ?? hoverGrid[rowIndex][tileIndex]"
        :hovered="!!hoverGrid[rowIndex][tileIndex]"
      />
    </div>
  </div>

  <div style="height: 30px;"></div>

  <div class="blockblast-selection-bar">
    <div
      v-for="(selection, index) in selections"
      :key="index"
      class="blockblast-selection"
    >
      <div>
        <BlockBlastDraggable 
        :draggedSideLength="draggedSideLength" 
        :selection="selection"
        v-if="selection"
        @coords="coords => {
          if (coords === null) {
            if (isValidPlacement) {
              for (let i = 0; i < hoverGrid.length; i++) {
                for (let j = 0; j < hoverGrid[i].length; j++) {
                  if (hoverGrid[i][j] !== null) {
                    grid[i][j] = hoverGrid[i][j];
                  }
                }
              }

              const placed = selections[index];

              selections[index] = null;

              placedObject(placed);
            }

            hoverGrid = NULL_GRID;
            selectedInfo = null;

            return;
          }

          selectedInfo = {
            index,
            position: coords,
          };
        }"
        @mousemove="e => {
          // debugger;
          const rect = gridRef.getBoundingClientRect();
          const x = e.clientX;
          const y = e.clientY;

          const col = (x - rect.left) / (rect.width / WIDTH);
          const row = (y - rect.top) / (rect.height / HEIGHT);

          hoverCoordinates = [col, row];
        }"
        @touchmove="e => {
          const rect = gridRef.getBoundingClientRect();
          const x = e.touches[0].clientX;
          const y = e.touches[0].clientY;

          const col = (x - rect.left) / (rect.width / WIDTH);
          const row = (y - rect.top) / (rect.height / HEIGHT);

          hoverCoordinates = [col, row];
        }"
      />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, computed } from 'vue';
import BlockBlastTile from './BlockBlastTile.vue';
import BlockBlastDraggable from './BlockBlastDraggable.vue';

const WIDTH = 8;
const HEIGHT= 8;

const SELECTION_COUNT = 3;

const hoverCoordinates = ref(null as [number, number] | null);
const gridRef = ref(null! as HTMLDivElement);

const gameOver = ref(false);

function getRandomType() {
  const types = [
    'cherry-blossom-pink',
    'mountbatten',
    'cool-gray',
  ];

  return types[Math.floor(Math.random() * types.length)];
}

function getRandomShape() {
  const shapes = [
    // O
    [
      [1, 1],
      [1, 1],
    ],
    [
      [1, 1],
      [1, 1],
    ],
    [
      [1, 1],
      [1, 1],
    ],
    [
      [1, 1],
      [1, 1],
    ],
    // T
    [
      [1, 1, 1],
      [0, 1, 0],
    ],
    [
      [0, 1, 0],
      [1, 1, 1],
    ],
    [
      [0, 1],
      [1, 1],
      [0, 1],
    ],
    [
      [1, 0],
      [1, 1],
      [1, 0],
    ],
    // Z/S
    [
      [1, 1, 0],
      [0, 1, 1],
    ],
    [
      [0, 1, 1],
      [1, 1, 0],
    ],
    [
      [0, 1],
      [1, 1],
      [1, 0],
    ],
    [
      [1, 0],
      [1, 1],
      [0, 1],
    ],
    // I
    [
      [1, 1, 1, 1],
    ],
    [
      [1],
      [1],
      [1],
      [1],
    ],
    // L
    [
      [1, 0],
      [1, 0],
      [1, 1],
    ],
    [
      [1, 1, 1],
      [1, 0, 0],
    ],
    [
      [1, 1],
      [0, 1],
      [0, 1],
    ],
    [
      [0, 0, 1],
      [1, 1, 1],
    ],
    // J
    [
      [0, 1],
      [0, 1],
      [1, 1],
    ],
    [
      [1, 1, 1],
      [0, 0, 1],
    ],
    [
      [1, 0],
      [1, 0],
      [1, 1],
    ],
    [
      [1, 0, 0],
      [1, 1, 1],
    ],
    // Helper pieces
    [
      [1],
    ],
    [
      [1],
    ],
    [
      [1, 1],
    ],
    [
      [1],
      [1],
    ]
  ]

  return shapes[Math.floor(Math.random() * shapes.length)];
}

const score = ref(0);

const selections = ref<
  ({
    shape: number[][];
    type: string | null;
  } | null)[]
>(
  new Array(SELECTION_COUNT).fill(0).map(() => ({
    shape: getRandomShape(),
    type: getRandomType(),
  }))
);

function placedObject(selection: {
  shape: number[][];
  type: string | null;
} | null) {
  if (!selection) return;

  score.value += 1;

  if (selections.value.every(selection => selection === null)) {
    regenerateSelection();
  }

  // check if there are blocks in a row or column
  // check rows
  for (let i = 0; i < HEIGHT; i++) {
    const row = grid.value[i];
    if (row.every(tile => !!tile)) {
      score.value += WIDTH;
      grid.value[i] = new Array(WIDTH).fill(null);
    }
  }
  // check columns
  for (let i = 0; i < WIDTH; i++) {
    const column = grid.value.map(row => row[i]);
    if (column.every(tile => !!tile)) {
      score.value += HEIGHT;
      for (let j = 0; j < HEIGHT; j++) {
        grid.value[j][i] = null;
      }
    }
  }

  // check for game over
  if (!canPlaceAnyOfSelections()) {
    gameOver.value = true;
  }
}

function canPlaceAnyOfSelections() {
  console.log('chcking if can place any of selections');
  for (let i = 0; i < selections.value.length; i++) {
    const selection = selections.value[i];
    if (!selection) continue;

    const shape = selection.shape;

    // loop through the grid
    for (let x = 0; x < HEIGHT; x++) {
      for (let y = 0; y < WIDTH; y++) {
        if ((() => {
          for (let i = 0; i < shape.length; i++) {
            for (let j = 0; j < shape[i].length; j++) {
              if (shape[i][j] === 1) {
                const newX = x + i;
                const newY = y + j;
  
                if (newX >= HEIGHT || newY >= WIDTH || grid.value[newX][newY] !== null) {
                  return false;
                }
              }
            }
          }
          return true;
        })()) return true;
      }
    }
  }

  return false;
}

function resetGrid() {
  grid.value = new Array(WIDTH).fill(0).map(() => new Array(HEIGHT).fill(null as string | null));
}

function regenerateSelection() {
  selections.value = 
    new Array(SELECTION_COUNT).fill(0).map(() => ({
      shape: getRandomShape(),
      type: getRandomType(),
    }));
}

regenerateSelection();

const selectedInfo = ref(null as {
  index: number;
  /**
   * Includes sub-tile positioning
   */
  position: [number, number];
} | null);

// Store tile types which is a string
const grid = ref(new Array(WIDTH).fill(0).map(() => new Array(HEIGHT).fill(null as string | null)));

const draggedSideLength = computed(() => {
  if (!gridRef.value) return 0;
  return gridRef.value.getBoundingClientRect().width / WIDTH;
});

const NULL_GRID = new Array(WIDTH).fill(0).map(() => new Array(HEIGHT).fill(null as string | null));

const hoverGrid = ref(NULL_GRID);

watch(hoverCoordinates, () => {
  if (!selectedInfo.value) return hoverGrid.value = NULL_GRID;
  if (!hoverCoordinates.value) return hoverGrid.value = NULL_GRID;

  const selectedMeta = selections.value[selectedInfo.value.index];
  if (!selectedMeta) return hoverGrid.value = NULL_GRID;

  const tile = selectedInfo.value.position;

  const originTileX = Math.round(hoverCoordinates.value[1] - tile[1]);
  const originTileY = Math.round(hoverCoordinates.value[0] - tile[0]);

  const shape = selectedMeta.shape;

  const tempGrid = new Array(WIDTH).fill(0).map(() => new Array(HEIGHT).fill(null as string | null));

  // check if collision
  for (let i = 0; i < shape.length; i++) {
    for (let j = 0; j < shape[i].length; j++) {
      if (shape[i][j] === 1) {
        const x = originTileX + i;
        const y = originTileY + j;

        if (x < 0 || x >= HEIGHT || y < 0 || y >= WIDTH) {
          return hoverGrid.value = NULL_GRID;
        }

        if (grid.value[x][y] !== null) {
          return hoverGrid.value = NULL_GRID;
        }

        tempGrid[x][y] = selectedMeta.type;
      }
    }
  }

  return hoverGrid.value = tempGrid;
});

const isValidPlacement = computed(() => {
  if (hoverCoordinates.value === null) return false;
  return hoverGrid.value.some(row => row.some(tile => tile !== null));
})

</script>


<style> 
:root {
  --blockblast-grid-gap: 2px;
  --blockblast-grid-border-radius: 4px;
}
.blockblast-gameover-splash {
  z-index: 100;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  width: 500px;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 10px;
  background-color: rgba(0, 0, 0, 0.8); /* Dark shade around the overlay */
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
}

.blockblast-gameover-splash > button {
  background-color: var(--primary);
  font-size: 2.7rem;
  border: none;
  border-radius: 4px;
  padding: 5px;
  cursor: pointer;
  padding: 0 15px 8px 15px;
}

.blockblast-gameover-splash > span {
  font-size: 3rem;
  font-weight: bold;
  color: var(--text);
}

.blockblast-top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: var(--background);
}

.blockblast-top-bar > span {
  font-size: 3rem;
  font-weight: bold;
}

.blockblast-top-bar > .blockblast-restart {
  background-color: var(--primary);
  font-size: 2.7rem;
  border: none;
  border-radius: 4px;
  padding: 5px;
  cursor: pointer;
  padding: 0 15px 8px 15px;
}

.blockblast-grid {
  position: relative;
  aspect-ratio: 1 / 1;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  gap: var(--blockblast-grid-gap);
}

.blockblast-grid-row {
  display: flex;
  flex-shrink: 1;
  flex: 1;
  gap: var(--blockblast-grid-gap);
}

.blockblast-selection-bar {
  display: flex;
}

.blockblast-selection {
  flex: 1;
  width: 100%;
  height: 100%;
  aspect-ratio: 1 / 1;
  border-radius: var(--blockblast-grid-border-radius);
  /* background-color: var(--secondary); */
}

.blockblast-selection > * {
  padding: 5px;
  width: 100%;
  height: 100%;
}
</style>
