<template>
      <div class="dashboard p-4">
        <!-- end nav -->
        <!-- grid wrapper card -->
    
        <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">
          <h2 class="font-bold text-lg text-gray-800 dark:text-gray-200">
            Activity Infomation
          </h2>
          <p class="text-gray-400 font-lexend font-normal">
            This is a list of latest activities detection
          </p>
          <div class="wrapping-table mt-10">
            <div class="wrapper-button p-5 flex justify-between mt-3">
              <!-- <select name="" id=""
                class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
                <option value="">Camera 1</option>
              </select>
    
              <select name="" id=""
                class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
                <option value="">House ID</option>
              </select>
    
              <select name="" id=""
                class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
                <option value="">01/01/2023</option>
              </select>
    
              <select name="" id=""
                class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
                <option value="">30/03/2023</option>
              </select>
    
              <select name="" id=""
                class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
                <option value="">Months</option>
              </select> -->
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
    
    
    
              <!-- <select v-model="selectedMonth">
                  <option value="">Select Month</option>
              </select> -->
    
    
              <!-- <select v-model="selectedMonth" class="dark:bg-gray-800 dark:hover:bg-gray-700 dark:border-gray-700 border max-w-lg px-4 py-3 block rounded-md text-gray-500 dark:text-gray-400">
                  <option value="">Months</option>
              </select> -->
            </div>
          </div>
    
                  <div class="wrapping-table mt-10">
                    
                  </div>
                  
            </div>
      </div>
    </template>
    
    
    <script>
    // @ is an alias to /src
    import { Icon } from "@iconify/vue";
    import axios from 'axios';
    import VueLazyload from 'vue-lazyload';
    
    
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
      },
      created() {
        this.fetchActivities();
      },
      mounted() {
        this.fetchActivities();
      }
    };
    </script>
    