<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { fade } from 'svelte/transition';

    let showUnauthorizedMessage = false;
    let countdown = 5;

    onMount(() => {
        const token = localStorage.getItem('jwtToken');
        if (!token) {
            showUnauthorizedMessage = true;
            const interval = setInterval(() => {
                countdown--;
                if (countdown <= 0) {
                    clearInterval(interval);
                    goto('/login');
                }
            }, 1000);
        }

        // Validate token and check if user is admin
        // Implement a function to check role based on the token
    });

    function logout() {
        localStorage.removeItem('jwtToken');  // Clear the JWT token
        goto('/login');  // Redirect to the login page
    }
</script>

<h1>Student Dashboard</h1>
<p>Welcome, Student!</p>

<!-- Add a logout button -->
<button on:click={logout}>Logout</button>

{#if showUnauthorizedMessage}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>Access Unauthorized. Try again :p</p>
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
