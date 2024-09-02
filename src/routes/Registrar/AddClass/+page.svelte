<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { jwtDecode } from 'jwt-decode';

    let years = [];
    let semesters = [];
    let subjects = [];
    let error = '';
    let successMessage = '';
    let showUnauthorizedMessage = false;
    let countdown = 5;
    let redirectMessage = '';
    let userRole = '';

    let selectedYear = '';
    let selectedSemester = '';
    let selectedSubject = '';
    let teacherId = ''; // Assume this will be provided by the user or another API call

    onMount(async () => {
        await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');

        if (!token) {
            unauthorizedAccess("No token found, redirecting to login.");
            return;
        }

        try {
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;

            if (userRole !== 'Registrar') {
                redirectMessage = `Role '${userRole}' does not have access to this page.`;
                unauthorizedAccess("Redirecting you to your role-specific page.");
                return;
            }

            const yearlist = await fetch('http://localhost:4000/registrar/years', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (yearlist.ok) {
                years = await yearlist.json();
            } else {
                error = `Failed to fetch years: ${yearlist.statusText}`;
            }

            const semesterlist = await fetch('http://localhost:4000/registrar/semesters', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (semesterlist.ok) {
                semesters = await semesterlist.json();
            } else {
                error = `Failed to fetch semesters: ${semesterlist.statusText}`;
            }

            const subjectlist = await fetch('http://localhost:4000/registrar/subjects', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (subjectlist.ok) {
                subjects = await subjectlist.json();
            } else {
                error = `Failed to fetch subjects: ${subjectlist.statusText}`;
            }

        } catch (error) {
            console.error('Error:', error);

            console.log(userRole);
            unauthorizedAccess("Error decoding token, redirecting to login.");
        }
    });

    async function addClass() {
        const token = localStorage.getItem('jwtToken');
        try {
            const response = await fetch('http://localhost:4000/registrar/addclass', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}`
                },
                body: JSON.stringify({
                    subjectcode: selectedSubject,
                    semester: selectedSemester,
                    year: selectedYear,
                    teacherid: teacherId
                })
            });

            if (response.ok) {
                successMessage = `Class added successfully!
                [${selectedSubject}] [Year: ${selectedYear}] [Semester: ${selectedSemester}] [Teacher ID: ${teacherId}]`;

                // Hide the success message after 5 seconds
                setTimeout(() => {
                    successMessage = '';
                }, 5000);

                // Optionally, you can reset the selection
                selectedYear = '';
                selectedSemester = '';
                selectedSubject = '';
                teacherId = '';
            } else {
                error = `Failed to add class: ${response.statusText}`;
                setTimeout(() => {
                    error = '';
                }, 4000);
            }
        } catch (err) {
            error = `Error adding class: ${err.message}`;
        }
    }

    function unauthorizedAccess(message) {
        console.error(message);
        showUnauthorizedMessage = true;
        redirectMessage = message;
        const interval = setInterval(() => {
            countdown--;
            if (countdown <= 0) {
                clearInterval(interval);
                if (redirectMessage.includes("login")) {
                    goto('/login');
                } else {
                    goto(`/${userRole}`);
                }
            }
        }, 1000);
    }
</script>

<div class="container mt-4" style="height: 70vh;">
    <h2>Add Class</h2>

    {#if successMessage}
        <div class="alert alert-success" role="alert">
            {successMessage}
        </div>
    {/if}

    {#if error}
        <div class="alert alert-danger" role="alert">
            {error}
        </div>
    {/if}

    <div class="mb-3">
        <label for="subjectSelect" class="form-label">Select Subject</label>
        <div class="dropdown">
            <select id="subjectSelect" class="form-select" bind:value={selectedSubject} required size="1" style="max-height: 150px; overflow-y: auto;">
                <option value="">Choose...</option>
                {#each subjects as subject}
                    <option value={subject.subjectcode}>{subject.subjectcode} - {subject.title}</option>
                {/each}
            </select>
        </div>
    </div>
    
    <div class="mb-3">
        <label for="yearSelect" class="form-label">Select Year</label>
        <select id="yearSelect" class="form-select" bind:value={selectedYear} required>
            <option value="">Choose...</option>
            {#each years as year}
                <option value={year.year}>{year.year}</option>
            {/each}
        </select>
    </div>
    
    <div class="mb-3">
        <label for="semesterSelect" class="form-label">Select Semester</label>
        <select id="semesterSelect" class="form-select" bind:value={selectedSemester} required>
            <option value="">Choose...</option>
            {#each semesters as semester}
                <option value={semester.semester}>{semester.semester}</option>
            {/each}
        </select>
    </div>
    
    <div class="mb-3">
        <label for="teacherId" class="form-label">Teacher ID</label>
        <input type="text" id="teacherId" class="form-control" bind:value={teacherId} required placeholder="Enter Teacher ID" />
    </div>
    

    <button class="btn btn-primary" on:click={addClass}>Add Class</button>
</div>

<style>
    .container {
        max-width: 600px;
        max-height: 100vh;
    }
</style>
