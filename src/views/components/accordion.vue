<template>
  <div class="dashboard p-4">
    <!-- end nav -->
    <!-- grid wrapper card -->
    <div class="flex justify-center">
      <div class="text-center">
        <button
          class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-800 rounded-full disabled:opacity-50 flex items-center"
          @click="resetZoom">
          <Icon icon="tabler:zoom-reset" /> Reset Zoom
        </button>
      </div>
    </div>

    <div class="wrapper-card grid lg:grid-cols-4 grid-cols-1 md:grid-cols-2 gap-2 mt-5">
      <canvas id="myChart" height="100"></canvas>
    </div>

    <!-- end wrapper card -->

    <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">
      <h2 class="font-bold text-lg text-gray-800 dark:text-gray-200">
        Activity Infomation
      </h2>
      <p class="text-gray-400 font-lexend font-normal">
        This is a list of latest activities detection
      </p>
      <div class="wrapping-table mt-10">
        <div class="wrapper-button p-5 flex justify-between mt-3">

          <!-- Camera Select Box -->
          <select v-model="selectedCamera"
            class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
            <option value="All Cameras">All Cameras</option>
            <option v-for="camera in distinctCameras" :key="camera" :value="camera">
              Camera {{ camera }}
            </option>
          </select>

          <!-- House Select Box -->
          <select v-model="selectedHouse"
            class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
            <option value="All Houses">All Houses</option>
            <option v-for="house in distinctHouses" :key="house" :value="house">
              House ID {{ house }}
            </option>
          </select>

          <!-- Start Date Select Box -->
          <select v-model="selectedStartDate"
            class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
            <option value="All Start Dates">All Start Dates</option>
            <option v-for="date in distinctStartDates" :key="date" :value="date">
              {{ date }}
            </option>
          </select>

          <!-- End Date Select Box -->
          <select v-model="selectedEndDate"
            class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
            <option value="All End Dates">All End Dates</option>
            <option v-for="date in distinctEndDates" :key="date" :value="date">
              {{ date }}
            </option>
          </select>

        </div>
      </div>

      <div class="wrapping-table mt-10">
        <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400 lg:overflow-auto overflow-x-scroll">
          <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
            <tr>
              <td class="px-6 py-3">Time</td>
              <td class="px-6 py-3">Type</td>
              <td class="px-6 py-3">Family Member</td>
              <td class="px-6 py-3">House ID</td>
              <td class="px-6 py-3">Camera ID</td>
              <td class="px-6 py-3">Image Capture</td>
            </tr>
          </thead>
          <tbody>
            <!-- Loop through the activities for the current page -->
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 odd:bg-white even:bg-gray-50" 
                v-for="(activity, index) in currentActivities" 
                :key="activity.activity_id">

              <td class="px-6 py-4">{{ activity.timestamp }}</td>
              <td class="px-6 py-4">
                <span :class="getActivityColor(activity.activity_type) + ' px-3 py-1 rounded-md'">
                  {{ activity.activity_type }}
                </span>
              </td>
              <td class="px-6 py-4">{{ activity.family_member }}</td>
              <td class="px-6 py-4">{{ activity.house_id }}</td>
              <td class="px-6 py-4">{{ activity.camera_id }}</td>
              <td class="px-6 py-4">
                <img :src="'data:image/png;base64,' + activity.image" alt="Thumbnail" style="height: 100px;" />
              </td>
            </tr>
          </tbody>
        </table>

        <!-- Pagination Controls -->
        <div class="flex justify-center mt-4 space-x-2">
          <!-- Button to navigate to the previous page -->
          <button class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-800 rounded-full disabled:opacity-50"
            @click="previousPage" :disabled="currentPage === 1">
            Previous
          </button>

          <!-- Buttons to jump to specific pages -->
          <span v-for="page in totalPages" :key="page">
            <button
              class="px-3 py-2 bg-blue-200 hover:bg-blue-300 text-blue-800 rounded-full disabled:bg-blue-400 disabled:opacity-50"
              @click="goToPage(page)" :disabled="currentPage === page">
              {{ page }}
            </button>
          </span>

          <!-- Button to navigate to the next page -->
          <button class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-800 rounded-full disabled:opacity-50"
            @click="nextPage" :disabled="currentPage === totalPages">
            Next
          </button>
        </div>

      </div>


    </div>
  </div>
</template>

<!-- //....................................................
//.............................iii....................
//.............................iii.............tttt...
//.............................iii.............tttt...
//.............................iii.............tttt...
//..ssssssss...ccccccc..rrrrrr.iii.ipppppppp..pttttt..
//..ssssssss..ccccccccc.rrrrrr.iii.ippppppppp.pttttt..
//.sssssssssssccccccccc.rrrrrr.iii.ipppppppppppttttt..
//.sssssssssssccc..cccc.rrrr...iii.ipppp.ppppp.tttt...
//..ssssssss.sccc.......rrr....iii.ippp...pppp.tttt...
//..ssssssss.sccc.......rrr....iii.ippp...pppp.tttt...
//.sssssssssssccc..cccc.rrr....iii.ippp...pppp.tttt...
//.sssss.sssssccccccccc.rrr....iii.ipppppppppp.tttt...
//.ssssssssss.cccccccc..rrr....iii.ippppppppp..ttttt..
//..ssssssss..cccccccc..rrr....iii.ippppppppp..ttttt..
//...ssssss.....cccc...............ippp.ppp......ttt..
//.................................ippp...............
//.................................ippp...............
//.................................ippp...............
//.................................................... -->



<style>
.timeline-container {
  position: relative;
  width: 100%;
  display: flex;
  padding: 40px 20px;
}

.y-axis {
  width: 4px;
  background-color: #000;
  margin-right: 20px;
}

.timeline {
  position: relative;
  width: 100%;
  display: flex;
  flex-direction: column;
}

.timeline-line {
  position: absolute;
  left: 10px;
  top: 0;
  bottom: 30px;
  width: 4px;
  background-color: #007bff;
}

.grid-line { /* Styling for the grid lines */
  position: absolute;
  left: 10px;
  width: 4px;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.1);
}

.timeline-item {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
  padding: 20px;
}

.timeline-marker {
  width: 20px;
  height: 20px;
  background-color: #007bff;
  border-radius: 50%;
  position: relative;
  margin-bottom: 10px;
  margin-left: -3px;
}

.timeline-content {
  text-align: left;
}

.x-axis {
  position: absolute;
  left: 10px;
  right: 0;
  bottom: 0;
  height: 4px;
  background-color: #000;
}
</style>


<script>
// @ is an alias to /src
import { Icon } from "@iconify/vue";
import axios from 'axios';
import VueLazyload from 'vue-lazyload';
import Chart from 'chart.js/auto';
import 'chartjs-adapter-date-fns';
import zoomPlugin from 'chartjs-plugin-zoom'; // Import the zoom plugin
import { Zoom } from 'chartjs-plugin-zoom';

Chart.register(zoomPlugin); // Register the zoom plugin

export default {
  directives: {
    lazy: VueLazyload
  },
  name: "Dashboard",
  data() {
    return {
      // for more guide apexchart.js
      // https://apexcharts.com/docs/chart-types/line-chart/
      // chart data area

      myChart: null,
      currentPage: 1, // Currently viewed page
      itemsPerPage: 5, // Adjust this to show more or fewer items per page

      activities_chart_timeline: [],
      activities_chart: [],
      activities: [], // fetched from API

      selectedCamera: "All Cameras",
      selectedHouse: "All Houses",
      selectedStartDate: "All Start Dates",
      selectedEndDate: "All End Dates",

      activityColors: {
        'bend down to pick up things': 'bg-blue-200',
        'sit': 'bg-green-200',
        'sit and look at the wrist watch': 'bg-yellow-200',
        'sit and clap hands': 'bg-orange-200',
        'sit and talk on the phone': 'bg-pink-200',
        'sit and point': 'bg-red-200',
        'sit and wave': 'bg-indigo-200',
        'stand': 'bg-purple-200',
        'stand and look at the wrist watch': 'bg-teal-200',
        'stand and clap hands': 'bg-cyan-200',
        'stand and talk on the phone': 'bg-lime-200',
        'stand and point': 'bg-emerald-200',
        'stand and wave': 'bg-amber-200',
        'walk': 'bg-fuchsia-200',
        'fall down': 'bg-violet-200',
        'fall': 'bg-red-500',
        'unstable movement': 'bg-rose-200'
      }

    };
    // end chart data line
  },
  computed: {
    filteredActivities() {
      return this.activities.filter(activity => {
        return (this.selectedCamera === "All Cameras" || activity.camera_id == this.selectedCamera) &&
          (this.selectedHouse === "All Houses" || activity.house_id == this.selectedHouse) &&
          (this.selectedStartDate === "All Start Dates" || new Date(activity.timestamp) >= new Date(this.selectedStartDate)) &&
          (this.selectedEndDate === "All End Dates" || new Date(activity.timestamp) <= new Date(this.selectedEndDate));
      });
    },
    distinctCameras() {
      return [...new Set(this.activities.map(activity => activity.camera_id))];
    },
    distinctHouses() {
      return [...new Set(this.activities.map(activity => activity.house_id))];
    },
    distinctStartDates() {
      return [...new Set(this.activities.map(activity => activity.timestamp.split('T')[0]))];
    },
    distinctEndDates() {
      return this.distinctStartDates; // Since you want distinct dates, we can reuse the distinctStartDates
    },
    // Calculating the total number of pages
    totalPages() {
      return Math.ceil(this.activities.length / this.itemsPerPage);
    },
    chartTitle() {
      const firstDataPoint = new Date(2023, 6, 8, 0);  // You might want to get this dynamically from your data
      return firstDataPoint.toLocaleDateString('default', { month: 'long', day: '2-digit', year: 'numeric' });
    },
    refreshPagination() {
      this.currentPage = 1;
      // Update other pagination variables if needed
    },
    // Getting the set of items to display on the current page
    currentActivities() {
        // Get the activities for the current page based on the filtered activities
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.filteredActivities.slice(start, end);
    },
    resetZoom() {
      if (this.myChart) {
        this.myChart.resetZoom(); // Call resetZoom method on chart instance
      }
    },
  },
  watch: {
    // Watch for changes in the selected filters
    selectedCamera: 'refreshPagination',
    selectedHouse: 'refreshPagination',
    selectedStartDate: 'refreshPagination',
    selectedEndDate: 'refreshPagination',
  },
  methods: {
    prepareChartData() {

      const uniqueActivityTypes = [...new Set(this.activities_chart_timeline.map(activity => activity.activity_type))];

      // Sit, Fall, Walk, Stand
      const colors = ['rgba(174,255,99,0.6)', 'rgba(2,71,255,0.6)', 'rgba(255, 206, 86, 0.6)', 'rgba(239,0,0,0.97)'];

      const datasets = {};
      uniqueActivityTypes.forEach((type, index) => {
        datasets[type] = {
          data: [],
          label: type.charAt(0).toUpperCase() + type.slice(1),
          backgroundColor: colors[index % colors.length],
          pointRadius: 6 // Increase this value to make the dots larger
        };
      });

      this.activities_chart_timeline.forEach(activity => {
        const dataset = datasets[activity.activity_type];
        if (dataset) {
          dataset.data.push({
            x: new Date(activity.timestamp),
            y: uniqueActivityTypes.indexOf(activity.activity_type)
          });
        }
      });

      return {
        datasets: Object.values(datasets),
        labels: uniqueActivityTypes.map(type => type.charAt(0).toUpperCase() + type.slice(1))
      };

    },
    mapActivityToValue(activityType) {
      // This function maps each activity type to a numerical value for the y-axis.
      // You can customize this mapping as needed.
      const values = { 'walk': 1, 'fall': 2, 'sit': 3, 'stand': 4 };
      return values[activityType] || 0;
    },
    createChart() {

      const { datasets, labels } = this.prepareChartData();

      const canvas = document.getElementById('myChart');
      const ctx = canvas.getContext('2d');

      var verticalLinePlugin = {
        id: 'verticalLinePlugin',
        beforeDatasetsDraw: function (chart) {
          var xScale = chart.scales['x'];
          var yScale = chart.scales['y'];

          // Get the maximum x-value across all datasets
          var maxXValue = Math.max(...chart.data.datasets.flatMap(dataset => dataset.data.map(point => point.x)));

          // Get the x position for the maximum x-value
          var xPos = xScale.getPixelForValue(new Date(maxXValue));

          // Draw the line
          chart.ctx.beginPath();
          chart.ctx.moveTo(xPos, yScale.bottom);
          chart.ctx.strokeStyle = '#ff0000';
          chart.ctx.lineTo(xPos, yScale.top);
          chart.ctx.stroke();
        }
      };

      // คำนวณแสตมป์เวลาสุดท้าย
      var lastTimestamp = new Date(Math.max(...datasets.flatMap(dataset => dataset.data.map(point => point.x))));

      // ตั้งค่าแสตมป์เวลาสุดท้ายให้อยู่กึ่งกลางของช่วงเวลาที่แสดง
      var minTimeToShow = new Date(lastTimestamp);
      minTimeToShow.setHours(minTimeToShow.getHours() - 12); // 3 ชั่วโมงก่อนแสตมป์เวลาสุดท้าย
      var maxTimeToShow = new Date(lastTimestamp);
      maxTimeToShow.setHours(maxTimeToShow.getHours() + 12); // 3 ชั่วโมงหลังแสตมป์เวลาสุดท้าย


      this.myChart = new Chart(ctx, {
        type: 'scatter',
        data: {
          datasets: datasets
        },
        options: {
          plugins: {
            zoom: {
              pan: {
                enabled: true,
                mode: 'x', // Panning directions - either 'x', 'y', or 'xy'
              },
              zoom: {
                wheel: {
                  enabled: true,
                },
                pinch: {
                  enabled: false,
                },
                mode: 'x', // Zooming directions - either 'x', 'y', or 'xy'
              }
            }
          },
          scales: {
            x: {
              type: 'time',
              position: 'bottom',
              min: minTimeToShow, // ตั้งค่านี้
              max: maxTimeToShow  // และตั้งค่านี้
            },
            y: {
              type: 'linear',
              ticks: {
                callback: function (value, index, values) {
                  return labels[value] || '';
                }
              }
            }
          },
        },
        plugins: [zoomPlugin, verticalLinePlugin],
      });
    },
    async fetchActivitiesTimeline() {
      try {
        const response = await axios.get("http://localhost:8000/get-activities");
        this.activities_chart_timeline = response.data.data;
        this.createChart();
      } catch (error) {
        console.error("Error details fetchActivitiesTimeline:", error.response || error.request);
      }
    },
    // This in the main fetch activity
    async fetchActivities() {
      try {
        const response = await axios.get("http://localhost:8000/get-activities");
        this.activities = response.data.data; // Note the additional .data to access the 'data' key in the response
      } catch (error) {
        console.error("Error details :", error.response || error.request);
      }
    },
    async fetchActivitiesChart() {
          // Fetch the data
          // const response = await axios.get("http://localhost:8000/get-activities");
          // const activities_chart = response.data.data;
          // console.log("fetchActivitiesChart",activities_chart);
    },
    getActivityColor(activity) {
      return this.activityColors[activity] || 'bg-gray-200';  // Return 'bg-gray-200' as default if no match is found
    },
    // Navigate to the next page
    nextPage() {
      if (this.currentPage < this.totalPages) this.currentPage += 1;
    },
    // Navigate to the previous page
    previousPage() {
      if (this.currentPage > 1) this.currentPage -= 1;
    },
    // Jump to a specific page
    goToPage(page) {
      this.currentPage = page;
    },
  },
  created() {
    this.fetchActivities();
  },
  components: {
    Icon,
  },
  mounted() {
    this.fetchActivitiesTimeline();
    this.fetchActivities();
    this.fetchActivitiesChart();
  }
};

</script>
