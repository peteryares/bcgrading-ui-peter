<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { fade } from 'svelte/transition';
    import {jwtDecode} from 'jwt-decode';


   
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
            console.error('Error:', error); // Log the entire error object

            console.log(userRole)
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

    function clearFilters() {
    selectedYear = '';
    selectedSemester = '';
    selectedSubject = '';
    filterClasses();
}



</script>

<div class="container text-center">
    <div class="btn-group me-2">
       
    </div>

    <div class="btn-group me-2">
    
    </div>

    <div class="btn-group">
     
    </div>
    <button type="button" class="btn text-white btn-dark" on:click={clearFilters}>Clear Filters <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16">
        <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/>
      </svg></button>
</div>






<h2>Subject List</h2>



  
            <!-- Show filtered table if there are results -->
            <div style="max-height: 73vh; overflow-y: auto;">
                <table class="table table-bordered" style="width: 100% !important;">
                <thead>
                    <tr>
                       
                        <th>Subject Code</th>
                        <th>Subject Title</th>

                    </tr>
                </thead>
                <tbody>
                    {#each subjects as subject}
                    <tr>
                        <td>{subject.subjectcode}</td>
                        <td>{subject.title}</td>                
                    </tr>
                    {/each}
                </tbody>
            </table>
            </div>
      

{#if showUnauthorizedMessage}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>{redirectMessage}</p>
        <p>Redirecting you in {countdown} seconds...</p>
    </div>
</div>
{/if}


{#if error}
    <p>{error}</p>
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
