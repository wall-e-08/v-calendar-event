<template>
  <div id="main-calendar">
    <div class="container mt-5">
      <div class="row">
        <div class="col-md-8 offset-2">
          <div class="text-center">{{ month_str +", " + current_year }}</div>
          <div class="calendar-container">
            <div class="display-grid grid-col-7 day-container">
              <div v-bind:key="key" v-for="(wk, key) in week_day">{{ wk }}</div>
            </div>
            <div class="display-grid grid-col-7">
              <div v-bind:key="'m-'+key" v-for="(d, key) in month_starts_at"></div>
              <div v-bind:key="key" v-for="(d, key) in month_total_days">{{ d }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "SimpleCalendar",
  delimiters: ["[[", "]]"],
  props: {
    week_day: {
      type: Array,
      default: () => [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ]
    },
    months: {
      type: Array,
      default: () => [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ]
    },
  },
  data () {
    return {
      period_count: 8,
      break_after: 4,
      month_starts_at: 0, //Sunday
      current_date: new Date(),
      current_month: 0, //January
      current_year: 1900,
    };
  },
  computed: {
    month_str: function () {
      return this.months[this.current_month]
    },
    month_total_days: function () {
      let _t = this;
      let d = new Date(_t.current_year, _t.current_month + 1, 0);
      return d.getDate();
    }
  },
  created: function () {
    let _t = this;
    _t.current_month = _t.current_date.getMonth();
    _t.current_year = _t.current_date.getFullYear();
    let d = new Date(`${_t.current_year}-${_t.current_month+1}-01`); //vue.warn
    _t.month_starts_at = d.getDay();
  }
};
</script>


<style lang="scss" scoped>


</style>
