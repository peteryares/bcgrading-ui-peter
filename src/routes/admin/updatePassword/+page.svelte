<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { fade } from 'svelte/transition';
    import { jwtDecode } from 'jwt-decode';
    import { writable } from 'svelte/store'; // Import writable from svelte/store

    let accounts = [];
    let error = '';
    let showUnauthorizedMessage = false;
    let countdown = 5;
    let redirectMessage = '';
    let userRole = '';
    const selectedAccount = writable(null); // Initialize writable store
    let passwordError = ''; // New state for password error
    let passwordSuccessMessage = ''; // New state for password success message
   
    let bootstrap;

    onMount(async () => {
        const bootstrapModule = await import('bootstrap/dist/js/bootstrap.bundle.min.js');
        bootstrap = bootstrapModule.default;

        const token = localStorage.getItem('jwtToken');
        if (!token) {
            unauthorizedAccess("No token found, redirecting to login.");
            return;
        }

        try {
            const decodedToken = jwtDecode(token);
            userRole = decodedToken.role;

            if (userRole !== 'Admin') {
                redirectMessage = `Role '${userRole}' does not have access to this page.`;
                unauthorizedAccess("Redirecting you to your role-specific page.");
                return;
            }

            const userlist = await fetch('http://localhost:4000/admin', {
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            if (userlist.ok) {
                accounts = await userlist.json();
            } else {
                error = `Failed to fetch accounts: ${userlist.statusText}`;
            }
        } catch (error) {
            unauthorizedAccess("Error decoding token, redirecting to login.");
        }
    });

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

    async function changePassword(event) {
        event.preventDefault();
        const formData = new FormData(event.target);
        const accountId = ($selectedAccount && $selectedAccount.id) || null;

        if (!accountId) {
            console.error("Account ID is not available.");
            return;
        }

        try {
            const response = await fetch(`http://localhost:4000/admin/updatepassword/${accountId}`, {
                method: 'PUT',
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(Object.fromEntries(formData))
            });

            if (response.ok) {
                passwordSuccessMessage = 'Password change successful!'; // Set the success message
                const modal = document.getElementById('changePasswordModal');
                const bsModal = bootstrap.Modal.getInstance(modal);
                setTimeout(() => bsModal.hide(), 500); // Hide modal after a short delay



                modal.addEventListener('hidden.bs.modal', () => {
            const form = modal.querySelector('form');
            if (form) {
                form.reset(); // Reset form fields
            }
            // Optionally, clear success and error messages
            passwordSuccessMessage = '';
            passwordError = '';
        });


            } else {
                passwordError = `Failed to change password: ${response.statusText}`;
            }
        } catch (error) {
            passwordError = `Error changing password: ${error.message}`;
        }
    }


   


</script>


<h1 class="text-center">Change Passwoords</h1>
{#if !error}
  <table class="table table-bordered">
    <thead>
      <tr>
        <th>ID</th>
        <th>First Name</th>
        <th>Last Name</th>
        <th>Username</th>
        <th>Role</th>
        <th>Created</th>
        <th>Updated</th>
        <th>Status</th>
        <th>Change Password</th>
      </tr>
    </thead>
    <tbody>
      {#each accounts as accountinfo}
        <tr>
          <td>{accountinfo.id}</td>
          <td>{accountinfo.firstName}</td>
          <td>{accountinfo.lastName}</td>
          <td>{accountinfo.username}</td>
          <td>{accountinfo.role}</td>
          <td>{accountinfo.created}</td>
          <td>{accountinfo.updated}</td>
          <td>
            {#if accountinfo.isActive}
              <p class="badge-lg text-center text-bg-success">Active</p>
            {:else}
              <p class="badge-lg text-center text-bg-danger">Inactive</p>
            {/if}
          </td>
          
         
          <td>
            <button 
              class="btn btn-success" 
              data-bs-toggle="modal" 
              data-bs-target="#changePasswordModal"
              on:click={() => selectedAccount.set(accountinfo)}
            >
              Change Password
            </button>
          </td>
          
        </tr>
      {/each}
    </tbody>
  </table>
{/if}

{#if showUnauthorizedMessage}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>{redirectMessage}</p>
        <p>Redirecting you in {countdown} seconds...</p>
    </div>
</div>
{/if}

<!-- Bootstrap Modal for Changing Password -->
<!-- Modal HTML -->
<div class="modal fade" id="changePasswordModal" tabindex="-1" aria-labelledby="changePasswordModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="changePasswordModalLabel">Change Password</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <form on:submit={changePassword}>
                <div class="modal-body">
                    <div class="mb-3">
                        <label for="password" class="form-label">New Password</label>
                        <input type="password" class="form-control" id="password" name="password" required>
                    </div>
                    <div class="mb-3">
                        <label for="confirmPassword" class="form-label">Confirm Password</label>
                        <input type="password" class="form-control" id="confirmPassword" name="confirmPassword" required>
                    </div>
                    {#if passwordError}
                        <div class="alert alert-danger">{passwordError}</div>
                    {/if}
                    {#if passwordSuccessMessage}
                        <div class="alert alert-success">{passwordSuccessMessage}</div>
                    {/if}
                </div>
                <div class="modal-footer">
                    <button type="submit" class="btn btn-primary">Save changes</button>
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
                 
                </div>
            </form>
        </div>
    </div>
</div>

<!-- Success message after update -->

<!-- Error message if any -->
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

    .alert {
        margin-top: 10px;
    }
</style>
