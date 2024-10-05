<script>
  import { onMount } from 'svelte';
  import Canvas from './components/Canvas.svelte';
  import Node from './components/Node.svelte';
  import MakeNode from './components/MakeNode.svelte';
  import DarkNode from './components/DarkNode.svelte';
  import TextNode from './components/TextNode.svelte';
  import { darkMode } from './stores/darkMode.js';
  import NodeFactory from './components/NodeFactory.svelte';

  let isNodeFactoryOpen = false;
  let nodeFactoryWidth = 0;
  let animationDuration = 300; // ms

  function toggleNodeFactory() {
    isNodeFactoryOpen = !isNodeFactoryOpen;
    nodeFactoryWidth = isNodeFactoryOpen ? 25 : 0;
  }

  let nodes = [
    { id: 1, component: Node, props: { x: 100, y: 100, label: 'node.1' } },
    { id: 2, component: Node, props: { x: 300, y: 100, label: 'node.2' } },
    { id: 3, component: Node, props: { x: 500, y: 100, label: 'node.3' } },
    { id: 4, component: MakeNode, props: { x: 100, y: 300, label: 'make.node' } },
    { id: 5, component: DarkNode, props: { x: 300, y: 300, label: 'dark.node' } },
    { id: 6, component: TextNode, props: { x: 500, y: 300, content: 'hello.world', title: 'text.node' } }
  ];

  function handleNodeMove(event) {
    const { id, x, y } = event.detail;
    nodes = nodes.map(node => node.id === id ? { ...node, props: { ...node.props, x, y } } : node);
  }

  function handleCreateNode(event) {
    const { type, x, y } = event.detail;
    const newNode = createNewNode(type, x, y);
    if (newNode) {
      nodes = [...nodes, newNode];
    }
  }

  function createNewNode(type, x, y) {
    const id = nodes.length + 1;
    switch (type) {
      case 'basic':
        return { id, component: Node, props: { x, y, label: `node.${id}` } };
      case 'make':
        return { id, component: MakeNode, props: { x, y, label: 'make.node' } };
      case 'dark':
        return { id, component: DarkNode, props: { x, y, label: 'dark.node' } };
      case 'text':
        return { id, component: TextNode, props: { x, y, content: 'hello.world', title: 'text.node' } };
      default:
        return null;
    }
  }

  function handleContentUpdate(event) {
    const { id, content, title } = event.detail;
    nodes = nodes.map(node => node.id === id ? { ...node, props: { ...node.props, content, title } } : node);
  }

  $: searchableNodes = nodes.map(node => ({
    id: node.id,
    label: node.props.label || node.props.title || `node.${node.id}`,
    x: node.props.x,
    y: node.props.y
  }));

  onMount(() => {
    // Any initialization code can go here
  });
</script>

<div class="app-container" class:dark-mode={$darkMode}>
  <div 
    class="node-factory-tab"
    on:click={toggleNodeFactory}
    style="transform: translateX({nodeFactoryWidth}vw); transition: transform {animationDuration}ms ease;"
  >
    node.factory
  </div>
  <div class="content-container">
    <div 
      class="node-factory"
      style="width: {nodeFactoryWidth}vw; transition: width {animationDuration}ms ease;"
    >
      <NodeFactory on:createNode={handleCreateNode} />
    </div>
    <div class="separator" style="left: {nodeFactoryWidth}vw; transition: left {animationDuration}ms ease;"></div>
    <div 
      class="canvas-container"
      style="width: calc(100vw - {nodeFactoryWidth}vw); transition: width {animationDuration}ms ease;"
    >
      <Canvas {searchableNodes} on:createNode={handleCreateNode}>
        {#each nodes as node (node.id)}
          <svelte:component 
            this={node.component}
            {...node.props}
            id={node.id}
            on:move={handleNodeMove}
            on:createNode={handleCreateNode}
            on:contentUpdate={handleContentUpdate}
          />
        {/each}
      </Canvas>
    </div>
  </div>
</div>

<style>
  .app-container {
    position: relative;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }

  .node-factory-tab {
    position: fixed;
    top: 50%;
    left: 0;
    background-color: #3498db;
    color: white;
    padding: 6px 10px;
    cursor: pointer;
    border-radius: 0 5px 5px 0;
    z-index: 1000;
    transform-origin: left center;
    writing-mode: vertical-rl;
    text-orientation: mixed;
    font-size: 14px;
    transition: background-color 0.3s ease;
  }

  .content-container {
    display: flex;
    width: 100vw;
    height: 100vh;
  }

  .node-factory {
    background-color: #f0f0f0;
    overflow: hidden;
    border-right: 2px solid #3498db;
    transition: background-color 0.3s ease;
  }

  .separator {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 2px;
    background-color: #3498db;
    transition: background-color 0.3s ease;
  }

  .canvas-container {
    flex-grow: 1;
  }

  /* Dark mode styles */
  .dark-mode .node-factory-tab {
    background-color: #2980b9;
  }

  .dark-mode .node-factory {
    background-color: #2c3e50;
    border-right-color: #2980b9;
  }

  .dark-mode .separator {
    background-color: #2980b9;
  }
</style>