<script>
    import { onMount } from 'svelte';
    import { page } from '$app/stores';

    let grades = [];
    let classInfo = {};
    let error = '';
    let activeTab = 'all'; // Tab for sorting by terms

    $: classid = $page.params.classid;

    onMount(async () => {
        try {
            const response = await fetch(`http://localhost:4000/skwela/grades/${classid}`);
            if (response.ok) {
                const data = await response.json();
                classInfo = {
                    year: data.year,
                    semester: data.semester,
                    subject: data.subject,
                    title: data.title,
                    classid: data.classid
                };
                grades = data.GradeListBy_Classid; // Use the correct key to access grades
            } else {
                error = `Failed to fetch grades: ${response.statusText}`;
            }
        } catch (err) {
            error = `Error fetching grades: ${err.message}`;
        }
    });

    function filterGrades(term) {
        activeTab = term;
    }

    // Function to sort grades based on the active tab
    $: sortedGrades = (() => {
        if (activeTab === 'all') {
            return grades;
        }
        return grades.filter(grade => grade.term === activeTab);
    })();
</script>

<h1>Grades for Class ID {classid}</h1>
<h2>{classInfo.subject} - {classInfo.title}</h2>
<p><strong>Year:</strong> {classInfo.year}</p>
<p><strong>Semester:</strong> {classInfo.semester}</p>

{#if error}
    <p>{error}</p>
{:else}
    <div class="nav nav-tabs" role="tablist">
        <button class="nav-link {activeTab === 'all' ? 'active' : ''}" on:click={() => filterGrades('all')} role="tab">All Terms</button>
        <button class="nav-link {activeTab === 'Prelim' ? 'active' : ''}" on:click={() => filterGrades('Prelim')} role="tab">Prelim</button>
        <button class="nav-link {activeTab === 'Midterm' ? 'active' : ''}" on:click={() => filterGrades('Midterm')} role="tab">Midterm</button>
        <button class="nav-link {activeTab === 'Final' ? 'active' : ''}" on:click={() => filterGrades('Final')} role="tab">Final</button>
    </div>

    <table class="table table-striped mt-3">
        <thead>
            <tr>
                <th>Grade ID</th>
                <th>Student Name</th>
                <th>Term</th>
                <th>Student ID</th>
            </tr>
        </thead>
        <tbody>
            {#each sortedGrades as grade}
                <tr>
                    <td>{grade.gradeid}</td>
                    <td>{grade.studentname}</td>
                    <td>{grade.term}</td>
                    <td>{grade.studentid}</td>
                </tr>
            {/each}
        </tbody>
    </table>
{/if}
