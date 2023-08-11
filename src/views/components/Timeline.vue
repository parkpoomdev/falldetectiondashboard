<template>
      <div class="timeline-container">
        <div class="timeline">
          <div
            v-for="(event, index) in events"
            :key="index"
            class="event"
            :class="{ 'right': index % 2 === 1 }"
          >
            <div class="event-date">{{ formatDate(event.date) }}</div>
            <div class="event-description">{{ event.description }}</div>
          </div>
        </div>
      </div>
    </template>
    
    <script>
    export default {
      props: {
        events: Array,
      },
      methods: {
        formatDate(date) {
          return new Date(date).toLocaleDateString();
        },
      },
    };
    </script>
    
    <style>
    .timeline-container {
      max-width: 600px;
      margin: 0 auto;
    }
    
    .timeline {
      position: relative;
      padding: 0 0 20px 20px;
    }
    
    .timeline::before {
      content: '';
      position: absolute;
      left: 5px;
      top: 0;
      bottom: 0;
      width: 2px;
      background-color: #000;
    }
    
    .event {
      width: calc(100% - 20px);
      padding: 10px;
      margin: 20px 0;
      position: relative;
    }
    
    .event.right {
      margin-left: 20px;
    }
    
    .event-date {
      position: absolute;
      left: -60px;
      background: #fff;
      padding: 5px;
    }
    
    .event-description {
      padding: 10px;
      background-color: #f3f3f3;
      border-radius: 5px;
    }
    </style>
    