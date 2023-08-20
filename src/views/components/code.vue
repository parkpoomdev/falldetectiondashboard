<template>
  <div>
    <canvas id="activityChart"></canvas>
  </div>
</template>

<script>
import { dbRef, onValue } from 'firebase/database';
import { db } from '@/firebase'; // Import your Firebase configuration
import Chart from 'chart.js/auto';
import moment from 'moment';

export default {
  data() {
    return {
      activities: {},
      chart: null,
    };
  },
  mounted() {
    this.fetchActivities();
  },
  methods: {
    async fetchActivities() {
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
            dataPoints.push({
              x: moment(activity.timestamp, "YYYY-MM-DD hh:mm A").toDate(),
              y: activity.activity_type
            });

            if (uniqueActivityTypes.indexOf(activity.activity_type) === -1) {
              uniqueActivityTypes.push(activity.activity_type);
            }

            colors.push(activity.activity_type === "fall" ? "red" : "blue");
          }

          const lastTimestamp = dataPoints.reduce((max, point) => Math.max(max, point.x), 0);
          const halfDay = 12 * 60 * 60 * 1000;
          const minTime = new Date(lastTimestamp - halfDay);
          const maxTime = new Date(lastTimestamp + halfDay);

          // If the chart already exists, update its data
          if (this.chart) {
            this.chart.data.datasets[0].data = dataPoints;
            this.chart.data.datasets[0].backgroundColor = colors;
            this.chart.options.scales.x.min = minTime;
            this.chart.options.scales.x.max = maxTime;
            this.chart.options.scales.y.labels = uniqueActivityTypes;
            this.chart.update();
          } else {
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
                }
              },
              plugins: [{
                beforeDatasetsDraw: (chart, options, el) => {
                  const ctx = chart.ctx;
                  const xAxis = chart.scales['x'];
                  const yAxis = chart.scales['y'];
                  const centerPosition = (xAxis.left + xAxis.right) / 2;

                  ctx.save();
                  ctx.strokeStyle = 'rgba(239,0,0,0.97)';
                  ctx.lineWidth = 2;
                  ctx.beginPath();
                  ctx.moveTo(centerPosition, yAxis.top);
                  ctx.lineTo(centerPosition, yAxis.bottom);
                  ctx.stroke();
                  ctx.restore();
                }
              }]
            });
          }
        } else {
          console.log("No activities found!");
        }
      });
    },
  },
};
</script>
