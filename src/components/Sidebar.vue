<template>
  <div class="sidebar">
    <slot></slot>
  </div>
</template>


<style scoped>
.sidebar {
  width: 300px;
  padding: 20px 20px 20px 30px;
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  background-color: rgb(230, 230, 230);
}
</style>