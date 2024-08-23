<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    let firstName = '';
    let lastName = '';
    let username = '';
    let password = '';
    let confirmPassword = '';
    let role = 'Student'; // Default role
    let message = '';

    const roles = ['Admin', 'Registrar', 'Student', 'Teacher'];

    async function handleSubmit() {
        if (password !== confirmPassword) {
            message = "Passwords do not match.";
            return;
        }

        // Retrieve JWT token from localStorage
        const token = localStorage.getItem('jwtToken');

        try {
            const response = await fetch('http://localhost:4000/admin', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}` // Include JWT token
                },
                body: JSON.stringify({
                    firstName,
                    lastName,
                    username,
                    password,
                    confirmPassword,
                    role
                })
            });

            if (!response.ok) {
                const errorData = await response.json();
                throw new Error(errorData.message || 'Error creating account');
            }

            const result = await response.json();
            message = 'Account created successfully!';
            goto('/Admin/addAccount'); // Redirect to a success page or dashboard
        } catch (error) {
            message = error.message || 'Error creating account';
        }
    }
</script>

<h1>Add Account</h1>
<section class="h-25 h-custom" style="background-color: #8fc4b7;">
    <div class="container py-5 h-100">
      <div class="row d-flex justify-content-center align-items-center h-100">
        <div class="col-lg-8 col-xl-6">
          <div class="card rounded-3">
           
            <div class="card-body p-4 p-md-5">
              <h3 class="mb-4 pb-2 pb-md-0 mb-md-5 px-md-2">Registration Info</h3>
  
              <form class="px-md-2" on:submit|preventDefault={handleSubmit}>
  
                <div class="form-outline mb-4">
                  <input type="text" id="firstName" class="form-control" bind:value={firstName} />
                  <label class="form-label" for="firstName">First Name</label>
                </div>
  
                <div class="form-outline mb-4">
                  <input type="text" id="lastName" class="form-control" bind:value={lastName} />
                  <label class="form-label" for="lastName">Last Name</label>
                </div>
  
                <div class="form-outline mb-4">
                  <input type="text" id="username" class="form-control" bind:value={username} />
                  <label class="form-label" for="username">Username</label>
                </div>
  
                <div class="form-outline mb-4">
                  <input type="password" id="password" class="form-control" bind:value={password} />
                  <label class="form-label" for="password">Password</label>
                </div>
  
                <div class="form-outline mb-4">
                  <input type="password" id="confirmPassword" class="form-control" bind:value={confirmPassword} />
                  <label class="form-label" for="confirmPassword">Confirm Password</label>
                </div>
  
                <div class="mb-4">
                  <select class="form-select" bind:value={role}>
                    <option value="" disabled>Select Role</option>
                    {#each roles as r}
                      <option value={r}>{r}</option>
                    {/each}
                  </select>
                </div>
  
                <button type="submit" class="btn btn-success btn-lg mb-1">Submit</button>
  
              </form>
  
              {#if message}
                <div class="alert alert-info mt-4" role="alert">
                  {message}
                </div>
              {/if}
  
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>

{#if message}
    <p>{message}</p>
{/if}
