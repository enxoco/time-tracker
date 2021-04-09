<template>
  <table>
    <thead>
      <tr>
        <td>Client/Project</td>
        <td>Job #</td>
        <td>Total Time</td>
        <td>Mon</td>
        <td>Tues</td>
        <td>Wed</td>
        <td>Thur</td>
        <td>Fri</td>
        <td>Sat</td>
        <td>Sun</td>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(task, index) in timesheet" :key="index">
        <td>{{ task["name"] }}</td>
        <td>{{ task["code"] }}</td>
        <td>{{ getWeeklyTime(task["name"]) }}</td>
        <td v-for="(day, i) in dateCodes" :key="i">
          <div v-for="(task, x) in task['tasks']" :key="x">
            <span v-if="task.date == day" class="date-time">{{
              roundTime(task.elapsedTime)
            }}</span>
            <!-- <span v-else>0</span> -->
          </div>
        </td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <td></td>
        <td></td>
        <td>
          <strong>{{ getTotalTime() }}</strong>
        </td>
        <td v-for="i in [1, 2, 3, 4, 5, 6, 7]" :key="i">
          <strong>{{ getDailyTime(i) }}</strong>
        </td>
      </tr>
    </tfoot>
  </table>
</template>
<script>
export default {
  props: ["timesheet", "dateCodes", "tasks"],
  methods: {
    getDailyTime(index) {
      var curr = new Date(); // get current date
      var first = curr.getDate() - curr.getDay() + +index; // First day is the day of the month - the day of the week
      let [month, day] = new Date(curr.setDate(first))
        .toLocaleDateString("en-US")
        .split("/");

      let totalTime = 0;
      this.tasks.map((task) => {
        if (task.date == `${month}/${day}`) totalTime += task.elapsedTime;
      });

      return this.roundTime(totalTime)
    },
    getTotalTime() {
      let totalTime = 0;
      this.tasks.map((task) => {
        totalTime += task.elapsedTime;
      });
      return this.secondsToHms(totalTime);

      // return date.toISOString().substr(11,8)
    },
    getWeeklyTime(taskName) {
      let totalTime = 0;
      this.tasks.map((task) => {
        if (task.item === taskName) {
          totalTime += task.elapsedTime;
        }
      });
      return this.roundTime(totalTime)
    },

    secondsToHms(d) {
      d = Number(d);
      var h = Math.floor(d / 3600);
      var m = Math.floor((d % 3600) / 60);

      if (m < 15) {
        m = 25;
      } else if (m < 30) {
        m = 50;
      } else if (m < 45) {
        m = 75;
      } else if (m > 45 && m < 59) {
        h += 1;
        m = "00";
      }

      return `${h}.${m}`;
    },
    roundTime(time){
    const date = new Date(null);
      date.setSeconds(time);
      const utc = date.toISOString();
      let [h, m] = utc.substr(11, 5).split(":");

      if (h < 10) {
        h = h.split("")[1];
      }
      if (m == 0) {
        return `${h}`;
      } else if (m > 1 && m <= 15) {
        m = 25;
        return `${h}.${m}`;
      } else if (m <= 30) {
        m = 5;
        return `${h}.${m}`;
      } else if (m <= 45) {
        m = 75;
        return `${h}.${m}`;
      } else {
        m = 0;
        h = h + 1;
        return `${h}.${m}`;
      }
    }
  },
};
</script>
<style scoped>
table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
  background: white;
  border-radius: 10px;
  box-shadow: 2px 2px 5px 2px #eeeeee;
  position: relative;
  top: 100px;
}

td,
th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}
thead tr {
  font-weight: bold;
}
@media print {
  .header,
  .copyright {
    display: none !important;
  }
}
</style>
