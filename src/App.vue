<template>
 <div class="wrapper" id="app">
   <div class="header">
     <span>
       <h1>Work Tracker</h1>
       <p>Your To-Do list and time tracking simplified</p>
     </span>
     <span class="add-task-button" v-on:click="addTask">&#43;</span>
   </div>
   <span class="right">Total Time: {{totalTime}}</span>
      <TodoList :tasks="tasks"/>
      <span class="copyright">Copyright 2020 <a href="https://iliveinaterminal.com">Mike Conrad</a>.  Part of <a href="https://theopensuite.com">The Open Suite</a><a href="https://github.com/enxoco/time-tracker" style="float:right;">Get the code</a></span>
 </div>
</template>

<script>
import TodoList from "./components/TodoList.vue";
export default {
  name: "App",
  components: {
    TodoList
  },
  data: function() {
    return {
      tasks: [],
      totalTime: 0
    };
  },
  methods: {
    addTask() {
      this.tasks.push({
        item: null,
        timer: 0,
        elapsedTime: 0,
        active: false,
        id: 0
      });
      this.saveTasks();
    },
    saveTasks() {
      const parsed = JSON.stringify(this.tasks);
      localStorage.setItem("tasks", parsed);
      this.pendingDelete = null;
      this.modalActive = false;
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
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Oxygen:wght@400;700&display=swap");
* {
  margin: 0px;
}

body {
  font-family: "Oxygen";
  background: #eeeeee;
  color: #222831;
}

.wrapper {
  padding: 2.5% 10%;
}
.wrapper .header {
  display: grid;
  grid-template-columns: 88% auto;
}
.wrapper .header .add-task-button {
  color: white;
  font-size: 48px;
  font-weight: 400;
  background: #32e0c4;
  height: 80px;
  width: 80px;
  border-radius: 50%;
  border: 5px solid #393e46;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
}
.wrapper .header .add-task-button:hover {
  color: #32e0c4;
  background: white;
}
.wrapper .header p {
  padding-left: 10px;
}

h1 {
  font-weight: 700;
  font-size: 60px;
  margin: 0px;
}

.task-list {
  width: 100%;
  padding-left: 0px;
  margin-left: 0px;
  margin-top: 50px;
}

.task {
  list-style: none;
  position: relative;
  display: grid;
  grid-template-columns: 60% 15% auto auto;
  margin-bottom: 30px;
}
.task input {
  width: 100%;
  box-shadow: 1px 6px 8px rgba(57, 62, 70, 0.4);
  border: 0px;
  padding-left: 30px;
  border-radius: 10px;
  font-size: 24px;
  font-weight: 400;
}
.task span {
  /* position: absolute;
  right: -32px; */
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px;
  /* top: 0;
  bottom: 29px; */
  background: #32e0c4;
  box-shadow: 1px 6px 8px rgba(57, 62, 70, 0.4);

  color: white;
  padding: 30px 40px;
}

.right {
  float: right;
  font-size: 14px;
  margin-right: 25%;
  margin-top: 20px;
}
@media (max-width: 425px) {
  .wrapper {
    padding: 20px;
  }
  .add-task-button {
    position: absolute;
    top: 20px;
    right: 20px;
  }
  .task span {
    padding: 10px;
  }
  .task {
    grid-template-columns: auto auto auto auto;
  }
  .task input {
    padding-left: 10px;
  }
}
.copyright {
    position: absolute;
    bottom: 20px;
    left: 10%;
    right: 30%;
}
.copyright a {
  font-weight: bold;
  text-decoration: none;
  color:#393e46;
}


</style>
