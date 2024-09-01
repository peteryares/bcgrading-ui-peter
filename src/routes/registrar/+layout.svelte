<script>
    import 'bootstrap/dist/css/bootstrap.min.css';
    import { onMount } from 'svelte';
    import { page } from '$app/stores';
    import { fade } from 'svelte/transition';
    import { goto } from '$app/navigation';
  
    let offcanvasElement;
    let bootstrap;  // Variable to hold the imported Bootstrap module
    // let showLogoutConfirm = false;
    // on:click={showConfirmLogout}
  
    
  
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
    // function showConfirmLogout() {
    //   showLogoutConfirm = true;
    // }
  
    // Function to handle logout
    function logout() {
      localStorage.removeItem('jwtToken');  // Clear the JWT token
      goto('/login');  // Redirect to the login page immediately
    }
  
    // Function to confirm logout
    // function confirmLogout() {
    //   showLogoutConfirm = false;
    //   closeOffcanvas();
    //   logout();
    // }
  
    // // Function to cancel logout
    // function cancelLogout() {
    //   showLogoutConfirm = false;
    // }
  
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
  
  <nav class="navbar fixed-top custom-navbar-size">
    <div class="container-fluid">
      <!-- Move the toggler button to the left -->
      <div class="d-flex align-items-center gap-2">
        <div>
          <a href="/Admin"><img src="https://cebu.mis.benedictocollege.edu.ph/assets/logo-21a9a44cc070aa7b0436551dba367c97e53bce3864cb2151d4ed24682b8ae540.png" alt="" class="logo"></a>
        </div>
  
        <button class="navbar-toggler bg-transparent navbar-button" type="button" data-bs-toggle="offcanvas" data-bs-target="#offcanvasDarkNavbar">
          <img src="https://assets.website-files.com/62e8f5c9dbfdccaf94d287ac/62ea9a21af559238cc12a830_menu_FILL0_wght400_GRAD0_opsz48%20(2).svg" alt="" class="navbar-toggler-icon navbar-span">
        </button>
  
        
      </div>
      <div>
        <img src="/src/lib/images/profile-circle.svg" alt="" class="profile">
      </div>
      
      <!-- Offcanvas Menu -->
      <!-- <div bind:this={offcanvasElement} class="offcanvas offcanvas-start text-white custom-offcanvas-size navbar" tabindex="-1" id="offcanvasDarkNavbar">
        <div class="offcanvas-header">
          <img src="/src/lib/images/profile-circle.svg" alt="" class="profile">
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="offcanvas" aria-label="Close"></button>
        </div>
        <div class="offcanvas-body">
          <nav>
            <ul class="navbar-nav justify-content-end flex-grow-1 pe-3">
              <li class="nav-item">
                <a class="nav-link active" href="/Registrar">Home</a>
              </li>
              <li class="nav-item">
                <a class="nav-link" href="/Registrar/ClassList/">List of Classes </a>
              </li>
              <li class="nav-item">
                <a class="nav-link" href="/Registrar/AddSubject/">Add Subjects</a>
              </li>
           
              <li>
                <hr class="dropdown-divider">
              </li>
            
            </ul>
          </nav>
        
          <button type="button" class="nav-link btn btn-md btn-danger p-2" data-bs-toggle="modal" data-bs-target="#staticBackdrop">
            Logout 
             </button>
        </div>
      </div> -->

      <div bind:this={offcanvasElement} class="offcanvas offcanvas-start text-white custom-offcanvas-size " tabindex="-1" id="offcanvasDarkNavbar">
        <div class="offcanvas-header">
          <div class="d-flex align-items-center justify-content-between gap-2">
            <img src="/src/lib/images/profile-circle.svg" alt="" class="profile">
         
          </div>
          <button type="button" class="btn-close btn-close-white" data-bs-dismiss="offcanvas" aria-label="Close">
           
          </button>
        </div>
        <div class="offcanvas-body d-flex flex-column justify-content-between">
          <nav>
            <ul class="navbar-nav justify-content-end flex-grow-1 pe-3 ">
            
              <li class="nav-item">
                <a class="nav-link" href="/Registrar">
                  <img src="/src/lib/images/home.svg" alt="">
                  Home</a>
              </li>
              <li class="nav-item">
                <a class="nav-link" href="/Registrar/ClassList/">
                  <img src="/src/lib/images/th-large.svg" alt="" class="ml-3 box">
                  List of Classes </a>
              </li>
              <li class="nav-item">
                <a class="nav-link" href="/Registrar/AddSubject/">
                  <img src="/src/lib/images/customer-lists-fill.svg" alt="">
                  Add Subjects</a>
              </li>
           
            </ul>
          </nav>
        
          
          <button type="button" class="btn btn-sm btn-danger p-2" data-bs-toggle="modal" data-bs-target="#staticBackdrop">
            Logout 
             </button>
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
  
  <!-- Button trigger modal -->
  
  
  <!-- Modal logout-->
  <div class="modal fade" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered modal-md">
      <div class="modal-content ">
        
        <div class="modal-body">
          <p class="text-center fw-bold fs-5">You are logging out of Benedicto College Grading System.</p>
          <p class="text-center mt-2 text-body-secondary">Are you sure you want to log out?</p>
      </div>
        <div class="modal-footer">
        
    <div class="w-100">
          <div class="d-flex justify-content-center align-items-center gap-2">
            <button type="button" class="btn  btn-danger  w-100" on:click={logout}>Yes</button>
            <button type="button" class="btn  btn-secondary w-100" data-bs-dismiss="modal">No </button>
          </div>
      </div>
    </div>
      </div>
    </div>
  </div>
  
  <!-- {#if showLogoutConfirm}
  {/if} -->
  
  <!-- <div class="popup" in:fade>
    <div class="modal-content center">
    <div class="modal-footer">
      <p>Are you sure you want to log out?</p>
      <button class="btn btn-outline-danger" on:click={confirmLogout}>Yes</button>
      <button class="btn btn-outline-secondary" on:click={cancelLogout}>No</button>
      <button class="btn btn-outline-success opacity-75" on:click={cancelLogout}>Maybe</button>
    </div>
  </div>
  </div> -->
  
  
  <style>
  
    .custom-offcanvas-size {
      width: 15%; /* Adjust this percentage to control the size of the offcanvas */
      max-width: 15%; /* Ensures the offcanvas doesn't exceed this width */
      background-color: #001A56;
    }
  
    .custom-navbar-size {
      height: 10%; /* Adjust this percentage to control the size of the offcanvas */
      max-height: 10%;
    }

    .dropdown-menu{
    border: none !important;
  }

  .navbar{
    background-color: #001A56;
  }

  .dropdown-item:hover{
    background-color: transparent !important;
    color: #FF6100 !important;
  }

  .logo{
  
    width: 100%;
    height: 30px;
  
  }

  .navbar-button{
    outline: none;
    border: none;
  }

  .profile{
    width: 40px;
    height: 40px;
    filter: invert(1);
  }

  li > a > img{
    filter: invert(1);
  }

  li > a{
    color: white;
    display: flex;
    gap: 1rem;
    
  }

  a::after{
    display: none;

  }

  a:hover{
    color: #FF6100 !important;
  }

  .box{
    height: 18px;
    margin-left: 3px;
    width: 18px;
  }
  </style>
  

