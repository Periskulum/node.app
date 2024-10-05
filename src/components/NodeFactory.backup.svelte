<script>
  import { createEventDispatcher, onMount } from 'svelte';
  import { darkMode } from '../stores/darkMode.js';
  import Node from './Node.svelte';
  import MakeNode from './MakeNode.svelte';
  import DarkNode from './DarkNode.svelte';
  import TextNode from './TextNode.svelte';

  const dispatch = createEventDispatcher();

  let factoryNodes = [
    { id: 'factory-1', component: Node, props: { x: 20, y: 20, label: 'node.1', width: 80, height: 40 } },
    { id: 'factory-2', component: MakeNode, props: { x: 20, y: 100, label: 'make.node', width: 180, height: 80 } },
    { id: 'factory-3', component: DarkNode, props: { x: 20, y: 180, label: 'dark.node', width: 120, height: 80 } },
    { id: 'factory-4', component: TextNode, props: { x: 20, y: 260, content: 'hello.world', title: 'text.node', width: 300, height: 150 } }
  ];

  let factoryBounds;
  let factoryWidth = 0;
  let factoryHeight = 0;
  const PADDING = 10;

  onMount(() => {
    updateFactoryDimensions();
    window.addEventListener('resize', updateFactoryDimensions);
    return () => {
      window.removeEventListener('resize', updateFactoryDimensions);
    };
  });

  function updateFactoryDimensions() {
    if (factoryBounds) {
      factoryWidth = factoryBounds.clientWidth;
      factoryHeight = factoryBounds.clientHeight;
      repositionNodes();
    }
  }

  function repositionNodes() {
    factoryNodes = factoryNodes.map(node => ({
      ...node,
      props: {
        ...node.props,
        x: Math.max(PADDING, Math.min(node.props.x, factoryWidth - node.props.width - PADDING)),
        y: Math.max(PADDING, Math.min(node.props.y, factoryHeight - node.props.height - PADDING))
      }
    }));
  }

  function handleNodeMove(event) {
    const { id, x, y } = event.detail;
    factoryNodes = factoryNodes.map(node => {
      if (node.id === id) {
        const newX = Math.max(PADDING, Math.min(x, factoryWidth - node.props.width - PADDING));
        const newY = Math.max(PADDING, Math.min(y, factoryHeight - node.props.height - PADDING));
        return { ...node, props: { ...node.props, x: newX, y: newY } };
      }
      return node;
    });
  }

  function handleNodeSelect(event) {
    // Implement selection logic if needed
  }
</script>

<div class="node-factory-content" bind:this={factoryBounds}>
  <h2 class="node-factory-header">node.factory</h2>
  <div class="factory-canvas">
    {#each factoryNodes as node (node.id)}
      <div class="factory-node">
        <svelte:component 
          this={node.component}
          {...node.props}
          id={node.id}
          on:move={handleNodeMove}
          on:select={handleNodeSelect}
          isStatic={false}
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
    text-align: center;
    transition: background-color 0.3s ease;
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
  }

  :global(.dark-mode) .node-factory-header {
    background-color: #2980b9;
  }

  :global(.dark-mode) .factory-canvas {
    background-color: #1a1a1a;
  }
</style>