<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import 'bootstrap/dist/css/bootstrap.min.css';
    import backgroundImage from '$lib/images/background.png';
    import logo from '$lib/images/bc-logo.png';

    let username = '';
    let password = '';
    let error = '';
    

  
    onMount(async () => {
 
  if (!localStorage.getItem('x')) {
    localStorage.setItem('x', 'true');
    // Reload the page
    window.location.reload();
  } else {
    localStorage.removeItem('x');
  }
});




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
                throw new Error('incorrect username and/or passowrd :(');
            }

            const data = await response.json();
            const role = data.role;

            // Save JWT token to local storage or cookies as needed
            localStorage.setItem('jwtToken', data.jwtToken);

            // Redirect based on the role
            if (role === 'Admin') {
                goto('/Admin');
            } else if (role === 'Registrar') {
                goto('/Registrar');
            } else if (role === 'Teacher') {
                goto('/Teacher');
            } else if (role === 'Student') {
                goto('/Student');
            } else {
                throw new Error('Unknown role');
            }
        } catch (err) {
            error = err.message;
        }
    }
</script>




<!-- 
<h3 class="card-header">Login</h3>


<div class="card-body">
<form on:submit|preventDefault={handleLogin}>

    
    <label for="username">Username:</label>
    <input id="username" bind:value={username} required />

    <label for="password">Password:</label>
    <input id="password" type="password" bind:value={password} required />

    <button type="submit">Login</button>
</form>
</div> -->


<div class="parent-container">
<div class="form__container">
    <form on:submit|preventDefault={handleLogin}>
      <div class="header__form">
       
        <img src={logo} alt="Logo" />
      
      </div>

      <div class="main__input__container">
      
        <div class="input-container">
          <input type="text" id="username" bind:value={username} required />
          <label for="username" class="label">Username</label>
          <div class="underline"></div>
        </div>
        <div class="input-container">
          <input type="password" id="password" bind:value={password} required/>
          <label for="password" class="label">Password</label>
          <div class="underline"></div>
   
        </div>
      
        <div class="input-container">
            {#if error}
          <p style="color: red">{error}</p>
            {/if}
        </div>
        <div class="button">
          <button type="submit" class="sign__in__button">Sign In</button>
        </div>

        
      </div>
    </form>
  </div>

</div>



<style>

*,
*::before,
*::after {
  box-sizing: border-box;
}
* {
  margin: 0;
}





.parent-container {
  display: flex;
  justify-content: center;  /* Center horizontally */
  align-items: center;      /* Center vertically */
  height: 100vh;            /* Full viewport height */
}

.form__container {
  width: 500px;
  padding: 3rem 0;
  background-color: white;
  border-radius: 10px;
  margin: 0 auto;
  
  border: 1px solid rgb(8, 8, 8);
 
}

.header__form {
  display: flex;
  align-items: center;
  margin-bottom: 4rem;
  justify-content: center;
  gap: 1rem;
}

.header__form img {
  width: 100px;
  height: 100px;
}

/* INPUT */

.input-container {
  position: relative;
  margin: 2.5rem auto;
  width: 300px;
}

.input-container input,
.input-container input {
  font-size: 1rem;
  width: 100%;
  border: none;
  border-bottom: 2px solid #ccc;
  padding: 5px 0;
  background-color: transparent;
  outline: none;
}

.input-container .label {
  position: absolute;
  top: 0.5rem;
  left: 0;
  font-size: 0.925rem;
  color: #050505;
  transition: all 0.3s ease;
  pointer-events: none;
}

.input-container input[type="text"]:focus ~ .label,
.input-container input[type="text"]:valid ~ .label {
  top: -20px;
  font-size: 0.925rem;
  color: #ccc;
}

.input-container input[type="password"]:focus ~ .label,
.input-container input[type="password"]:valid ~ .label {
  top: -20px;
  font-size: 0.925rem;
  color: #ccc;
}

.input-container .underline {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 2px;
  width: 100%;
  background-color: orangered;
  transform: scaleX(0);
  transition: all 0.3s ease;
}

.input-container input[type="text"]:focus ~ .underline,
.input-container input[type="text"]:valid ~ .underline {
  transform: scaleX(1);
}

.input-container input[type="password"]:focus ~ .underline,
.input-container input[type="password"]:valid ~ .underline {
  transform: scaleX(1);
}



.button {
  text-align: center;
  margin: 2rem 0;
}

.sign__in__button {
  cursor: pointer;
  border: none;
  background-color: rgb(25, 25, 177);
  width: 200px;
  color: white;
  border-radius: 10px;
  padding: 0.8rem 0;
}

.sign__in__button:hover {
  transition: 200ms;
  background-color: orangered;
}

</style>