<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { fade } from 'svelte/transition';
    import { jwtDecode } from 'jwt-decode';

    let showUnauthorizedMessage = false;
    let countdown = 5;

    onMount(() => {
        const token = localStorage.getItem('jwtToken');

        // If there's no token, redirect to login
        if (!token) {
            unauthorizedAccess("No token found, redirecting to login.");
            return;
        }

        try {
            const decodedToken = jwtDecode(token);
            console.log("Decoded Token:", decodedToken); // Debugging line

            const userRole = decodedToken.role;
            console.log("User Role:", userRole); // Debugging line

            // Check if the user has the correct role to access the admin page
            if (userRole !== 'Registrar') {
                unauthorizedAccess(`Role '${userRole}' does not have access to this page.`);
            }
        } catch (error) {
            // Handle potential errors in decoding or missing role in token
            console.error("Error decoding token:", error);
            unauthorizedAccess("Error decoding token, redirecting to login.");
        }
    });

    function unauthorizedAccess(message) {
        console.error(message); // Debugging line
        showUnauthorizedMessage = true;
        const interval = setInterval(() => {
            countdown--;
            if (countdown <= 0) {
                clearInterval(interval);
                goto('/login');
            }
        }, 1000);
    }

    function logout() {
        localStorage.removeItem('jwtToken');  // Clear the JWT token
        goto('/login');  // Redirect to the login page
    }
</script>

<h1>Registrar Dashboard</h1>
<p>Welcome, Registrar!</p>

<!-- Add a logout button -->
<button on:click={logout}>Logout</button>

{#if showUnauthorizedMessage}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>Access Unauthorized. Try again </p>
        <p>Redirecting you to the login page in {countdown} seconds...</p>
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
