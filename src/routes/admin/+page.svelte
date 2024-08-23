<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    onMount(() => {
        const token = localStorage.getItem('jwtToken');
        if (!token) {
            goto('/login');
        }

        // Validate token and check if user is admin
        // Implement a function to check role based on the token
    });

    function logout() {
        localStorage.removeItem('jwtToken');  // Clear the JWT token
        goto('/login');  // Redirect to the login page
    }
</script>

<h1>Admin Dashboard</h1>
<p>Welcome, Admin!</p>

<!-- Add a logout button -->
<button on:click={logout}>Logout</button>
