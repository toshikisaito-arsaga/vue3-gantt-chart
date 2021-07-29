<template>
  <div>
    <div>
      <DatePicker v-model="date" />
    </div>
    <div>
      <span>{{ date }}</span>
    </div>
  </div>
</template>

<script>
import { DatePicker } from 'v-calendar';

export default {
  name: 'Calender',
  components: {
    DatePicker,
  },
  updated() {
    console.log(this.date.getDate());
  },
  data() {
    return {
      date: new Date(),
    };
  },
}
</script>

<style>

</style>
