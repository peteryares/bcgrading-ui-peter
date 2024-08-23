  <!-- src/routes/__layout.svelte -->
<script>
  import 'bootstrap/dist/css/bootstrap.min.css';
  import { fade } from 'svelte/transition';
  import { goto } from '$app/navigation';
  let showLogoutConfirm = false;

  function logout() {
    localStorage.removeItem('jwtToken');  // Clear the JWT token
    goto('/login');  // Redirect to the login page immediately
}

function showConfirmLogout() {
        showLogoutConfirm = true;
    }

    function confirmLogout() {
        showLogoutConfirm = false;
        logout();
    }

    function cancelLogout() {
        showLogoutConfirm = false;
    }

</script>



<div class="app-container">
  <nav class="navbar navbar-expand navbar-dark bg-dark">
    <div class="container">
      <div class="navbar-nav me-auto">
        <a class="nav-item nav-link text-white" href="/Admin">ADMIN ACCOUNT</a>
        <a class="nav-item nav-link text-white" href="/Admin/addAccount/">Add Account</a>
        <a class="nav-item nav-link text-white" href="/settings">Settings</a>
      </div> </div>
      <div class="navbar-nav ms-auto">
        <button class="nav-item nav-link text-white" on:click={showConfirmLogout}>LOGOUT</button>
      </div>
   
  </nav>
  <div class="card">
    <div class="card-header">

    </div>
    <div class="card-body">
      <slot></slot>
    </div>
  </div>
</div>


{#if showLogoutConfirm}
<div class="popup" in:fade>
    <div class="popup-content">
        <p>Are you sure you want to log out?</p>
        <button on:click={confirmLogout}>Yes</button>
        <button on:click={cancelLogout}>No</button>
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

    .popup-content button {
        margin: 5px;
        padding: 10px 20px;
        font-size: 16px;
        border: none;
        border-radius: 1px;
        cursor: pointer;
    }

    .popup-content button:first-of-type {
        background-color: #4CAF50; /* Green for Yes */
        color: white;
    }

    .popup-content button:last-of-type {
        background-color: #f44336; /* Red for No */
        color: white;
    }
</style>
