<script>
    import { onMount } from 'svelte';
    import { page } from '$app/stores';

    let semesters = [];
    let error = '';

    // Access the year from the params using the $page store
    $: year = $page.params.year;

    onMount(async () => {
      try {
        const response = await fetch('http://localhost:4000/semesters');
        if (response.ok) {
          semesters = await response.json();
        } else {
          error = `Failed to fetch semesters: ${response.statusText}`;
        }
      } catch (err) {
        error = `Error fetching semesters: ${err.message}`;
      }
    });
</script>

<h1>Semesters for {year}</h1>
{#if error}
  <p>{error}</p>
{:else}
  <ul>
    {#each semesters as semester}
      <li>
        <!-- Generate a link using the year and semester parameters -->
        <a href={`/year/${year}/semester/${semester.semester}`}>{semester.semester}</a>
      </li>
    {/each}
  </ul>
{/if}
