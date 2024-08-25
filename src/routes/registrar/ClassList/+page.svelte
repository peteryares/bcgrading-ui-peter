<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { fade } from 'svelte/transition';
    import {jwtDecode} from 'jwt-decode';

    let years = [];
    let semesters = [];
    let subjects = [];
    let error = '';
    let showUnauthorizedMessage = false;
    let countdown = 5;
    let redirectMessage = '';
    let userRole = '';
    let showLogoutConfirm = false;

    onMount(async () => {

       // Import Bootstrap JS
       await import('bootstrap/dist/js/bootstrap.bundle.min.js');

        const token = localStorage.getItem('jwtToken');

        
        if (!token) {
            // No token found, redirect to login page
            unauthorizedAccess("No token found, redirecting to login.");
            return;
        }

        try {
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;  // Save userRole for later use

            // Check if the user has the correct role to access the page
            if (userRole !== 'Registrar') {
                // Unauthorized access, redirect to the page for the current user role
                redirectMessage = `Role '${userRole}' does not have access to this page.`;
                unauthorizedAccess("Redirecting you to your role-specific page.");
                return;
            }

            // Fetch the years if the user is authorized
            const yearlist = await fetch('http://localhost:4000/registrar/years', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (yearlist.ok) {
                years = await yearlist.json();
            } else {
                error = `Failed to fetch years: ${yearlist.statusText}`;
            }


            const semesterlist = await fetch('http://localhost:4000/registrar/semesters', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (semesterlist.ok) {
                semesters = await semesterlist.json();
            } else {
                error = `Failed to fetch semesters: ${semesterlist.statusText}`;
            }
            

            const subjectlist = await fetch('http://localhost:4000/registrar/subjects', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (subjectlist.ok) {
                subjects = await subjectlist.json();
            } else {
                error = `Failed to fetch subjects: ${subjectlist.statusText}`;
            }

            

        
        } catch (error) {
            unauthorizedAccess("Error decoding token, redirecting to login.");
        }

           
    });

    function unauthorizedAccess(message) {
        console.error(message); // Debugging line
        showUnauthorizedMessage = true;
        redirectMessage = message;
        const interval = setInterval(() => {
            countdown--;
            if (countdown <= 0) {
                clearInterval(interval);
                if (redirectMessage.includes("login")) {
                    goto('/login');  // Redirect to login page
                } else {
                    goto(`/${userRole}`);  // Redirect to User Role
                }
            }
        }, 1000);
    }

    // function logout() {
    //     localStorage.removeItem('jwtToken');  // Clear the JWT token
    //     goto('/login');  // Redirect to the login page immediately
    // }

    // // function showConfirmLogout() {
    // //     showLogoutConfirm = true;
    // // }

    // // function confirmLogout() {
    // //     showLogoutConfirm = false;
    // //     logout();
    // // }

    // // function cancelLogout() {
    // //     showLogoutConfirm = false;
    // // }
</script>


<!-- {#if !error}
    <ul>
        {#each years as year}
            <li>{year.year}</li>
        {/each}
    </ul>
{/if} -->
<div class="container text-center">
<div class="btn-group me-2">
    <button type="button" style="width: 160px !important;" class="btn dropdown-toggle border-dark" data-bs-toggle="dropdown" aria-expanded="false">
      YEAR               
    </button>
    {#if !error}
    <ul class="dropdown-menu" style="max-height: 200px; overflow-y: auto;">
        {#each years as year}
      <li class="bg-light border"><a class="dropdown-item " href="/">{year.year}</a></li>
      {/each}

      
    </ul>
    {/if}
  </div>

{#if !error}
<div class="btn-group  me-2">
    <button type="button" style="width: 160px !important;" class="btn dropdown-toggle border-dark" data-bs-toggle="dropdown">
      SEMESTER               
    </button>
    <ul class="dropdown-menu" style="max-height: 200px; overflow-y: auto;">
        {#each semesters as semester}
        <li class="bg-light border">
            <a class="dropdown-item" href="/">{semester.semester}</a>
        </li>
        {/each}
    </ul>
</div>
{/if}


{#if !error}
<div class="btn-group ">
    <button type="button" style="width: 500px !important;" class="btn dropdown-toggle border-dark" data-bs-toggle="dropdown">
      SUBJECTS            
    </button>
    <ul class="dropdown-menu" style="max-height: 200px; overflow-y: auto;">
        {#each subjects as subject}
        <li class="bg-light border">
            <a class="dropdown-item" href="/">{subject.subjectcode} - {subject.title}</a>
        </li>
        {/each}
    </ul>
</div>
{/if}
</div>
<h2>Class List</h2>
<table class="table table-bordered border border-dark">
    <thead>
      <tr>
        <th scope="col">#</th>
        <th scope="col">First</th>
        <th scope="col">Last</th>
        <th scope="col">Handle</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th scope="row">1</th>
        <td>Mark</td>
        <td>Otto</td>
        <td>@mdo</td>
      </tr>
      <tr>
        <th scope="row">2</th>
        <td>Jacob</td>
        <td>Thornton</td>
        <td>@fat</td>
      </tr>
      <tr>
        <th scope="row">3</th>
        <td colspan="2">Larry the Bird</td>
        <td>@twitter</td>
      </tr>
    </tbody>
  </table>

<!-- Display error message if any -->
{#if error}
    <p>{error}</p>
{/if}

<!-- <button on:click={showConfirmLogout}>Logout</button> -->

<!-- {#if showLogoutConfirm}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>Are you sure you want to log out?</p>
        <button on:click={confirmLogout}>Yes</button>
        <button on:click={cancelLogout}>No</button>
    </div>
</div>
{/if} -->

{#if showUnauthorizedMessage}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>{redirectMessage}</p>
        <p>Redirecting you in {countdown} seconds...</p>
    </div>
</div>
{/if}

<style>
    .popup {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.9);
        color: white;
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 1000;
    }

    .popup-content {
        padding: 20px;
        background-color: rgba(255, 255, 255, 0.1);
        border-radius: 8px;
        text-align: center;
    }
   
</style>
