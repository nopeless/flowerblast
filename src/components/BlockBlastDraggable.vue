<template>
  <div
    class="blockblast-draggable" 
    ref="draggableRef"
    :class="{ dragging: isActuallyDragging }"
    :style="[isDragging ? style : undefined, {
      width: isActuallyDragging ? `${draggedSideLength * SIDE_LENGTH}px` : undefined,
      height: isActuallyDragging ? `${draggedSideLength * SIDE_LENGTH}px` : undefined,
      transform: isActuallyDragging && dragCoordinates ? `translate(${-(dragCoordinates[0] + shape.left) / 2 * draggedSideLength}px, ${-(dragCoordinates[1] + shape.top) / 2 * draggedSideLength}px)` : undefined,
    }]"
    @mousemove="e => {
      emit('mousemove', e);
    }"
    @touchmove="e => {
      emit('touchmove', e);
    }"
    >
    <div
      v-for="(row, rowIndex) in shape.data"
      :key="rowIndex"
      class="blockblast-draggable-row"
    >
    <BlockBlastTile
      v-for="(tile, tileIndex) in row"
      :key="tileIndex"
      :tileType="selection.type"
      :hidden="tile === 0"
      :draggable="true"
      ></BlockBlastTile>
    </div>
  </div>
</template>

<script setup lang="ts">
import BlockBlastTile from './BlockBlastTile.vue';
import { computed, ref, watch } from 'vue';
import { useDraggable } from '@vueuse/core'

const { selection, draggedSideLength } = defineProps<{
  selection: {
    shape: number[][];
    type: string | null;
  };
  draggedSideLength: number;
}>();

const emit = defineEmits<{
  (e: 'coords', coordinates: [number, number] | null): void;
  (e: 'mousemove', event: MouseEvent): void;
  (e: 'touchmove', event: TouchEvent): void;
}>();

const SIDE_LENGTH = 5;

const shape = computed(() => {
  // Pad the shape to be square
  // calculate left and top offset
  const shapeWidth = selection.shape[0].length;
  const shapeHeight = selection.shape.length;
  const leftOffset = Math.floor((SIDE_LENGTH - shapeWidth) / 2);
  const topOffset = Math.floor((SIDE_LENGTH - shapeHeight) / 2);

  const paddedShape = Array.from({ length: SIDE_LENGTH }, () =>
    Array.from({ length: SIDE_LENGTH }, () => 0)
  );
  for (let i = 0; i < shapeHeight; i++) {
    for (let j = 0; j < shapeWidth; j++) {
      paddedShape[i + topOffset][j + leftOffset] = selection.shape[i][j];
    }
  }
  return { data: paddedShape, left: leftOffset, top: topOffset };
});

const draggableRef = ref(null! as HTMLDivElement);

const dragCoordinates = ref<[number, number] | null>(null);

watch(dragCoordinates, coords => {
  emit('coords', coords);
});

const isActuallyDragging = ref(false);

const { isDragging, style } = useDraggable(draggableRef, {
  // preventDefault: true,
  onStart: (event) => {
    const { x, y } = event;

    const row = x / (draggableRef.value.clientWidth / SIDE_LENGTH) - shape.value.left;
    const col = y / (draggableRef.value.clientHeight / SIDE_LENGTH) - shape.value.top;

    dragCoordinates.value = [row, col];
  },
  onEnd: () => {
    dragCoordinates.value = null;
  },
});

// workaround because useDraggable doesn't behave the way I want to
watch(isDragging, () => {
  if (!isDragging.value) isActuallyDragging.value = false;
});
watch(style, () => {
  isActuallyDragging.value = true;
})
</script>

<style>
.blockblast-draggable {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  /* background-color: rgba(255, 0, 0, 0.308); */
}

.blockblast-draggable.dragging {
  cursor: grabbing;
  position: fixed;
}

.blockblast-draggable-row {
  display: flex;
}
</style>
