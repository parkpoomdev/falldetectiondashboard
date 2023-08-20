<template>

  <div class="dashboard p-4">
      <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">
        <h2 class="font-bold text-lg text-gray-800 dark:text-gray-200">
          Activity Infomation
        </h2>
        <p class="text-gray-400 font-lexend font-normal">
          This is a list of latest activities detection
        </p>
        <div style="position: relative; width: 600px; height: 400px;">
          <canvas id="activityChart" width="600" height="400"></canvas>
          <div v-if="isLoading" id="loader" style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);">
            <div v-if="isLoading" class="spinner"></div>
          </div>
<!--          <div v-if="isLoading" id="loader">Loading...</div>-->
        </div>
    <!--        <div>-->
    <!--          <canvas id="activityChart" height="150"></canvas>-->
    <!--        </div>-->

        <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">
          <div class="wrapping-table mt-10">
            <div class="wrapper-button p-5 flex justify-between mt-3">
              <div>
                <h3>Activities:</h3>
                <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400 lg:overflow-auto overflow-x-scroll">
                  <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                  <tr>
                    <th class="px-6 py-3">Activity Type</th>
                    <th class="px-6 py-3">Image</th>
                    <th class="px-6 py-3">Camera ID</th>
                    <th class="px-6 py-3">Family Member</th>
                    <th class="px-6 py-3">House ID</th>
                    <th class="px-6 py-3">Timestamp</th>
                  </tr>
                  </thead>
                  <tbody>
                  <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 odd:bg-white even:bg-gray-50"
                      v-for="activity in activities" :key="activity.timestamp">
                    <td class="px-6 py-4">{{ activity.activity_type }}</td>
                    <td class="px-6 py-4">
                      <img :src="'data:image/png;base64,' + activity.Image"  alt=""/>
                    </td>
                    <td class="px-6 py-4">{{ activity.camera_id }}</td>
                    <td class="px-6 py-4">{{ activity.family_member }}</td>
                    <td class="px-6 py-4">{{ activity.house_id }}</td>
                    <td class="px-6 py-4">{{ activity.timestamp }}</td>
                  </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      </div>

  </div>

</template>

<!--<notifications width="100px" />-->

<style>
.alert {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 20px;
  background-color: white;
  border: 1px solid #ccc;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}
.spinner {
  border: 8px solid #f3f3f3; /* Light grey */
  border-top: 8px solid #3498db; /* Blue */
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
.notification {
  padding: 10px;
  background-color: #f4f4f4;
  border: 1px solid #ddd;
  margin-top: 10px;
}
.notification-popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 20px;
  background-color: #f4f4f4;
  border: 1px solid #ddd;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}
</style>

<script>

  // @ is an alias to /src
  import { initializeApp } from "firebase/app";
  import { getDatabase, ref as dbRef, push, set, get, onValue} from "firebase/database";
  import { Chart } from 'chart.js';
  import moment from 'moment';
  import { useToast } from "vue-toastification";
  import Dashboard from "@/views/Dashboard.vue";
  // import Snotify from 'vue-snotify';
  // import Snotify from 'vue-snotify';

  const firebaseConfig = {
    apiKey: "AIzaSyDbfcijUitgAt8-dVKbtD4mDXU8SNUFDJQ",
    authDomain: "telehealth-project-df1b0.firebaseapp.com",
    databaseURL: "https://telehealth-project-df1b0-default-rtdb.asia-southeast1.firebasedatabase.app",
    projectId: "telehealth-project-df1b0",
    storageBucket: "telehealth-project-df1b0.appspot.com",
    messagingSenderId: "634800475333",
    appId: "1:634800475333:web:30d8f4edbc386c98ccb4ac",
    measurementId: "G-N353X7F0D9"
  };

  initializeApp(firebaseConfig);
  const db = getDatabase();

  export default {
    setup(){
      const toast = useToast();
      return { toast }
    },
    data() {
      return {
        lastAlertedTimestamp: 0,
        activities: [],
        chart: null, // Define chart here
        isLoading: false,
        alertMessage: '',
        isFirstRender: true,
        initialLoad: true,
        dialog: false,
        showAlert: false
      }
    },
    created() {
      this.showAlert = true;
    },
    mounted() {
      this.fetchActivities();
      this.showAlert = true;
    }, methods: {
      closeAlert() {
        this.showAlert = false;
      },
      triggerToast() {
        this.toast("Hi from LogRocket", {
          position: "top-right",
          timeout: 5000,
          closeOnClick: true,
          pauseOnFocusLoss: true,
          pauseOnHover: true,
          draggable: true,
          draggablePercent: 0.6,
          showCloseButtonOnHover: false,
          hideProgressBar: true,
          closeButton: "button",
          icon: "fas fa-rocket",
          rtl: false
        });
      },
      startLoading() {
        this.isLoading = true;
      },
      stopLoading() {
        this.isLoading = false;
      },
      addActivities() {
        const activitiesData = {
          "activities_lists": [
            {
              "Image": "RandomEncodedBase64String-1",
              "activity_type": "walk",
              "camera_id": "1",
              "family_member": "Anong Sasithorn",
              "house_id": "1",
              "timestamp": "2023-02-22 12:53 PM"
            }
          ]
        };

        // Reference to the root of the database
        const rootRef = dbRef(db);
        // Set the 'activities_lists' child with the activitiesData object
        set(rootRef, activitiesData);
      },
      checkLatestActivity() {

        let latestTimestamp = 0;
        let latestActivity = null;

        for (let key in this.activities) {
          const activity = this.activities[key];
          const activityTimestamp = moment(activity.timestamp, "YYYY-MM-DD hh:mm A").valueOf();

          if (activityTimestamp > latestTimestamp) {
            latestTimestamp = activityTimestamp;
            latestActivity = activity;
          }

        }

        if (!this.initialLoad && latestActivity.activity_type === "fall" && latestTimestamp > this.lastAlertedTimestamp) {
          // Update the last alerted timestamp
          this.lastAlertedTimestamp = latestTimestamp;

          // Do something if the latest activity is detected as a fall
          this.$notify({
            type: "error",
            text: "Fall detected in the latest activity!"
          });

          console.log("Fall detected in the latest activity!");
          // Add your desired action here
        }
      },
      async fetchActivities() {

        this.$notify({ type: "success", text: "The operation completed" });

        // this.$snotify('foo', 'bar');

        // this.$snotify.async('Called with promise', 'Error async', () => new Promise((resolve, reject) => {
        //   setTimeout(() => reject({
        //     title: 'Error!!!',
        //     body: 'We got an example error!',
        //     config: {
        //       closeOnClick: true
        //     }
        //   }), 2000);
        // }));

        // ลง  $notify แต่ Redner Html ไม่ได้
        // Notify s ติดตั้งไม่ได้ คืออะไร ทำไง ดี

        this.startLoading();

        let latestTimestamp = 0;
        let latestActivity = null;

        // Do something if the latest activity is detected as a fall
        // this.$notify({
        //   type: "error",
        //   text: 'This is some <b>content</b>',
        //   // (optional)
        //   // Title (will be wrapped in div.notification-title)
        //   title: 'This is the <em>title</em>',
        // });
        // Reference to the 'activities_lists' child of the database
        const activitiesRef = dbRef(db, 'activities_lists');

        // Set up a real-time listener to the referenced child
        onValue(activitiesRef, (snapshot) => {
          if (snapshot.exists()) {
            // If the data exists, set the activities property with the fetched data
            this.activities = snapshot.val();

            // Process the data
            const dataPoints = [];
            const uniqueActivityTypes = [];
            const colors = [];

            for (let key in this.activities) {

              const activity = this.activities[key];
              const activityTimestamp = moment(activity.timestamp, "YYYY-MM-DD hh:mm A").valueOf();

              dataPoints.push({
                x: moment(activity.timestamp, "YYYY-MM-DD hh:mm A").toDate(),
                y: activity.activity_type
              });

              if (uniqueActivityTypes.indexOf(activity.activity_type) === -1) {
                uniqueActivityTypes.push(activity.activity_type);
              }

              if(activity.activity_type === "fall"){

              }

              colors.push(activity.activity_type === "fall" ? "red" : "blue");

              // if (activityTimestamp > ) {
              //   latestTimestamp = activityTimestamp;
              //   latestActivity = activity;
              // }

            }

            this.checkLatestActivity();
            this.initialLoad = false;

            const lastTimestamp = dataPoints.reduce((max, point) => Math.max(max, point.x), 0);
            const oneHour = 60 * 60 * 1000;
            const minTime = new Date(lastTimestamp - oneHour / 2);
            const maxTime = new Date(lastTimestamp + oneHour / 2);

            // If the chart already exists, update its data
            if (this.chart) {
              this.chart.destroy();
            }
            //
            // if (latestActivity && latestActivity.activity_type === "fall") {
            //   // Do something if the latest activity is detected as a fall
            //   // alert("Fall detected in the latest activity!")
            //   this.$notify({
            //     type: "error",
            //     text: "A fall has been detected!"
            //   });
            //   console.log("Fall detected in the latest activity!");
            //   // Add your desired action here
            // }

            // If the chart doesn't exist yet, create it
            const ctx = document.getElementById('activityChart').getContext('2d');
            this.chart = new Chart(ctx, {
              type: 'scatter',
              data: {
                datasets: [{
                  label: 'Activities',
                  data: dataPoints,
                  pointRadius: 5,
                  backgroundColor: colors,
                }]
              },
              options: {
                spanGaps: true, // enable for all datasets
                animation: false,
                scales: {
                  x: {
                    type: 'time',
                    min: minTime,
                    max: maxTime,
                  },
                  y: {
                    type: 'category',
                    labels: uniqueActivityTypes,
                  }
                },plugins: {
                  zoom: { // Zoom and pan configuration
                    pan: {
                      enabled: true,
                      mode: 'x',
                    },
                    zoom: {
                      wheel: {
                        enabled: true,
                        speed: 0.1,
                      },
                      pinch: {
                        enabled: true,
                      },
                      mode: 'x',
                      max: 10, // Adjust this value to set the maximum zoom level
                      min: 0.5  // Adjust this value to set the minimum zoom level
                    },
                  },
                  // Other plugins can be added here
                },
              },
              plugins: [{
                beforeDatasetsDraw: (chart, options, el) => {
                  const ctx = chart.ctx;
                  const xAxis = chart.scales['x'];
                  const yAxis = chart.scales['y'];

                  // Determine the timestamp for the vertical line
                  // You can replace this with the specific timestamp you want
                  const timestampForLine = new Date(lastTimestamp); // Example: the last timestamp in the data

                  // Get the x-coordinate for the vertical line based on the timestamp
                  const xPosition = xAxis.getPixelForValue(timestampForLine);

                  ctx.save();
                  ctx.strokeStyle = 'rgba(239,0,0,0.97)';
                  ctx.lineWidth = 2;
                  ctx.beginPath();
                  ctx.moveTo(xPosition, yAxis.top);
                  ctx.lineTo(xPosition, yAxis.bottom);
                  ctx.stroke();
                  ctx.restore();
                }
              }]
            });
            this.stopLoading();
          } else {
            console.log("No activities found!");
          }
        });
      },
    }
  }
</script>

