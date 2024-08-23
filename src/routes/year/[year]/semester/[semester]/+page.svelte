<!-- src/routes/year/[year]/semester/[semester]/+page.svelte -->
<script>
    import { onMount } from 'svelte';
    import { page } from '$app/stores';

    let classes = [];
    let error = '';

    $: year = $page.params.year;
    $: semester = $page.params.semester;

    onMount(async () => {
        try {
            const response = await fetch(`http://localhost:4000/skwela/year/${year}/semester/${semester}`);
            if (response.ok) {
                const data = await response.json();
                // Access the 'ClassListBy_Year_and_Semester' array
                classes = data.ClassListBy_Year_and_Semester;
            } else {
                error = `Failed to fetch classes: ${response.statusText}`;
            }
        } catch (err) {
            error = `Error fetching classes: ${err.message}`;
        }
    });
</script>

<h1>Classes for {semester} Semester in Year {year}</h1>

{#if error}
    <p>{error}</p>
{:else}
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Class ID</th>
                <th>Subject Code</th>
                <th>Semester</th>
                <th>Year</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            {#each classes as klase}
                <tr>
                    <td>{klase.classid}</td>
                    <td>{klase.subjectcode}</td>
                    <td>{klase.semester}</td>
                    <td>{klase.year}</td>
                    <td><a href={`/grades/${klase.classid}`}>View Grades</a></td>
                </tr>
            {/each}
        </tbody>
    </table>
{/if}
