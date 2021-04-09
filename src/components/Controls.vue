<template>
  <div class="flex">
    <ul>
      <li v-for="(task, index) in tasks" :data-index="index" v-bind:key="index">
        <input v-model="task.item" @keyup="saveTasks()" />
        <button @click="start(task.id)" :class="{ active: task.active }">
          <play-icon size="1.5x" class="custom-class active" :class="{ active: task.active }"></play-icon>
        </button>
        <button @click="stop(task.id)">
          <pause-icon size="1.5x" class="custom-class" :class="{ inactive: task.active }"></pause-icon>
        </button>
        <button @click="deleteTask(task.id)">
          <x-icon size="1.5x" class="custom-class inactive"></x-icon>
        </button>
        <span class="timer">{{ (formattedElapsedTime(task.id)) ? formattedElapsedTime(task.id) : "00:00:00" }}</span>
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

      saveTasks() {
        const parsed = JSON.stringify(this.tasks);
        localStorage.setItem("tasks", parsed);
      },

      formattedElapsedTime(index) {
        let task = this.tasks.filter(task => task.id == index)
        if (task.elapsedTime) {
          const date = new Date(null);
          date.setSeconds(task.elapsedTime / 1000);
          const utc = date.toUTCString();
          return utc.substr(utc.indexOf(":") - 2, 8);
        }
      },
      start(index) {
        let task = this.tasks.filter(task => task.id == index)

        if (task.active) {
          return;
        }
        task.timer = workerTimers.setInterval(() => {
          task.elapsedTime += 1000;
          this.saveTasks();
        }, 1000);
        task.active = true;
      },
      stop(index) {
        let task = this.tasks.filter(task => task.id == index)

        workerTimers.clearInterval(task.timer);
        task.active = false;
        this.saveTasks();
      },
      reset(index) {
        let task = this.tasks.filter(task => task.id == index)

        task.elapsedTime = 0;
        task.active = false;
        this.saveTasks();
      },
      deleteTask(index) {
        this.tasks.filter(task => task.id != index)
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