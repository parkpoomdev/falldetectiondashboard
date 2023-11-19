<template>

  <div class="dashboard p-4">
      <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">
        <h2 class="font-bold text-lg text-gray-800 dark:text-gray-200">
          Activity Infomation
        </h2>

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
<!--        <div style="position: relative; width: 600px; height: 400px;">-->
<!--          <canvas id="activityChart" width="600" height="400"></canvas>-->
<!--          <div v-if="isLoading" id="loader" style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);">-->
<!--            <div v-if="isLoading" class="spinner"></div>-->
<!--          </div>-->
<!--&lt;!&ndash;          <div v-if="isLoading" id="loader">Loading...</div>&ndash;&gt;-->
<!--        </div>-->
    <!--        <div>-->
    <!--          <canvas id="activityChart" height="150"></canvas>-->
    <!--        </div>-->

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
        <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">
          <div class="wrapping-table mt-10">
            <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400 lg:overflow-auto overflow-x-scroll">
              <thead>
                <tr>
                  <th>Family Member</th>
                  <th>Action</th>
                </tr>
              </thead>

              <tbody>
                <tr v-for="member in distinctFamilyMembers" :key="member" class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 odd:bg-white even:bg-gray-50">
                  <td class="px-6 py-4">{{ member }}</td>
                  <td class="px-6 py-4">
                    <!-- Button that acts as a link to another page -->
                  <button @click="navigateToOtherPage(member)">See Realtime Activity</button>
                 </td>
                </tr>
              </tbody>
            </table>
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

  import { initializeApp } from "firebase/app";
  import { getDatabase, ref as dbRef, push, set, get, onValue} from "firebase/database";
  import { Chart } from 'chart.js';
  import moment from 'moment';
  import { useToast } from "vue-toastification";
  import Dashboard from "@/views/Dashboard.vue";

  // const firebaseConfig = {
  //   apiKey: "AIzaSyDbfcijUitgAt8-dVKbtD4mDXU8SNUFDJQ",
  //   authDomain: "telehealth-project-df1b0.firebaseapp.com",
  //   databaseURL: "https://telehealth-project-df1b0-default-rtdb.asia-southeast1.firebasedatabase.app",
  //   projectId: "telehealth-project-df1b0",
  //   storageBucket: "telehealth-project-df1b0.appspot.com",
  //   messagingSenderId: "634800475333",
  //   appId: "1:634800475333:web:30d8f4edbc386c98ccb4ac",
  //   measurementId: "G-N353X7F0D9"
  // };

  const firebaseConfig = {
    apiKey: "AIzaSyDILC8ph12BiCuLrr1af6truM6OYCzBNo0",
    authDomain: "fall-data-analytics-62e33.firebaseapp.com",
    databaseURL: "https://fall-data-analytics-62e33-default-rtdb.asia-southeast1.firebasedatabase.app",
    projectId: "fall-data-analytics-62e33",
    storageBucket: "fall-data-analytics-62e33.appspot.com",
    messagingSenderId: "305819022730",
    appId: "1:305819022730:web:f32fc3a878725f38bb4473",
    measurementId: "G-6RWX9S9BG5"
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
        chartInitialized: false
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

    },
    mounted() {
      this.showAlert = true;
      this.fetchActivitiesSummary();
    },
    methods: {
      navigateToOtherPage(member) {
        // Here, you can use Vue Router or another method to navigate to the desired page.
        // For example, using Vue Router:
        // this.$router.push({ name: 'OtherPage', params: { familyMember: member } });

        // If you're not using Vue Router, you can use a simple window.location.href:
        window.location.href = `realtimeactivityshow?familyMember=${member}`;
      },
      async fetchActivitiesSummary() {

        const activitiesSummaryRef = dbRef(db, 'activities_summary');

        try {
          const snapshot = await get(activitiesSummaryRef);
          if (snapshot.exists()) {
            this.activitiesSummary = snapshot.val();
            // console.log("Data as JSON:", JSON.stringify(this.activitiesSummary));
          } else {
            console.log("No data available");
          }
        } catch (error) {
          console.error("Error fetching data:", error);
        }

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

