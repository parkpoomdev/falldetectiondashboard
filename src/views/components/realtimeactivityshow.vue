<template>

  <div class="dashboard p-4">
      <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">
        <h2 class="font-bold text-lg text-gray-800 dark:text-gray-200">
          See Realtime Activity For {{ familyMember }}
        </h2>

        <!-- Display the chart if familyMember is valid -->
        <div v-if="isValidFamilyMember">
          <!-- Your chart rendering logic here -->
        </div>

        <!-- Display a message if familyMember is not valid -->
        <div v-else>
          <p>Please provide a valid family member to view the chart.</p>
        </div>
<!--        <br/>-->
<!--        <div style='border: 1px solid #e5e7eb; border-radius: 10px; display: inline-block; padding: 10px;'>-->
<!--          <select v-model="selectedFamilyMember" class="form-select form-select-lg mb-3">-->
<!--            <option value="All Families">All Families</option>-->
<!--            <option v-for="member in familyMembers" :key="member" :value="member">-->
<!--              {{ member }}-->
<!--            </option>-->
<!--          </select>-->

<!--          <button @click="filterActivitiesByFamilyMember" class="btn btn-primary">Filter</button>-->

<!--        </div>-->
<!--        <br/><br/>-->

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
<!--          <div>-->
<!--            <canvas id="activityChart" height="150"></canvas>-->
<!--          </div>-->

<!--        <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">-->
<!--          <div class="wrapping-table mt-10">-->
<!--            <div class="wrapper-button p-5 flex justify-between mt-3">-->
<!--              <div>-->
<!--                <h3>Activities:</h3>-->
<!--                <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400 lg:overflow-auto overflow-x-scroll">-->
<!--                  <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">-->
<!--                  <tr>-->
<!--                    <th class="px-6 py-3">Family Member</th>-->
<!--                    <th class="px-6 py-3">See Realtime Activity</th>-->
<!--                  </tr>-->
<!--                  </thead>-->
<!--                    <tbody>-->
<!--                      <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 odd:bg-white even:bg-gray-50"-->
<!--                          v-for="activity in activities" :key="activity.timestamp">-->
<!--                        <td class="px-6 py-4">{{ activity.family_member }}</td>-->
<!--                        <td class="px-6 py-4">{{ activity.family_member }}</td>-->
<!--                      </tr>-->
<!--                    </tbody>-->
<!--                </table>-->
<!--              </div>-->
<!--            </div>-->
<!--          </div>-->
<!--        </div>-->
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

  import { initializeApp } from "firebase/app";
  import { getDatabase, ref as dbRef, push, set, get, onValue} from "firebase/database";
  import { Chart } from 'chart.js';
  import moment from 'moment';
  import { useToast } from "vue-toastification";
  import Dashboard from "@/views/Dashboard.vue";

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
        showAlert: false,
        activitiesSummary: [],
        chartInitialized: false,
        familyMember: null,
        isValidFamilyMember: false,
      }
    },
    created() {
      this.showAlert = true;
    },
    computed: {
      distinctFamilyMembers() {
        const activitiesArray = Array.isArray(this.activitiesSummary) ? this.activitiesSummary : Object.values(this.activitiesSummary);
        return [...new Set(activitiesArray.map(activity => activity.family_member))];
      },
      familyMember() {
        return this.$route.query.familyMember;
      }
    },
    mounted() {

      this.showAlert = true;
      this.fetchActivitiesSummary();

      // Get familyMember from the query string
      this.familyMember = this.$route.query.familyMember;

      // Validate familyMember
      if (this.familyMember && this.familyMember.trim() !== '') {
        this.isValidFamilyMember = true;
        this.fetchActivitiesSummary();
      } else {
        this.isValidFamilyMember = false;
      }

    },
    methods: {
      fetchActivitiesSummary() {
        // Check if the familyMember is valid before fetching data
        if (!this.isValidFamilyMember) {
          console.warn("Invalid family member. Data fetch aborted.");
          return;
        }

        const activitiesSummaryRef = dbRef(db, 'activities_summary');

        onValue(activitiesSummaryRef, (snapshot) => {
          if (snapshot.exists()) {
            const allActivities = snapshot.val();

            // Filter the activities based on the familyMember from the query string
            this.activitiesSummary = Object.values(allActivities).filter(activity => {
              return activity.family_member === this.familyMember;
            });

            const activitiesArray = Array.isArray(this.activitiesSummary) ? this.activitiesSummary : Object.values(this.activitiesSummary);

            // Convert the data into the required format for Chart.js
            const dataPoints = activitiesArray.map(activity => {
              return {
                x: moment(activity.timestamp, "YYYY-MM-DD hh:mm:ss A").toDate(),
                y: activity.activity_type
              };
            });

            // Detect Fall Color
            const dataColors = activitiesArray.map(activity => {
              return activity.activity_type === 'fall' ? 'red' : 'blue';
            });

            // Dynamically generate labels based on distinct activity types
            const uniqueActivityTypes = [...new Set(activitiesArray.map(activity => activity.activity_type))];

            // Check if the latest activity is a fall
            const latestActivity = activitiesArray[activitiesArray.length - 1];
            if (latestActivity && latestActivity.activity_type === 'fall') {
              this.$notify({
                type: "error",
                text: "Fall detected in the latest activity!"
              });
            }

            this.plotDataOnChart(dataPoints, uniqueActivityTypes, dataColors);

          } else {
            console.log("No data available");
          }
        }, (error) => {
          console.error("Error listening to data changes:", error);
        });
      }
      ,
      plotDataOnChart(dataPoints, uniqueActivityTypes, dataColors) {
        const ctx = document.getElementById('activityChart').getContext('2d');
        if (this.chart) {
          this.chart.destroy();
        }
        this.chart = new Chart(ctx, {
          type: 'scatter',
          data: {
            datasets: [{
              label: 'Activities',
              data: dataPoints,
              pointRadius: 5,
              backgroundColor: dataColors, // Using a fixed color for simplicity
            }]
          },
          options: {
            scales: {
              x: {
                type: 'time'
              },
              y: {
                type: 'category',
                labels: uniqueActivityTypes
              }
            }
          }
        });
      },
      closeAlert() {
        this.showAlert = false;
      },
      startLoading() {
        this.isLoading = true;
      },
      stopLoading() {
        this.isLoading = false;
      },
    }
  }
</script>

