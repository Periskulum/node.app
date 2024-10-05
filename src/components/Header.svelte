<script>
  const title = "My Svelte Web App"
</script>

<header>
  <h1>{title}</h1>
  <nav>
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  </nav>
</header>

<style>
  header {
    background-color: #f4f4f4;
    padding: 1rem;
    margin-bottom: 1rem;
  }

  h1 {
    margin: 0;
    font-size: 1.5rem;
  }

  nav ul {
    list-style-type: none;
    padding: 0;
    display: flex;
    gap: 1rem;
  }

  nav a {
    text-decoration: none;
    color: #333;
  }
</style>