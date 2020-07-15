<template>
  <div class="links">
    <button v-on:click="$emit('add-task')">
      <plus-icon size="1.5x" class="custom-class inactive"></plus-icon>
    </button>
    Total Time: {{ totalTime }}
  </div>
</template>

<script>
  import {
    PlusIcon
  } from "vue-feather-icons";

  export default {
    name: 'TodoFooter',
    props: ['tasks', 'totalTime'],
    components: {
      PlusIcon
    }
  }
</script>
<style scoped>
.links {
  position: relative;
}
.links button {
  position: absolute;
  top: 0;
  right: 0;
}
</style>