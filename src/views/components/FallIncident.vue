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
      <br/>

<!--      <button @click="jumpToFall">Jump to Fall</button>-->

    </div>

    <div class="timeline">
      <ul>
        <li v-for="activity in filteredActivities" :key="activity.id">
          <span>{{ activity.date }}</span>
          <div class="content element" :style="activity.activity_type === 'fall' ? {'background-color': '#fdedea'} : {}">
            <h3>{{ activity.family_member }}</h3>
              <p>
                {{ activity.activity_type }}
                <br/>
                {{ activity.timestamp }}
              </p>
          </div>

        </li>

        <!-- Display this when there are no filtered activities -->
        <li class="forselect" v-if="!filteredActivities.length">
          <span style="margin-bottom: 10px">Select Family Member to see the fall incident</span>
          <div class="content element">
            <h3>Family Member</h3>
            <p>
              Activity Type Information
            </p>
          </div>
        </li>

        <!-- ... other existing list items ... -->
      </ul>
    </div>

  </div>
</template>

<style>

@import url('https://fonts.googleapis.com/css2?family=Poppins&display=swap');

.content{
  background:#e5effa;
  border: 2px solid #7fa3cb;
  padding: 8px;
  border-radius: 9px;
}
.timeline{
  width:800px;
  /*background-color:#072736;*/
  color:#fff;
  padding:30px 20px;
  /*box-shadow:0px 0px 10px rgba(0,0,0,.5);*/
}
.timeline ul{
  list-style-type:none;
  border-left:2px solid #094a68;
  padding:10px 5px;
}
.timeline ul li{
  padding:4px 20px;
  position:relative;
  cursor:pointer;
  transition:.5s;
}
.timeline ul li span{
  display:inline-block;
  background-color:#1685b8;
  border-radius:25px;
  padding:2px 5px;
  font-size:15px;
  text-align:center;
}
.timeline ul li .content h3{
  color:#34ace0;
  font-size:17px;
  padding-top:5px;
}
.timeline ul li .content p{
  padding:5px 0px 15px 0px;
  font-size:18px;
  color: black;
}
.timeline ul li:before{
  position:absolute;
  content:'';
  width:10px;
  height:10px;
  background-color:#34ace0;
  border-radius:50%;
  left:-11px;
  top:28px;
  transition:.5s;
}
.timeline ul li:hover{
  /*background-color:#071f2a;*/
}
.timeline ul li:hover:before{
  /* background-color:#0F0;
  box-shadow:0px 0px 10px 2px #0F0; */
}
.element {
  background-color: #b1e9ff;
  transition: background-color 0.3s ease;
  animation-name: fadeIn, slideIn;
  animation-duration: 1s, 2s;
  animation-delay: 0s, 1s;
}
.element:hover {
  background-color: #77daff;
}
@media (max-width:300px){
  .timeline{
    width:100%;
    padding:30px 5px 30px 10px;
  }
  .timeline ul li .content h3{
    color:#34ace0;
    font-size:15px;
  }

}

</style>

<script>
  import {initializeApp} from 'firebase/app';
  import 'firebase/database';
  import { getDatabase, ref as dbRef, push, set, get, onValue} from "firebase/database";
  import { Timeline, TimelineItem, TimelineTitle } from 'vue-cute-timeline'
  import 'vue-cute-timeline/dist/index.css'

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

  const theme = 'red';

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
        this.fetchFilteredData(newValue);
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
              if (data.val().family_member === familyName || familyName === 'All Families') {
                filteredData.push(data.val());
              }
            });

            // Sort the filtered data based on timestamp in descending order
            filteredData.sort((a, b) => {
              return new Date(b.timestamp) - new Date(a.timestamp);
            });

            this.filteredActivities = filteredData;

          } else {
            console.log("No activities found!");
          }
        });
      }

    }

  }
</script>

