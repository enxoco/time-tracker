<template>
  <div class="flex">
    <ul>
      <li v-for="(task, index) in tasks" :data-index="index" v-bind:key="index">
        <input v-model="task.item" @keyup="saveTasks()" />
        <button @click="start(index)" :class="{ active: task.active }">
          <play-icon size="1.5x" class="custom-class active" :class="{ active: task.active }"></play-icon>
        </button>
        <button @click="stop(index)">
          <pause-icon size="1.5x" class="custom-class" :class="{ inactive: task.active }"></pause-icon>
        </button>
        <button @click="deleteTask(index)">
          <x-icon size="1.5x" class="custom-class inactive"></x-icon>
        </button>
        <span class="timer">{{ (formattedElapsedTime(index)) ? formattedElapsedTime(index) : "00:00:00" }}</span>
      </li>
    </ul>
    <TodoFooter v-on:add-task="addTask()"/>
  </div>
</template>

<script>
  import {
    PauseIcon,
    PlayIcon,
    XIcon,
  } from "vue-feather-icons";
  import * as workerTimers from "worker-timers";
  import TodoFooter from './TodoFooter.vue'

  export default {
    name: 'TodoList',
    data: function () {
      return {
        tasks: [],
        totalTime: 0
      }
    },
    components: {
      PlayIcon,
      XIcon,
      PauseIcon,
      TodoFooter
    },
    props: ['tasks', 'totalTime'],
    methods: {
      addTask() {
        this.tasks.push({ item: null, timer: 0, elapsedTime: 0, active: false });
        this.saveTasks();
        console.log('save')
      },
      saveTasks() {
        const parsed = JSON.stringify(this.tasks);
        localStorage.setItem("tasks", parsed);
      },

      formattedElapsedTime(index) {
        if (this.tasks[index].elapsedTime) {
          const date = new Date(null);
          date.setSeconds(this.tasks[index].elapsedTime / 1000);
          const utc = date.toUTCString();
          return utc.substr(utc.indexOf(":") - 2, 8);
        }
      },
      start(index) {
        if (this.tasks[index].active) {
          return;
        }
        this.tasks[index].timer = workerTimers.setInterval(() => {
          this.tasks[index].elapsedTime += 1000;
          this.saveTasks();
        }, 1000);
        this.tasks[index].active = true;
      },
      stop(index) {
        workerTimers.clearInterval(this.tasks[index].timer);
        this.tasks[index].active = false;
        this.saveTasks();
      },
      reset(index) {
        this.tasks[index].elapsedTime = 0;
        this.tasks[index].active = false;
        this.saveTasks();
      },
      deleteTask(index) {
        this.tasks.splice(index, 1);
        this.saveTasks();
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.flex {
  flex-direction: column;
  background: white;
  box-shadow: 2px 2px 5px 1px gainsboro;
  border-radius: 5px;
  padding-right: 20px;
  padding-top: 20px;
  padding-bottom: 20px;
}
</style>