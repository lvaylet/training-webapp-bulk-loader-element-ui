<template>
  <div id="app">
    <section class="hero is-info is-bold">
      <div class="hero-body">
        <div class="container">
          <h1 class="title">
            Bulk Loader
          </h1>
          <h2 class="subtitle">
            of ODT class requests
          </h2>
        </div>
      </div>
    </section>

    <hello></hello>
  </div>
</template>

<script>
import Hello from './components/Hello'

export default {
  name: 'app',
  components: {
    Hello
  }
}
</script>

<style>
@import "~bulma/css/bulma.css";
@import "~element-ui/lib/theme-default/index.css";
@import "~font-awesome/css/font-awesome.css";
</style>
