  <!-- src/routes/__layout.svelte -->
<script>
  import 'bootstrap/dist/css/bootstrap.min.css';
</script>



<div class="app-container">
  <nav class="navbar navbar-expand navbar-dark bg-dark">
    <div class="navbar-nav">
    <a class="nav-item nav-link text-white" href="/admin">ADMIN ACCOUNT</a>
    <a class="nav-item nav-link text-white" href="/years">SchoolYear</a>
    <a class="nav-item nav-link text-white" href="/settings">Settings</a>
  </div>
  </nav>
  <div class="card">
    <div class="card-header">
      Layout Header
    </div>
    <div class="card-body">
      <slot></slot>
    </div>
  </div>
</div>
