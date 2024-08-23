<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    let username = '';
    let password = '';
    let error = '';

    async function handleLogin() {
        try {
            const response = await fetch('http://localhost:4000/accounts/authenticate', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ username, password })
            });

            if (!response.ok) {
                throw new Error('Login failed');
            }

            const data = await response.json();
            const role = data.role;

            // Save JWT token to local storage or cookies as needed
            localStorage.setItem('jwtToken', data.jwtToken);

            // Redirect based on the role
            if (role === 'Admin') {
                goto('/admin');
            } else if (role === 'Registrar') {
                goto('/registrar');
            } else if (role === 'Teacher') {
                goto('/teacher');
            } else if (role === 'Student') {
                goto('/student');
            } else {
                throw new Error('Unknown role');
            }
        } catch (err) {
            error = err.message;
        }
    }
</script>

<h1>Login</h1>
{#if error}
    <p style="color: red">{error}</p>
{/if}

<form on:submit|preventDefault={handleLogin}>
    <label for="username">Username:</label>
    <input id="username" bind:value={username} required />

    <label for="password">Password:</label>
    <input id="password" type="password" bind:value={password} required />

    <button type="submit">Login</button>
</form>
