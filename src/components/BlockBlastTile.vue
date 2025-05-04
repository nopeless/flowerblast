<template>
  <div class="blockblast-block" :class="{ hovered }">
    <div :class="[tileType, { draggable }, { isTile: !!tileType }]" v-if="hidden !== true">

    </div>
  </div>
</template>

<script setup lang="ts">
const { tileType, hovered, hidden, draggable } = defineProps<{
  tileType: string | null;
  hovered?: boolean;
  hidden?: boolean;
  draggable?: boolean;
}>();

</script>

<style>
:root {
  --blockblast-tile-border-radius: 4px;
  --blockblast-tile-border-thickness: 5px;
}

.blockblast-block {
  width: 100%;
  height: 100%;
  flex: 1;
  aspect-ratio: 1 / 1;
  border-radius: var(--blockblast-tile-border-radius);
}

.blockblast-block.hovered {
  opacity: 0.4;
}

.blockblast-block > .draggable {
  cursor: grab;
}

.blockblast-block > * {
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.61);
}

.blockblast-block > .isTile {
  box-sizing: border-box;
  border-radius: 0;

  border-top: var(--blockblast-tile-border-thickness) solid rgba(0, 0, 0, 0.2);
  border-left: var(--blockblast-tile-border-thickness) solid rgba(0, 0, 0, 0.4);
  border-right: var(--blockblast-tile-border-thickness) solid rgba(0, 0, 0, 0.6);
  border-bottom: var(--blockblast-tile-border-thickness) solid rgba(0, 0, 0, 0.8);
}

.blockblast-block > .cherry-blossom-pink {
  background-color: #ffa8b8;
}

.blockblast-block > .mountbatten {
  background-color: #967AA1;
}

.blockblast-block > .cool-gray {
  background-color: #8699B8;
}

</style>
