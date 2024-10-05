<script>
  import { createEventDispatcher, onMount } from 'svelte';
  import { darkMode } from '../stores/darkMode.js';
  import Node from './Node.svelte';
  import MakeNode from './MakeNode.svelte';
  import DarkNode from './DarkNode.svelte';
  import TextNode from './TextNode.svelte';

  const dispatch = createEventDispatcher();

  const GAP = 10; // Gap between nodes
  const TOP_MARGIN = 20; // Top margin for the first row of nodes
  const LEFT_MARGIN = 12; // Left margin for all nodes

  function getInitialNodes() {
    const makeNodeWidth = 180;
    const darkNodeWidth = 120;
    const basicNodeWidth = 80;
    const totalWidth = makeNodeWidth + darkNodeWidth + basicNodeWidth + GAP * 2;
    const darkNodeX = LEFT_MARGIN + makeNodeWidth + GAP + 20; // Moved 20 pixels to the right

    return [
      { id: 'factory-2', component: MakeNode, props: { x: LEFT_MARGIN, y: TOP_MARGIN, label: 'make.node', width: makeNodeWidth, height: 80 } },
      { id: 'factory-3', component: DarkNode, props: { x: darkNodeX, y: TOP_MARGIN, label: 'dark.node', width: darkNodeWidth, height: 80 } },
      { id: 'factory-1', component: Node, props: { x: LEFT_MARGIN + totalWidth - basicNodeWidth, y: TOP_MARGIN, label: 'basic.node', width: basicNodeWidth, height: 40 } },
      { id: 'factory-4', component: TextNode, props: { x: LEFT_MARGIN, y: TOP_MARGIN + 80 + GAP, content: 'hello.world', title: 'text.node', width: 300, height: 150 } }
    ];
  }

  let factoryNodes = getInitialNodes();
  let factoryBounds;
  let isDragging = false;
  let draggedNodeId = null;
  let startX, startY;

  function handleMouseDown(event, id) {
    if (event.button === 1) { // Middle mouse button
      isDragging = true;
      draggedNodeId = id;
      const node = factoryNodes.find(n => n.id === id);
      startX = event.clientX - node.props.x;
      startY = event.clientY - node.props.y;
      event.preventDefault(); // Prevent default middle-click behavior
    }
  }

  function handleMouseMove(event) {
    if (isDragging && draggedNodeId) {
      const newX = event.clientX - startX;
      const newY = event.clientY - startY;
      factoryNodes = factoryNodes.map(node => 
        node.id === draggedNodeId ? { ...node, props: { ...node.props, x: newX, y: newY } } : node
      );
    }
  }

  function handleMouseUp() {
    isDragging = false;
    draggedNodeId = null;
  }

  function resetNodePositions() {
    factoryNodes = getInitialNodes();
  }
</script>

<div class="node-factory-content" 
     bind:this={factoryBounds} 
     on:mousemove={handleMouseMove}
     on:mouseup={handleMouseUp}
     on:mouseleave={handleMouseUp}>
  <div class="node-factory-header">
    <h2>node.factory</h2>
    <button on:click={resetNodePositions} class="reset-button">
      <span class="material-icons">restart_alt</span>
    </button>
  </div>
  <div class="factory-canvas">
    {#each factoryNodes as node (node.id)}
      <div class="factory-node"
           style="left: {node.props.x}px; top: {node.props.y}px;"
           on:mousedown={(e) => handleMouseDown(e, node.id)}>
        <svelte:component 
          this={node.component}
          {...node.props}
          id={node.id}
          isFactoryNode={true}
          isNonFunctional={true}
        />
      </div>
    {/each}
  </div>
</div>

<style>
  .node-factory-content {
    height: 100%;
    display: flex;
    flex-direction: column;
  }

  .node-factory-header {
    background-color: #3498db;
    color: white;
    margin: 0;
    padding: 10px;
    font-size: 18px;
    font-weight: normal;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: background-color 0.3s ease;
  }

  .reset-button {
    background: none;
    border: none;
    color: white;
    cursor: pointer;
    padding: 5px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .factory-canvas {
    flex-grow: 1;
    position: relative;
    overflow: hidden;
    background-color: #f0f0f0;
    transition: background-color 0.3s ease;
  }

  .factory-node {
    position: absolute;
    pointer-events: auto;
  }

  :global(.dark-mode) .node-factory-header {
    background-color: #2980b9;
  }

  :global(.dark-mode) .factory-canvas {
    background-color: #1a1a1a;
  }

  :global(.factory-node *) {
    pointer-events: none;
  }
</style>