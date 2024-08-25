<script>
  import 'bootstrap/dist/css/bootstrap.min.css';
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { fade } from 'svelte/transition';
  import { goto } from '$app/navigation';

  let offcanvasElement;
  let bootstrap;  // Variable to hold the imported Bootstrap module
  let showLogoutConfirm = false;


  

  // Function to handle closing offcanvas
  function closeOffcanvas() {
    if (offcanvasElement && bootstrap) {
      const offcanvasInstance = bootstrap.Offcanvas.getInstance(offcanvasElement);
      if (offcanvasInstance) {
        offcanvasInstance.hide();
      }
    }
  }

  // Reactively close offcanvas on route change
  $: $page.url, closeOffcanvas();

  // Function to show confirmation popup for logout
  function showConfirmLogout() {
    showLogoutConfirm = true;
  }

  // Function to handle logout
  function logout() {
    localStorage.removeItem('jwtToken');  // Clear the JWT token
    goto('/login');  // Redirect to the login page immediately
  }

  // Function to confirm logout
  function confirmLogout() {
    showLogoutConfirm = false;
    logout();
  }

  // Function to cancel logout
  function cancelLogout() {
    showLogoutConfirm = false;
  }

  // Use onMount to handle client-side operations
  onMount(async () => {
    if (typeof window !== 'undefined') {
      // Dynamically import Bootstrap's JS only in the browser
      bootstrap = await import('bootstrap/dist/js/bootstrap.bundle.min.js');

      // Handle click events on nav links to close the offcanvas
      const navLinks = document.querySelectorAll('.navbar-nav .dropdown-item .nav-link:not(.dropdown-toggle)');
      navLinks.forEach(link => {
        link.addEventListener('click', () => {
          if (offcanvasElement) {
            const offcanvasInstance = bootstrap.Offcanvas.getInstance(offcanvasElement);
            if (offcanvasInstance) {
              offcanvasInstance.hide();
            }
          }
        });
      });

      // Add event listener to prevent closing the offcanvas when clicking inside the dropdown
      const dropdownMenus = document.querySelectorAll('.dropdown-menu-no-close');
      dropdownMenus.forEach(menu => {
        menu.addEventListener('click', (event) => {
          event.stopPropagation();
        });
      });
    }
  });

  
</script>

<nav class="navbar navbar-dark bg-dark fixed-top custom-navbar-size">
  <div class="container-fluid">
    <!-- Move the toggler button to the left -->
    <div class="d-flex align-items-center">
      <button class="navbar-toggler me-2" type="button" data-bs-toggle="offcanvas" data-bs-target="#offcanvasDarkNavbar">
        <span class="navbar-toggler-icon"></span>
      </button>
    </div>
    <p class="navbar-brand">Butang Profile diri</p>
    <!-- Offcanvas Menu -->
    <div bind:this={offcanvasElement} class="offcanvas offcanvas-start text-bg-dark custom-offcanvas-size" tabindex="-1" id="offcanvasDarkNavbar">
      <div class="offcanvas-header">
        <h5 class="navbar-brand">ADMIN PAGE</h5>
        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="offcanvas" aria-label="Close"></button>
      </div>
      <div class="offcanvas-body">
        <ul class="navbar-nav justify-content-end flex-grow-1 pe-3">
          <li class="nav-item">
            <a class="nav-link active" href="/Admin">Home</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="/Admin/addAccount/">Add user</a>
          </li>
          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="/" role="button" data-bs-toggle="dropdown" aria-expanded="false">
              Manage User Accounts
            </a>
            <ul class="dropdown-menu dropdown-menu-dark">
              <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/Accounts">View All users</a></li>
              <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/updateAccount">Update User Info</a></li>
              <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/updatePassword">Change Passwords</a></li>
              <li><a bind:this={offcanvasElement} class="dropdown-item" href="/Admin/deleteAccount">Deactivate User</a></li>
              <li><a bind:this={offcanvasElement} class="dropdown-item" href="/">Reactivate User</a></li>
            
            </ul>
          </li>

          <li>
            <hr class="dropdown-divider">
          </li>
          <li class="nav-item">
            <button class="nav-link" on:click={showConfirmLogout}>LOGOUT</button>
          </li>
        </ul>
      
        
      </div>
    </div>
  </div>
</nav>

<div class="app-container">
  <nav class="navbar navbar-expand navbar-dark bg-dark hover">
    <div class="container">
      <div class="navbar-nav me-auto">
   
      </div>
      <div class="navbar-nav ms-auto">
        <button class="nav-item nav-link text-white">LHAHAHAHA</button>
      </div>
    </div>
  </nav>
  <div class="card">
    <div class="card-header"></div>
    <div class="card-body">
      <slot></slot>
    </div>
  </div>
</div>

{#if showLogoutConfirm}
<div class="popup" in:fade>
 
    <p>Are you sure you want to log out?</p>
    <button class="btn btn-outline-danger" on:click={confirmLogout}>Yes</button>
    <button class="btn btn-outline-secondary" on:click={cancelLogout}>No</button>
    <button class="btn btn-outline-success opacity-75" on:click={cancelLogout}>Maybe</button>
 
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

  .custom-offcanvas-size {
    width: 15%; /* Adjust this percentage to control the size of the offcanvas */
    max-width: 15%; /* Ensures the offcanvas doesn't exceed this width */
  }

  .custom-navbar-size {
    height: 10%; /* Adjust this percentage to control the size of the offcanvas */
    max-height: 10%;
  }
</style>
