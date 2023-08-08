<template>
  <div class="dashboard p-4">
    <!-- end nav -->
    <!-- grid wrapper card -->
    <div class="flex justify-center">
      <div class="text-center">
        <button class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-800 rounded-full disabled:opacity-50 flex items-center" @click="resetZoom">
          <Icon icon="tabler:zoom-reset" /> Reset Zoom
        </button>
      </div>
    </div>

    <div class="wrapper-card grid lg:grid-cols-4 grid-cols-1 md:grid-cols-2 gap-2 mt-5">
      <canvas id="myChart"></canvas>
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
          <select v-model="selectedCamera" class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
              <option value="All Cameras">All Cameras</option>
              <option v-for="camera in distinctCameras" :key="camera" :value="camera">
                  Camera {{ camera }}
              </option>
          </select>

          <!-- House Select Box -->
          <select v-model="selectedHouse" class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
              <option value="All Houses">All Houses</option>
              <option v-for="house in distinctHouses" :key="house" :value="house">
                  House ID {{ house }}
              </option>
          </select>

          <!-- Start Date Select Box -->
          <select v-model="selectedStartDate" class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
              <option value="All Start Dates">All Start Dates</option>
              <option v-for="date in distinctStartDates" :key="date" :value="date">
                  {{ date }}
              </option>
          </select>

          <!-- End Date Select Box -->
          <select v-model="selectedEndDate" class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
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
                  <img :src="'data:image/png;base64,' + activity.image" alt="Thumbnail" style="height: 100px;"/>
                </td>
              </tr>
            </tbody>
          </table>

                <!-- Pagination Controls -->
          <div class="flex justify-center mt-4 space-x-2">
            <!-- Button to navigate to the previous page -->
            <button 
              class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-800 rounded-full disabled:opacity-50" 
              @click="previousPage" 
              :disabled="currentPage === 1">
              Previous
            </button>

            <!-- Buttons to jump to specific pages -->
            <span v-for="page in totalPages" :key="page">
              <button 
                class="px-3 py-2 bg-blue-200 hover:bg-blue-300 text-blue-800 rounded-full disabled:bg-blue-400 disabled:opacity-50" 
                @click="goToPage(page)" 
                :disabled="currentPage === page">
                {{ page }}
              </button>
            </span>

            <!-- Button to navigate to the next page -->
            <button 
              class="px-4 py-2 bg-gray-200 hover:bg-gray-300 text-gray-800 rounded-full disabled:opacity-50" 
              @click="nextPage" 
              :disabled="currentPage === totalPages">
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

<script>
// @ is an alias to /src
import { Icon } from "@iconify/vue";
import axios from 'axios';
import VueLazyload from 'vue-lazyload';
import Chart from 'chart.js/auto';
import * as d3 from 'd3';
import 'chartjs-adapter-date-fns';
import zoomPlugin from 'chartjs-plugin-zoom'; // Import the zoom plugin

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
      chart: null,
      currentPage: 1, // Currently viewed page
      itemsPerPage: 5, // Adjust this to show more or fewer items per page

   
      optionsArea: {
        xaxis: {
          categories: [2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022],
        },
        fontFamily: "Segoe UI, sans-serif",
        stroke: {
          curve: "straight",
        },
        chart: {
          toolbar: {
            show: true,
          },
          zoom: {
            enabled: false,
          },
          sparkline: {
            enabled: true,
          },
        },
        markers: {
          size: 0,
        },
        yaxis: {
          show: false,
        },
        fill: {
          type: "gradient",
          gradient: {
            shadeIntensity: 0,
            opacityFrom: 0.2,
            opacityTo: 0.5,
            stops: [0, 90, 100],
          },
        },
      },
      chart: {
        fontFamily: "lexend, sans-serif",
      },
      seriesArea: [
        {
          name: "Revenue",
          data: [30, 40, 45, 50, 49, 60, 70, 91],
        },
        {
          name: "Revenue (Previous period)",
          data: [20, 34, 45, 55, 79, 87, 90, 98],
        },
      ],
      optionsBar: {
        chart: {
          toolbar: {
            show: false,
          },
          zoom: {
            enabled: false,
          },
          sparkline: {
            enabled: true,
          },
        },
        legend: {
          show: false,
        },
        xaxis: {
          show: false,
        },
        yaxis: {
          show: false,
        },
        colors: ["#4f46e5", "#DC2626"],
        dataLabels: {
          enabled: false,
        },
        stroke: {
          curve: "straight",
        },
      },
      seriesBar: [
        {
          name: "Product 1",
          data: [30, 40, 45, 50, 49, 60, 70, 91],
        },
        {
          name: "Product 2",
          data: [20, 34, 45, 55, 79, 87, 90, 98],
        },
      ],
      optionsVisitor: {
        chart: {
          toolbar: {
            show: false,
          },
          zoom: {
            enabled: false,
          },
          sparkline: {
            enabled: true,
          },
        },
        legend: {
          show: false,
        },
        xaxis: {
          show: false,
        },
        yaxis: {
          show: false,
        },
        colors: ["#4f46e5"],
        dataLabels: {
          enabled: false,
        },
        fill: {
          type: "gradient",
          gradient: {
            shadeIntensity: 0,
            opacityFrom: 0,
            opacityTo: 0.3,
            stops: [0, 90, 100],
          },
        },
        stroke: {
          curve: "smooth",
        },
      },
      seriesVisitor: [
        {
          name: "Visitor ",
          data: [30, 40, 45, 50, 49, 60, 70, 91],
        },
      ],
      optionsDonut: {
        chart: {
          type: "donut",
        },
        legend: false,
        dataLabels: {
          enabled: false,
        },
        labels: ["admin", "SuperAdmin", "User", "Costumer"],
      },
      seriesDonut: [20, 15, 63, 83],
      tableTransaction: [
        {
          transaction: "Payment from Ethan Roger",
          datetime: "Apr 22, 2022",
          amount: "Rp.450.000",
          statusTransaction: "completed",
        },
        {
          transaction: "Payment from Taylor neal",
          datetime: "May 2, 2022",
          amount: "Rp.250.000",
          statusTransaction: "completed",
        },
        {
          transaction: "Payment from Tobi Ferreira",
          datetime: "May 5, 2022",
          amount: "Rp.150.000",
          statusTransaction: "progress",
        },
        {
          transaction: "Payment failed from #046577",
          datetime: "May 5, 2022",
          amount: "Rp.180.000",
          statusTransaction: "cancelled",
        },
      ],

      activities: [],
      monthNames: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
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

    // Getting the set of items to display on the current page
    currentActivities() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      return this.activities.slice(start, start + this.itemsPerPage);
    },
},
  methods: {
    async fetchActivities() {
      try {

        const response = await axios.get("http://localhost:8000/get-activities");
        this.activities = response.data.data; // Note the additional .data to access the 'data' key in the response

      } catch (error) {
        console.error("Error details:", error.response || error.request);
      }
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
  
    resetZoom() {
      if (this.chart) {
        this.chart.resetZoom(); // Call resetZoom method on chart instance
      }
    },
  },
  created() {
    this.fetchActivities();
  },
  components: {
		Icon,
	},
  mounted() {
    this.fetchActivities();

    var categories = ['sit', 'walk', 'fall', 'stand', 'fall down', 'bend down to pick up things', 'sit and wave', 'stand and wave', 'sit and clap hands', 'stand and clap hands', 'sit and talk on the phone', 'stand and talk on the phone', 'sit and point', 'stand and point', 'sit and look at the wrist watch', 'stand and look at the wrist watch', 'unstable movement'];

    // Define a color for each category
    var categoryColors = {
      'fall': 'red',
      'fall down': 'red',
      // Add more colors as needed for each category
    };


    // Generate or load the data points (this example generates random data)
    const allDataPoints = Array.from({ length: 1000 }, (_, i) => {
      const activity = categories[Math.floor(Math.random() * categories.length)];
      const isRed = ['fall', 'fall down'].includes(activity);
      return {
        x: new Date(2023, 7, 8 + Math.floor(i / 24), i % 24), // Example data across several days
        y: categories.indexOf(activity),
        backgroundColor: isRed ? 'red' : 'blue',
      };
    });


    // Filter data points for today
    const today = new Date();
    
    const dataPoints = Array.from({ length: 24 }, (_, i) => {
      const activity = categories[Math.floor(Math.random() * categories.length)];
      return {
        x: new Date(2023, 6, 8, i), // 8 June 2023
        y: categories.indexOf(activity),
        backgroundColor: ['fall', 'fall down'].includes(activity) ? 'red' : 'blue',
      };
    });
    
    var ctx = document.getElementById('myChart').getContext('2d');

    this.chart = new Chart(ctx, {
      type: 'scatter',
      data: {
        datasets: [
          {
            label: 'Activity',
            data: dataPoints,
            borderColor: dataPoints.map((point) => point.backgroundColor),
            backgroundColor: dataPoints.map((point) => point.backgroundColor),
            pointRadius: 5,
          },
        ],
      },
      options: {
        scales: {
          x: {
            type: 'time',
            time: {
              unit: 'hour',
            },
          },
          y: {
            type: 'linear',
            ticks: {
              callback: (value) => categories[value],
              stepSize: 1,
              min: 0,
              max: categories.length - 1,
            },
          },
        },
        plugins: {
          zoom: {
            pan: {
              enabled: true,
              mode: 'xy',
            },
            zoom: {
              wheel: {
                enabled: true,
              },
              pinch: {
                enabled: true,
              },
              mode: 'xy',
            },
          },
        },
      },
    });

  }
};


</script>
