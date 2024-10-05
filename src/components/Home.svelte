<script>
  let count = 0
  
  function incrementCount() {
    count += 1
  }
</script>

<section>
  <h2>Welcome to My Svelte Web App</h2>
  <p>This is a simple web application built with Svelte and JavaScript.</p>
  <button on:click={incrementCount}>
    Clicked {count} {count === 1 ? 'time' : 'times'}
  </button>
</section>

<style>
  section {
    background-color: #fff;
    padding: 1rem;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  h2 {
    margin-top: 0;
  }

  button {
    background-color: #4CAF50;
    border: none;
    color: white;
    padding: 10px 20px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 4px;
  }
</style>