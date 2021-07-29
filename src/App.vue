<template>
  <div>
    <Calender />
    <Gantt />
  </div>
</template>

<script>
import Calender from '@/components/Calender';
import Gantt from '@/components/Gantt';

export default {
  name: 'App',
  components: {
    Calender,
    Gantt,
  },
  updated() {
  },
  data() {
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
