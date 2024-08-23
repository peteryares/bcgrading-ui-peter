<!-- src/routes/years.svelte -->
<script>
    import { onMount } from 'svelte';
    
  
    let years = [];
    let error = '';
  
    onMount(async () => {
      try {
        const response = await fetch('http://localhost:4000/years');
        if (response.ok) {
          years = await response.json();
        } else {
          error = `Failed to fetch years: ${response.statusText}`;
        }
      } catch (err) {
        error = `Error fetching years: ${err.message}`;
      }
    });
  </script>
  
  <h1>Years</h1>
  {#if error}
    <p>{error}</p>
  {:else}
    <ul>
      {#each years as year}
        <li>
          <a href={`/year/${year.year}/semester`}>{year.year}</a>
        </li>
      {/each}
    </ul>
  {/if}
  