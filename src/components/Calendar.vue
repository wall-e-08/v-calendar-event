<template>
  <div id="main-calendar">
    <div class="container mt-5">
      <div class="row">
        <div class="col-md-8 offset-2">
          <div class="row text-center">
            <div class="col-md-4">
              <button class="btn btn-success" v-on:click="go_today">today</button>
            </div>
            <div class="col-md-4">
              <div class="month-nav">
                <button class="btn btn-primary btn-sm" v-on:click="prev_month"><i class="fas fa-caret-left"></i></button>
                <span class="d-inline-block" style=width:120px>{{ month_str }}</span>
                <button class="btn btn-primary btn-sm" v-on:click="next_month"><i class="fas fa-caret-right"></i></button>
              </div>
              <div class="year-nav mt-2">
                <button class="btn btn-primary btn-sm" v-on:click="prev_year"><i class="fas fa-caret-left"></i></button>
                <span class="d-inline-block" style=width:120px>{{ current_year }}</span>
                <button class="btn btn-primary btn-sm" v-on:click="next_year"><i class="fas fa-caret-right"></i></button>
              </div>
            </div>
            <div class="col-md-4">
              <!-- <button class="btn btn-success">zz</button> -->
            </div>
          </div>
          <div class="calendar-container mt-2">
            <div class="display-grid grid-col-7 day-container">
              <div v-bind:key="key" v-for="(wk, key) in week_day">{{ wk }}</div>
            </div>
            <div class="display-grid grid-col-7">
              <div class="text-secondary" v-bind:key="'m-'+key" v-for="(d, key) in prev_month_end_days">{{ d }}</div>
              <div v-bind:key="key" v-for="(d, key) in month_total_days">{{ d }}</div>
              <div class="text-secondary" v-bind:key="'nm'+key" v-for="(d, key) in next_month_start_days">{{ d }}</div>
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
      current_date: new Date(),
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
    },
    current_month: function () {
      return this.current_date.getMonth();
    },
    current_year: function () {
      return this.current_date.getFullYear();
    },
    month_starts_at: function () {
      let _t = this;
      let d = new Date(`${_t.current_year}-${_t.current_month+1}-01`); //vue.warn
      return d.getDay();
    },
    next_month_start_days: function () {
      let _t = this;
      let last_day_of_current_month = new Date(_t.current_year, _t.current_month, _t.month_total_days);
      let month_weeks = Math.ceil((_t.month_total_days + _t.month_starts_at)/7);

      if (month_weeks === 6)
        return 7 - last_day_of_current_month.getDay() - 1;
      else if (month_weeks === 5)
        return 14 - last_day_of_current_month.getDay() - 1; //extended 7 days + remaining weekdays (negative is for getDay index)
      else if (month_weeks === 4)
        return 14;  // plain 14, if 4 weeks
    },
    prev_month_end_days: function () {
      let _t = this;
      let prev_month_total_days = new Date(_t.current_year, _t.current_month, 0).getDate();
      let prev_month_last_days = [];

      for (let i = prev_month_total_days - _t.month_starts_at; i < prev_month_total_days; i++) {
        prev_month_last_days.push(i);
      }
      // console.log(`Starts: ${_t.month_starts_at}; prev: ${prev_month_last_days}`)
      return prev_month_last_days;
    }
  },
  methods: {
    prev_month: function () {
      let _t = this;
      _t.current_date = new Date(_t.current_year, _t.current_month - 1, 1);
    },
    next_month: function () {
      let _t = this;
      _t.current_date = new Date(_t.current_year, _t.current_month + 1, 1);
    },
    prev_year: function () {
      let _t = this;
      _t.current_date = new Date(_t.current_year - 1, _t.current_month, 1);
    },
    next_year: function () {
      let _t = this;
      _t.current_date = new Date(_t.current_year + 1, _t.current_month, 1);
    },
    go_today: function () {
      this.current_date = new Date();
    },
  },
};
</script>


<style lang="scss" scoped>


</style>
