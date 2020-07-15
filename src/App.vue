<template>
  <div class="container" id="app">
    <div>
      <h1 class="title">Work Tracker</h1>
      <TodoList :tasks="tasks" :totalTime="totalTime"/>
    </div>
  </div>
</template>

<script>
import TodoList from './components/TodoList.vue'
export default {
  name: "App",
  components: {
    TodoList,
  },
  data: function() {
    return {
      tasks: [],
      totalTime: 0,
    }
  },
      mounted() {
      if (localStorage.getItem("tasks")) {
        try {
          this.tasks = JSON.parse(localStorage.getItem("tasks"));
          this.totalTime = 0;
          this.tasks.forEach(task => {
            this.totalTime += task.elapsedTime;
          });
          const date = new Date(null);
          date.setSeconds(this.totalTime / 1000);
          const utc = date.toUTCString();
          this.totalTime = utc.substr(utc.indexOf(":") - 2, 8);
        } catch (e) {
          localStorage.removeItem("tasks");
        }
      }
    },
};
</script>

<style>
.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #55A630;
  letter-spacing: 1px;
}

.links {
  padding-top: 15px;
}
button {
  border-radius: 50%;
  padding: 5px 5px 2px 5px;
  border: none;
  box-shadow: 1px 1px 5px 1px gainsboro;
  cursor: pointer;
  outline: none;
  background: #f1faee;
}
button:hover {
  transform: scale3d(1.1, 1.1, 1.1);
}
button.active {
  transform: scale3d(1.2, 1.2, 1.2);
}
body {
  scrollbar-color: red yellow;
}
h1 {
  font-family: "Montserrat";
  font-size: 2.5rem;
  color: #2a9d8f;
  text-align: left;
}
.active {
  stroke: #2a9d8f;
}
.inactive {
  stroke: #e63946;
}
ul {
  list-style: none;
  /* height: 500px;
  align-items: flex-end;
  overflow-y: scroll;
  scrollbar-face-color: #2a9d8f; */
}
ul::-webkit-scrollbar {
  width: 12px;
  padding-right: 10px;
}

/* Track */
ul::-webkit-scrollbar-track {
  background: #a8dadc;
}
/* Handle */
ul::-webkit-scrollbar-thumb {
  background: #2a9d8f;
}
input {
  border: 1px solid gainsboro;
  padding: 10px;
  box-shadow: 1px 1px 1px 1px gainsboro;
  border-radius: 5px;
  margin-right: 10px;
  margin-bottom: 15px;
  min-width: 300px;
}

body {
  background: #efefef;
}
a {
  stroke: #252627;
}
.custom-class {
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
