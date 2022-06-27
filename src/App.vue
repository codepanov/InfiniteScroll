<template>
  <div id="app">
    <infinite-scroll></infinite-scroll>
  </div>
</template>

<script>
import InfiniteScroll from "./components/InfiniteScroll";

export default {
  name: "App",
  components: { InfiniteScroll },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
