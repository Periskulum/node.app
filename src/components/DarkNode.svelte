<script>
  import { createEventDispatcher } from 'svelte';
  import { darkMode } from '../stores/darkMode.js';
  import { getNextZIndex } from '../stores/zIndex.js';

  export let x = 0;
  export let y = 0;
  export let id;
  export let isSelected = false;
  export let isFactoryNode = false;
  export let isNonFunctional = false;

  const dispatch = createEventDispatcher();

  let isDragging = false;
  let startX, startY;
  let zIndex = getNextZIndex();

  function toggleDarkMode() {
    if (!isNonFunctional) {
      darkMode.update(mode => !mode);
    }
  }

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
  class="dark-node"
  class:selected={isSelected}
  class:factory-node={isFactoryNode}
  class:non-functional={isNonFunctional}
  style="left: {isFactoryNode ? 0 : x}px; top: {isFactoryNode ? 0 : y}px; z-index: {zIndex};"
  on:pointerdown={handleMouseDown}
  on:pointermove={handleMouseMove}
  on:pointerup={handleMouseUp}
  on:pointercancel={handleMouseUp}
>
  <span>dark.node</span>
  <button on:click={toggleDarkMode} disabled={isNonFunctional}>
    <span class="material-icons">
      {$darkMode ? 'light_mode' : 'dark_mode'}
    </span>
  </button>
</div>

<style>
  .dark-node {
    position: absolute;
    background-color: #34495e;
    color: white;
    padding: 12px 16px;
    border-radius: 8px;
    cursor: move;
    user-select: none;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    pointer-events: auto;
    touch-action: none;
    transition: box-shadow 0.3s ease, transform 0.3s ease;
    width: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .selected {
    box-shadow: 0 0 0 3px #3498db, 0 2px 10px rgba(0, 0, 0, 0.2);
    transform: scale(1.05);
  }

  .dark-node:hover {
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  }

  button {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 8px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 4px;
    cursor: pointer;
    font-size: 14px;
    width: 32px;
    height: 32px;
  }

  .material-icons {
    font-size: 18px;
  }

  .factory-node {
    position: relative;
  }

  .non-functional {
    pointer-events: none;
  }
</style>