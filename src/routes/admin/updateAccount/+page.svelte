<script>
    import { writable } from 'svelte/store';
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { fade } from 'svelte/transition';
    import { jwtDecode } from 'jwt-decode';

    let accounts = [];
    let error = '';
    let showUnauthorizedMessage = false;
    let countdown = 5;
    let redirectMessage = '';
    let userRole = '';
    const selectedAccount = writable(null);
    let successMessage = ''; // New state for success message

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

    async function updateAccount(event) {
        event.preventDefault();
        const formData = new FormData(event.target);
        const accountId = ($selectedAccount && $selectedAccount.id) || null;

        if (!accountId) {
            console.error("Account ID is not available.");
            return;
        }

        try {
            const response = await fetch(`http://localhost:4000/admin/${accountId}`, {
                method: 'PUT',
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(Object.fromEntries(formData))
            });

            if (response.ok) {
                accounts = await fetch('http://localhost:4000/admin', {
                    headers: {
                        'Authorization': `Bearer ${localStorage.getItem('jwtToken')}`
                    }
                }).then(res => res.json());

                successMessage = 'Update successful!'; // Set the success message
                const modal = document.getElementById('editAccountModal');
                const bsModal = bootstrap.Modal.getInstance(modal);
                setTimeout(() => bsModal.hide(), 1000); // Hide modal after a short delay
            } else {
                error = `Failed to update account: ${response.statusText}`;
            }
        } catch (error) {
            error = `Error updating account: ${error.message}`;
        }
    }
</script>


<h1>Accounts</h1>
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
        <th>Edit</th>
        <th>Delete</th>
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
          <td>{accountinfo.isActive ? 'Active' : 'Inactive'}</td>
          <td>
            <button 
              class="btn btn-primary" 
              data-bs-toggle="modal" 
              data-bs-target="#editAccountModal"
              on:click={() => selectedAccount.set(accountinfo)}
            >
              Edit
            </button>
          </td>
          <td>
            <!-- Add your delete button logic here -->
            <button class="btn btn-danger">Delete</button>
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

<!-- Bootstrap Modal -->
<div class="modal fade" id="editAccountModal" tabindex="-1" aria-labelledby="editAccountModalLabel" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="editAccountModalLabel">Edit Account</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      {#if $selectedAccount}
      <form id="editAccountForm" on:submit={updateAccount}>
        <div class="modal-body">
          <input type="hidden" name="id" value="{$selectedAccount.id}">
          <div class="mb-3">
            <label for="firstName" class="form-label">First Name</label>
            <input type="text" class="form-control" id="firstName" name="firstName" value="{$selectedAccount.firstName}" required>
          </div>
          <div class="mb-3">
            <label for="lastName" class="form-label">Last Name</label>
            <input type="text" class="form-control" id="lastName" name="lastName" value="{$selectedAccount.lastName}" required>
          </div>
          <div class="mb-3">
            <label for="username" class="form-label">Username</label>
            <input type="text" class="form-control" id="username" name="username" value="{$selectedAccount.username}" required>
          </div>
          <div class="mb-3">
            <label for="role" class="form-label">Role</label>
            <select class="form-select" id="role" name="role">
              <option value="Admin" selected={$selectedAccount.role === 'Admin'}>Admin</option>
              <option value="Student" selected={$selectedAccount.role === 'Student'}>Student</option>
              <option value="Registrar" selected={$selectedAccount.role === 'Registrar'}>Registrar</option>
              <option value="Teacher" selected={$selectedAccount.role === 'Teacher'}>Teacher</option>
            </select>
          </div>
        </div>
        <div class="modal-footer">
            <button type="submit" class="btn btn-primary" data-bs-dismiss="modal" >Save changes</button>
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
     
        </div>
      </form>
      {/if}
    </div>
  </div>
</div>

<!-- Success message after update -->
{#if successMessage}
<div class="alert alert-success" role="alert">
    {successMessage}
</div>
{/if}

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
