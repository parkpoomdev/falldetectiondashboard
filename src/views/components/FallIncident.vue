<template>

  <div class="dashboard p-4">
    <div class="mt-2 bg-white dark:bg-gray-800 p-5 w-full rounded-md box-border border dark:border-gray-700">

      <h2 class="font-bold text-lg text-gray-800 dark:text-gray-200">
        Fall Incident Detection
      </h2>

      <p class="text-gray-400 font-lexend font-normal">
        This is a list of fall activities detection filter by family and filter fall detection <br/><br/>
      </p>
      <select v-model="selectedFamilyMember">
        <option value="All Families">All Families</option>
        <option v-for="member in familyMembers" :key="member" :value="member">
          {{ member }}
        </option>
      </select>

      <table v-if="filteredActivities.length">
        <thead>
        <tr>
          <th>Activity Name</th>
          <th>Activity Type</th>
          <!-- Add other columns as needed -->
        </tr>
        </thead>
        <tbody>
        <tr v-for="activity in filteredActivities" :key="activity.id"> <!-- Assuming each activity has an id -->
          <td>{{ activity.family_member }}</td>
          <td>{{ activity.activity_type }}</td>
          <!-- Add other columns based on your data structure -->
        </tr>
        </tbody>
      </table>

<!--      <div v-for="activity in filteredActivities" :key="activity.id">-->
<!--        &lt;!&ndash; Assuming each activity has an id &ndash;&gt;-->
<!--        {{ activity.activity_name }} &lt;!&ndash; Replace with your actual data structure fields &ndash;&gt;-->
<!--      </div>-->

    </div>
  </div>
</template>


<script>

  import {initializeApp} from 'firebase/app';
  import 'firebase/database';
  // import {dbRef, getDatabase, ref, onValue} from "firebase/database";
  import { getDatabase, ref as dbRef, push, set, get, onValue} from "firebase/database";


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
    data() {
      return {
        houseIds: [],// default value for houseIds
        activities: [], // default value for activities
        familyMembers: [], // default value for familyMembers array
        selectedFamilyMember: 'All Families',// default value
        filteredActivities: []
      };
    },
    setup(){

    },
    created() {
      //
    },
    watch: {
      selectedFamilyMember(newValue, oldValue) {
        if (newValue !== 'All Families') {
          this.fetchFilteredData(newValue);
        } else {
          // Handle case for "All Families", maybe fetch all data or do something else
        }
      }
    },
    mounted() {
      this.fetchData();
      this.fetchFilteredData()
    }, methods: {
      async fetchData() {
        // Reference to the 'activities_lists' child of the database
        const activitiesRef = dbRef(db, 'activities_lists');

        // Create a Set to store distinct family members
        let distinctFamilyMembers = new Set();

        // Fetch the data from Firebase
        onValue(activitiesRef, (snapshot) => {
          if (snapshot.exists()) {
            snapshot.forEach(data => {
              // Add family member to the Set
              distinctFamilyMembers.add(data.val().family_member);

            });

            // Convert the Set to an array and store it in the component's data property
            this.familyMembers = [...distinctFamilyMembers];

            console.log("Family Member",    this.familyMembers);

          } else {
            console.log("No activities found!");
          }
        });
      },
      async fetchFilteredData(familyName) {

        const activitiesRef = dbRef(db, 'activities_lists');
        let filteredData = [];

        onValue(activitiesRef, (snapshot) => {
          if (snapshot.exists()) {
            snapshot.forEach(data => {
              if (data.val().family_member === familyName) {
                filteredData.push(data.val());
              }
            });

            this.filteredActivities = filteredData; // Assuming you have a reactive data property for this

          } else {
            console.log("No activities found!");
          }
        });
      }
    }

  }
</script>

