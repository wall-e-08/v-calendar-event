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
                      <div class="position-relative month-dates" v-bind:key="key" v-for="(wk_arr, key) in all_month_days_arr">
                        <table class="table mb-0 table-fixed">
                          <tr>
                              <td :class="'p-0 date-value ' + (d.type == 'current' ? '' : 'text-secondary')"
                                  v-bind:key="key" v-for="(d, key) in wk_arr" :data-type="d.type" :data-date="d.date">{{ d.val }}</td>
                          </tr>
                        </table>
                        <table class="table mb-0 table-fixed position-absolute event-table">
                          <tr>
                            <td class="p-0 event-by-day" v-bind:key="'empty-'+key21" v-for="(events, key21) in events_by_week[key]"></td>
                          </tr>
                          <tr>
                              <td class="p-0 date-value event-by-day"
                                  v-bind:key="key2" v-for="(events, key2) in events_by_week[key]">
                                <template v-for="(event, key3) in events">
                                  <b-button variant="success" size="sm" v-b-modal="'modal-'+event.date.replace(/\//g,'-')+key3" :key="'btn-'+key3">{{ event.date }}</b-button>
                                  <b-modal :id="'modal-'+event.date.replace(/\//g,'-')+key3" :key="'modal-'+key3" :title="event.title">
                                    <p class="my-4">TITLE: {{ event.title }}</p>
                                    <p class="my-4">DATE: {{ event.date.replace(/\//g, '-') }}</p>
                                    <p class="my-4">START TIME: {{ event.start_time }}</p>
                                    <p class="my-4">END TIME: {{ event.end_time }}</p>
                                    <p class="my-4">DESCRIPTION: {{ event.description }}</p>
                                  </b-modal>
                                </template>
                              </td>
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
import json_data from '../data/test.json'

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
      let date_val;

      for(let i=0; i<6; i++) {
        let child_arr = []
        for(let j=0; j<7; j++) {
          if (i === 0) {  // for first week
            if (j < _t.prev_month_end_days.length){
              date_val = _t.prev_month_end_days[j];
              child_arr.push({
                type: "prev",
                val: date_val,
                date: _t.get_date_string(new Date(_t.current_year, _t.current_month-1, date_val)),
              });
            }
            else {
              date_val = j - _t.prev_month_end_days.length + 1;
              child_arr.push({
                type: "current",
                val: date_val,
                date: _t.get_date_string(new Date(_t.current_year, _t.current_month, date_val)),
              });
            }
          } else if (i < 4)  {  // for week 2 to 4
            // this whole loop is only for current month
            date_val = arr[i-1][6].val + j+1;
            child_arr.push({
              type: "current",
              val: date_val,
              date: _t.get_date_string(new Date(_t.current_year, _t.current_month, date_val)),
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
                  date_val = arr[i-1][6].val + j+1;
                  child_arr.push({
                    type: "current",
                    val: date_val,
                    date: _t.get_date_string(new Date(_t.current_year, _t.current_month, date_val)),
                  });
                  // console.log(`5TH current: ${arr[i-1][6].val + j+1}`);
                } else {
                  // proceed to next month
                  date_val = j+1 - current_month_remaining_days;
                  child_arr.push({
                    type: "next",
                    val: date_val,
                    date: _t.get_date_string(new Date(_t.current_year, _t.current_month+1, date_val)),
                  });
                  // console.log(`5th next: ${j+1 - current_month_remaining_days}`);
                }
              } else {
                // only current month will be shown in 5th week
                date_val = arr[i-1][6].val + j+1;
                child_arr.push({
                  type: "current",
                  val: date_val,
                  date: _t.get_date_string(new Date(_t.current_year, _t.current_month, date_val)),
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
                  date_val = arr[i-1][6].val + j+1;
                  child_arr.push({
                    type: "current",
                    val: date_val,
                    date: _t.get_date_string(new Date(_t.current_year, _t.current_month, date_val)),
                  });
                  // console.log(`6th current: ${arr[i-1][6].val + j+1}`);
                } else {
                  // proceed to next month
                  date_val = j+1 - current_month_remaining_days;
                  child_arr.push({
                    type: "next",
                    val: date_val,
                    date: _t.get_date_string(new Date(_t.current_year, _t.current_month+1, date_val)),
                  });
                  // console.log(`6th next: ${j+1 - current_month_remaining_days}`);
                }
              } else {
                // current month already ended in 5th week
                date_val = arr[i-1][6].val + j+1;
                child_arr.push({
                  type: "next",
                  val: date_val,
                  date: _t.get_date_string(new Date(_t.current_year, _t.current_month+1, date_val)),
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
    events_by_week: function () {
      let _t = this,
          events_arr = [], ev_arr_inside = [],
          now_date, date_split, data_date;

      for (let i = 0; i < _t.all_month_days_arr.length; i++) {
        ev_arr_inside = [];
        for (let j = 0; j < _t.all_month_days_arr[i].length; j++) {
          now_date = new Date(_t.all_month_days_arr[i][j].date);
          ev_arr_inside.push(
            json_data.filter(function(each_data){
              date_split = each_data.date.split('/').map((a) => parseInt(a));
              data_date = new Date(date_split[2], date_split[0]-1, date_split[1]); // format was m/d/Y
              return data_date.getTime() == now_date.getTime();
            })
          )
        }
        events_arr.push(ev_arr_inside);
      }
      return events_arr;
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
    get_date_string: function (date_obj) {
      return `${date_obj.getFullYear()}-${date_obj.getMonth()+1}-${date_obj.getDate()}`
    }
  },
};
</script>


<style lang="scss" scoped>
.date-value {
  height: 110px;
  border-left: 1px solid #fd7;
}
.event-by-day{
  height: 25px;
  border: none !important;
  .btn{
    font-size: 11px;
  }
}
.event-table {
  top: 0;
  left: 0;
}
</style>
