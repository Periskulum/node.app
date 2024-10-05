<script>
  import { onMount, afterUpdate, createEventDispatcher } from 'svelte';
  import SearchBar from './SearchBar.svelte';
  import { darkMode } from '../stores/darkMode.js';

  export let searchableNodes = [];

  const dispatch = createEventDispatcher();

  let canvas;
  let ctx;
  let width;
  let height;
  let panX = 0;
  let panY = 0;
  let isPanning = false;
  let startPanX, startPanY;

  $: isDarkMode = $darkMode;

  onMount(() => {
    ctx = canvas.getContext('2d');
    resizeCanvas();
    window.addEventListener('resize', resizeCanvas);
    window.addEventListener('mousemove', handleGlobalMouseMove);
    window.addEventListener('mouseup', handleGlobalMouseUp);
    return () => {
      window.removeEventListener('resize', resizeCanvas);
      window.removeEventListener('mousemove', handleGlobalMouseMove);
      window.removeEventListener('mouseup', handleGlobalMouseUp);
    };
  });

  afterUpdate(() => {
    drawGrid();
  });

  function resizeCanvas() {
    width = window.innerWidth;
    height = window.innerHeight;
    canvas.width = width;
    canvas.height = height;
    drawGrid();
  }

  function drawGrid() {
    if (!ctx) return;
    
    ctx.clearRect(0, 0, width, height);
    ctx.fillStyle = isDarkMode ? '#1a1a1a' : '#f0f0f0';
    ctx.fillRect(0, 0, width, height);

    const dotSize = 2;
    const spacing = 20;
    const dotColor = isDarkMode ? 'rgba(255, 255, 255, 0.3)' : 'rgba(0, 0, 0, 0.3)';

    for (let x = panX % spacing; x < width; x += spacing) {
      for (let y = panY % spacing; y < height; y += spacing) {
        ctx.fillStyle = dotColor;
        ctx.fillRect(x, y, dotSize, dotSize);
      }
    }
  }

  function handleMouseDown(event) {
    if (event.button === 1) { // Middle mouse button
      isPanning = true;
      startPanX = event.clientX - panX;
      startPanY = event.clientY - panY;
    } else if (event.button === 0) { // Left mouse button
      dispatch('canvasClick');
    }
  }

  function handleGlobalMouseMove(event) {
    if (isPanning) {
      panX = event.clientX - startPanX;
      panY = event.clientY - startPanY;
      drawGrid();
      updateNodesPosition();
    }
  }

  function handleGlobalMouseUp() {
    isPanning = false;
  }

  function handleNodeSelect(event) {
    const { x, y } = event.detail;
    panX = width / 2 - x;
    panY = height / 2 - y;
    drawGrid();
    updateNodesPosition();
  }

  function updateNodesPosition() {
    const nodesContainer = document.querySelector('.nodes');
    if (nodesContainer) {
      nodesContainer.style.transform = `translate(${panX}px, ${panY}px)`;
    }
  }

  function handleDragOver(event) {
    event.preventDefault();
    event.dataTransfer.dropEffect = 'copy';
  }

  function handleDrop(event) {
    event.preventDefault();
    const type = event.dataTransfer.getData('text/plain');
    const rect = canvas.getBoundingClientRect();
    const x = event.clientX - rect.left - panX;
    const y = event.clientY - rect.top - panY;
    dispatch('createNode', { type, x, y });
  }
</script>

<div 
  class="canvas-container" 
  class:dark-mode={isDarkMode}
  on:dragover={handleDragOver}
  on:drop={handleDrop}
>
  <canvas
    bind:this={canvas}
    on:mousedown={handleMouseDown}
  ></canvas>
  <SearchBar {searchableNodes} on:select={handleNodeSelect} />
  <div class="nodes" style="transform: translate({panX}px, {panY}px);">
    <slot></slot>
  </div>
</div>

<style>
  .canvas-container {
    position: relative;
    width: 100%;
    height: 100vh;
    overflow: hidden;
    transition: background-color 0.3s ease;
  }

  canvas {
    position: absolute;
    top: 0;
    left: 0;
  }

  .nodes {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
    will-change: transform;
  }

  .dark-mode {
    background-color: #1a1a1a;
    color: #fff;
  }
</style>