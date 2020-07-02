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
    },
    mounted() {
      if (localStorage.getItem("tasks")) {
        try {
          this.tasks = JSON.parse(localStorage.getItem("tasks"));
        } catch (e) {
          localStorage.removeItem("tasks");
        }
      }
    },
  }
</script>
<style scoped>

</style>