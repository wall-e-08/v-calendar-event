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
          <div class="row">
            <div class="col-12">
              <div class="calendar-container table-warning mt-2">
                <table class="table table-fixed">
                  <tr>
                    <th class="bg-warning p-0">
                      <table class="table table-borderless table-fixed mb-0">
                        <tr>
                          <td class="text-center py-1" v-bind:key="'w-'+key" v-for="(wk, key) in week_day">{{ wk.substring(0,3) }}</td>
                        </tr>
                      </table>
                    </th>
                  </tr>
                  <tr>
                    <td class="p-0">
                      <div class="month-dates">
                        <table class="table mb-0 table-fixed">
                          <tr v-bind:key="key" v-for="(wk_arr, key) in all_month_days_arr">
                              <th :class="'pb-5 pl-2 pt-1 border border-light date-value ' + (d.type == 'current' ? '' : 'text-secondary')"
                                  v-bind:key="key" v-for="(d, key) in wk_arr" :data-type="d.type">{{ d.val }}</th>
                          </tr>
                        </table>
                      </div>
                    </td>
                  </tr>
                </table>
              </div>
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
    current_month_total_days: function () {
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
      let last_day_of_current_month = new Date(_t.current_year, _t.current_month, _t.current_month_total_days);
      let month_weeks = Math.ceil((_t.current_month_total_days + _t.month_starts_at)/7);

      if (month_weeks === 6)
        return 7 - last_day_of_current_month.getDay() - 1;
      else if (month_weeks === 5)
        return 14 - last_day_of_current_month.getDay() - 1; //extended 7 days + remaining weekdays (negative is for getDay index)
      else if (month_weeks === 4)
        return 14;  // plain 14, if 4 weeks
    },
    prev_month_end_days: function () {
      let _t = this;
      let prev_current_month_total_days = new Date(_t.current_year, _t.current_month, 0).getDate();
      let prev_month_last_days = [];

      for (let i = prev_current_month_total_days - _t.month_starts_at; i < prev_current_month_total_days; i++) {
        prev_month_last_days.push(i+1); // index starts from 0
      }
      return prev_month_last_days;
    },
    all_month_days_arr: function () {
      // prev_month_end_days
      // current_month_total_days
      // next_month_start_days
      let _t = this;
      let arr = [];

      for(let i=0; i<6; i++) {
        let child_arr = []
        for(let j=0; j<7; j++) {
          if (i === 0) {  // for first week
            if (j < _t.prev_month_end_days.length)
              child_arr.push({
                type: "prev",
                val: _t.prev_month_end_days[j]
              });
            else {
              child_arr.push({
                type: "current",
                val: j - _t.prev_month_end_days.length + 1,
              });
            }
          } else if (i < 4)  {  // for week 2 to 4
            // this whole loop is only for current month
            child_arr.push({
              type: "current",
              val: arr[i-1][6].val + j+1,
            });
          } else { // for week 5 & 6
            let current_month_remaining_days = _t.current_month_total_days - arr[i-1][6].val;
            let month_end_week = (_t.current_month_total_days - arr[3][6].val) >= 7 ? 6 : 5; // current month ending week number (value: i+1)
            
            if (i === 4) {
              // two logics: month ends in 5th week & ends in 6th
              if (month_end_week == 5) {
                // current month ends in 5th week (2 types of date will be showing)
                if ((current_month_remaining_days - j) > 0) {
                  // current month still running
                  child_arr.push({
                    type: "current",
                    val: arr[i-1][6].val + j+1,
                  });
                  // console.log(`5TH current: ${arr[i-1][6].val + j+1}`);
                } else {
                  // proceed to next month
                  child_arr.push({
                    type: "next",
                    val: j+1 - current_month_remaining_days,
                  });
                  // console.log(`5th next: ${j+1 - current_month_remaining_days}`);
                }
              } else {
                // only current month will be shown in 5th week
                child_arr.push({
                  type: "current",
                  val: arr[i-1][6].val + j+1,
                });
                // console.log(`5th current: ${arr[i-1][6].val + j+1}`);
              }
            } else if (i === 5) {
              // two logics: month ends in 6th week & already ended
              if (month_end_week == 6) {
                // current month ends in 6th week
                // current_month_remaining_days = _t.current_month_total_days - arr[4][6].val;
                if ((current_month_remaining_days - j) > 0) {
                  // current month still running
                  child_arr.push({
                    type: "current",
                    val: arr[i-1][6].val + j+1,
                  });
                  // console.log(`6th current: ${arr[i-1][6].val + j+1}`);
                } else {
                  // proceed to next month
                  child_arr.push({
                    type: "next",
                    val: j+1 - current_month_remaining_days,
                  });
                  // console.log(`6th next: ${j+1 - current_month_remaining_days}`);
                }
              } else {
                // current month already ended in 5th week
                  child_arr.push({
                    type: "next",
                    val: arr[i-1][6].val + j+1,
                  });
                // console.log(`6th next: ${arr[i-1][6].val + j+1}`);
              }
            }
          }
        }
        arr.push(child_arr);
      }
      return arr;
    },
  },
  methods: {
    range: function(start, end) {
      return Array.apply(0, Array(end)).map((el, i) => i + start);
    },
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
