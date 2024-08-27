<script>
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition';

    let accounts = [];
    let activeAccounts = [];
    let deletedAccounts = [];
    let error = null;
    let selectedTab = 'all'; // Default tab
    let showUnauthorizedMessage = false;
    let countdown = 5;
    let redirectMessage = '';

    // Fetch accounts on component mount
    onMount(async () => {
        try {
            const token = localStorage.getItem('jwtToken'); // Replace with your actual JWT token

            // Fetch active accounts
            const userlist = await fetch('http://localhost:4000/admin', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (userlist.ok) {
                activeAccounts = await userlist.json();
            } else {
                handleUnauthorized('Failed to fetch active accounts');
                return;
            }

            // Fetch inactive (deleted) accounts
            const inactiveUserlist = await fetch('http://localhost:4000/admin/deleted', {
                headers: {
                    'Authorization': `Bearer ${token}` // Include JWT token
                }
            });

            if (inactiveUserlist.ok) {
                deletedAccounts = await inactiveUserlist.json();
            } else {
                handleUnauthorized('Failed to fetch inactive accounts');
                return;
            }

            // Combine active and inactive accounts for the "All" tab
            accounts = [...activeAccounts, ...deletedAccounts];

        } catch (err) {
            handleUnauthorized(`Error fetching accounts: ${err.message}`);
        }
    });

    function handleUnauthorized(message) {
        redirectMessage = message;
        showUnauthorizedMessage = true;
        
        const interval = setInterval(() => {
            countdown--;
            if (countdown <= 0) {
                clearInterval(interval);
                
                // Ensure this is only executed in the browser
                if (typeof window !== 'undefined') {
                    window.location.href = '/login'; // Redirect to login page
                }
            }
        }, 1000);
    }
</script>

<!-- Unauthorized message popup -->
{#if showUnauthorizedMessage}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>{redirectMessage}</p>
        <p>Redirecting you in {countdown} seconds...</p>
    </div>
</div>
{/if}

<!-- Bootstrap tabs -->
<ul class="nav nav-tabs">
    <li class="nav-item">
        <button type="button" class="nav-link {selectedTab === 'all' ? 'active' : ''}" on:click={() => selectedTab = 'all'}>
            All Accounts
        </button>
    </li>
    <li class="nav-item">
        <button type="button" class="nav-link {selectedTab === 'active' ? 'active' : ''}" on:click={() => selectedTab = 'active'}>
            Active Accounts
        </button>
    </li>
    <li class="nav-item">
        <button type="button" class="nav-link {selectedTab === 'deleted' ? 'active' : ''}" on:click={() => selectedTab = 'deleted'}>
            Deleted Accounts
        </button>
    </li>
</ul>

<!-- Table to display accounts -->
{#if !error}
  <table class="table table-bordered">
    <thead>
      <tr>
        <th></th>
        <th>ID</th>
        <th>First Name</th>
        <th>Last Name</th>
        <th>Username</th>
        <th>Role</th>
        <th>Created</th>
        <th>Updated</th>
        <th>Status</th>
        <th></th>
        <th></th>
        <th></th>
        <th></th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      {#if selectedTab === 'all'}
        {#each accounts as accountinfo}
          <tr>
            <td></td>
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
            <!-- <td><button class="btn btn-primary">Edit</button></td>
            <td><button class="btn btn-danger">Delete</button></td> -->
          </tr>
        {/each}
      {/if}

      {#if selectedTab === 'active'}
        {#each activeAccounts as accountinfo}
          <tr>
            <td></td>
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
            <!-- <td><button class="btn btn-primary">Edit</button></td>
            <td><button class="btn btn-danger">Delete</button></td> -->
          </tr>
        {/each}
      {/if}

      {#if selectedTab === 'deleted'}
        {#each deletedAccounts as accountinfo}
          <tr>
            <td></td>
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
            <!-- <td><button class="btn btn-primary">Edit</button></td>
            <td><button class="btn btn-danger">Delete</button></td> -->
          </tr>
        {/each}
      {/if}
    </tbody>
  </table>
{:else}
  <div class="alert alert-danger">{error}</div>
{/if}

<style>
    .popup {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.8);
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
