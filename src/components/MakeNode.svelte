<script>
  import { createEventDispatcher } from 'svelte';
  import { getNextZIndex } from '../stores/zIndex.js';
  import { darkMode } from '../stores/darkMode.js';

  export let x = 0;
  export let y = 0;
  export let id;
  export let isSelected = false;
  export let isFactoryNode = false;
  export let isNonFunctional = false;

  let inputValue = '';
  const dispatch = createEventDispatcher();

  let isDragging = false;
  let startX, startY;
  let zIndex = getNextZIndex();

  $: isDarkMode = $darkMode;

  function handleSubmit() {
    if (!isNonFunctional && inputValue.trim()) {
      dispatch('createNode', { type: 'basic', label: inputValue.trim(), x: x + 200, y: y + 50 });
      inputValue = '';
    }
  }

  function handleKeydown(event) {
    if (!isNonFunctional && event.key === 'Enter') {
      handleSubmit();
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
  class="make-node"
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
  <span class="label">make.node</span>
  <div class="make-input-container">
    <input
      bind:value={inputValue}
      on:keydown={handleKeydown}
      placeholder="node.label"
      disabled={isNonFunctional}
    />
    <button class="add-btn" on:click={handleSubmit} disabled={isNonFunctional}>
      <span class="material-icons">add</span>
    </button>
  </div>
</div>

<style>
  .make-node {
    position: absolute;
    background-color: #2ecc71;
    color: white;
    padding: 12px;
    border-radius: 8px;
    cursor: move;
    user-select: none;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    pointer-events: auto;
    touch-action: none;
    transition: box-shadow 0.3s ease, transform 0.3s ease;
    width: 180px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .dark-mode {
    background-color: #27ae60;
  }

  .selected {
    box-shadow: 0 0 0 3px #3498db, 0 2px 10px rgba(0, 0, 0, 0.2);
    transform: scale(1.05);
  }

  .make-node:hover {
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  }

  .label {
    margin-bottom: 8px;
  }

  .make-input-container {
    display: flex;
    width: 100%;
    box-sizing: border-box;
  }

  input {
    flex-grow: 1;
    padding: 4px 8px;
    border: none;
    border-radius: 4px 0 0 4px;
    font-size: 14px;
    min-width: 0;
  }

  .add-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 0 4px 4px 0;
    padding: 4px 8px;
    cursor: pointer;
    font-size: 14px;
    min-width: 32px;
    flex-shrink: 0;
  }

  .material-icons {
    font-size: 18px;
  }

  input:disabled, button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .factory-node {
    position: relative;
  }

  .non-functional {
    pointer-events: none;
  }
</style>