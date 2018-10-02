<template>
  <div>
    <b-navbar toggleable="md" type="dark" variant="muted" fixed="top">
      <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
      <b-navbar-brand href="#">
        <b-link :to="{ name: 'Home'}">Home</b-link>
      </b-navbar-brand>
      <b-collapse is-nav id="nav_collapse">

        <b-navbar-nav class="ml-auto">
          <!-- <b-nav-item type="submit" v-b-tooltip.hover title="Buscas cadastradas">
            <b-link :to="{ name: 'Buscas'}">Buscas</b-link>
          </b-nav-item> -->
          <b-link :to="{ name: 'Buscas'}" class="nav-link">Buscas</b-link>
        </b-navbar-nav>

      </b-collapse>
    </b-navbar>
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
  body {
    background-color: #dee2e6;
  }
  .navbar {
    background: rgba(0, 0, 0, 1);
  }
  a, a:hover {
    text-decoration: none;
    color: inherit
  }
</style>
