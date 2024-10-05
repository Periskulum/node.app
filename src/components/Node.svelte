<script>
  import { createEventDispatcher } from 'svelte';
  import { darkMode } from '../stores/darkMode.js';
  import { getNextZIndex } from '../stores/zIndex.js';

  export let x = 0;
  export let y = 0;
  export let label = '';
  export let id;
  export let isSelected = false;
  export let isFactoryNode = false;
  export let isNonFunctional = false;

  let isDragging = false;
  let startX, startY;
  let zIndex = getNextZIndex();

  const dispatch = createEventDispatcher();

  $: isDarkMode = $darkMode;

  function handleMouseDown(event) {
    if (isNonFunctional || isFactoryNode) return;
    if (event.button === 0) { // Left mouse button for regular nodes
      startDragging(event);
    }
  }

  function startDragging(event) {
    isDragging = true;
    startX = event.clientX - x;
    startY = event.clientY - y;
    zIndex = getNextZIndex();
    dispatch('select');
    event.target.setPointerCapture(event.pointerId);
  }

  function handleMouseMove(event) {
    if (isDragging) {
      const newX = event.clientX - startX;
      const newY = event.clientY - startY;
      x = newX;
      y = newY;
      dispatch('move', { id, x: newX, y: newY });
    }
  }

  function handleMouseUp(event) {
    if (isDragging) {
      isDragging = false;
      event.target.releasePointerCapture(event.pointerId);
    }
  }
</script>

<div
  class="node"
  class:dark-mode={isDarkMode}
  class:selected={isSelected}
  class:factory-node={isFactoryNode}
  class:non-functional={isNonFunctional}
  style="left: {isFactoryNode ? 0 : x}px; top: {isFactoryNode ? 0 : y}px; z-index: {zIndex};"
  on:pointerdown={handleMouseDown}
  on:pointermove={handleMouseMove}
  on:pointerup={handleMouseUp}
  on:pointercancel={handleMouseUp}
>
  {label}
</div>

<style>
  .node {
    position: absolute;
    background-color: #3498db;
    color: white;
    padding: 12px;
    border-radius: 8px;
    cursor: move;
    user-select: none;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    pointer-events: auto;
    touch-action: none;
    transition: box-shadow 0.3s ease, transform 0.3s ease;
    width: 80px;
    word-wrap: break-word;
  }

  .dark-mode {
    background-color: #2980b9;
    color: #eee;
  }

  .selected {
    box-shadow: 0 0 0 3px #27ae60, 0 2px 10px rgba(0, 0, 0, 0.2);
    transform: scale(1.05);
  }

  .node:hover {
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  }

  .factory-node {
    position: relative;
  }

  .non-functional {
    pointer-events: none;
  }
</style>