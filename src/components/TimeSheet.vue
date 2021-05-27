<template>
  <div>
    <h2>Timesheet for Mike Conrad Week ending {{ dateCodes[6] }}</h2>

    <table>
      <thead>
        <tr>
          <td>Client/Project</td>
          <td>Job #</td>
          <td>Total Time</td>
          <td>Mon <br/> {{ dateCodes[0] }}</td>
          <td>Tues <br/> {{ dateCodes[1] }}</td>
          <td>Wed <br/> {{ dateCodes[2] }}</td>
          <td>Thurs <br/> {{ dateCodes[3] }}</td>
          <td>Fri <br/> {{ dateCodes[4] }}</td>
          <td>Sat <br/> {{ dateCodes[5] }}</td>
          <td>Sun <br /> {{ dateCodes[6] }}</td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(task, index) in filteredTasks" :key="index">
          <td contenteditable="true">{{ task.name }}</td>
          <td contenteditable="true">{{ task.code }}</td>
          <td contenteditable="true">{{ getWeeklyTime(task.name) }}</td>
          <td v-for="(day, i) in dateCodes" :key="i">
            <div v-for="(task, x) in task['tasks']" :key="x">
              <span v-if="task.date == day" class="date-time" contenteditable="true">{{
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
          <td contenteditable="true">
            <strong>{{ totalTime }}</strong>
          </td>
          <td v-for="i in [1, 2, 3, 4, 5, 6, 7]" :key="i" contenteditable="true">
            <strong>{{ getDailyTime(i) }}</strong>
          </td>
        </tr>
      </tfoot>
    </table>
  </div>
</template>
<script>
export default {
  props: ["timesheet", "dateCodes", "tasks", "totalTime", "weekOffset"],
  data() {
    return {
      totalTimeWeek: 0
    }
  },

  methods: {
    getDailyTime(index) {
      var curr = new Date(); // get current date
      var first = curr.getDate() - curr.getDay() + +index - this.weekOffset; // First day is the day of the month - the day of the week
      let [month, day] = new Date(curr.setDate(first))
        .toLocaleDateString("en-US")
        .split("/");

      let totalTime = 0;
      this.tasks.map((task) => {
        if (task.date == `${month}/${day}`) {
          totalTime += task.elapsedTime;
        }
      });

      return this.roundTime(totalTime)
    },
    getTotalTime() {
      let totalTime = 0;
      this.tasks.map((task) => {
        if (task.date === this.getTimeStamp) {
          totalTime += task.elapsedTime;
        }
        
      });
      let fullWeek = this.secondsToHms(totalTime)
      if (fullWeek === '40.25'){
        return '40'
      } else {
        return fullWeek
      }

      // return date.toISOString().substr(11,8)
    },
        //Get a timestamp using the selected day and week.
    getTimeStamp(){
      const { selectedDay: today, weekOffset: thisWeek } = this.preferences
      var curr = new Date;
      var first = curr.getDate() - curr.getDay() + +today - thisWeek; 
      let [month, day] = new Date(curr.setDate(first)).toLocaleDateString("en-US").split("/");
      return `${month}/${day}`
    },
    getWeeklyTime(taskName) {
      let totalTime = 0;
      this.tasks.map((task) => {
        if (task.item === taskName && this.dateCodes.includes(task.date) ) {
          totalTime += task.elapsedTime;
        }
      });
      this.totalTimeWeek += +this.roundTime(totalTime)
      // this.totalTimeWeek = this.totalTimeWeek
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
  computed: {
    filteredTasks(){
      
      // let tasks = []
      // for (var i = 0; i < this.timesheet.length; i++){
      //   if (this.timesheet[i].tasks.length != 0){
      //     tasks.push(this.timesheet[i])
      //   }
      // }
      // return tasks
      let headings = Object.keys(this.timesheet).filter(task => this.timesheet[task].tasks.length != 0)
      let filtered = []
      headings.map(key => filtered.push(this.timesheet[key]))
      return filtered
    }
  },
  mounted() {
      //     let taskHeadings = [];
      // this.timesheet.map((task) => {
      //   taskHeadings.push(task.item);
      // });
      // this.taskHeadings = [...new Set(taskHeadings)];
      // let days = []
      // this.timesheet.map(task => {
      //   days.push(task.date)
      // })
      // days = [...new Set(days)]
      // //Second we need to split the tasks up by days.
      
      // //Create an object to hold our timesheet data.
      // let timesheet = {}
      // //Map over our task names and make each one an object key
      // this.taskHeadings.map(name => {
      //   timesheet[name] = []
      // })
      // //map over our tasks and taskheading assigning each task to its associated taskheading object
      // this.tasks.map(task => {
      //   this.taskHeadings.map((name) => {
      //     if (task.item === name){
      //       timesheet[name] = {name, code: task.code, dates: [], tasks: []}
      //     }
      //   }
      //   )
      // })
      // //Get days/times associated with a task
      // this.taskHeadings.map(name => {
      //   timesheet[name].tasks = this.tasks.filter(item => item.item === name)
      // })
      // this.timesheet = null
      // //timesheet object is sorted by days so monday is 0, tuesday is 1, wednesday is 2, etc.
      // this.timesheet = timesheet
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
