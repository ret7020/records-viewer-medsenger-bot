<template>
  <div style="padding-bottom: 15px;">
    <loading v-if="!loaded"/>
    <load-error v-else-if="state == 'load-error'"/>
    <action-done v-else-if="state == 'done'"/>
    <div v-else>
      <dashboard-header :patient="patient"/>

      <div
          v-if="window_mode == 'settings' || window_mode == 'graph' || window_mode == 'log' || window_mode == 'conclusion'">
        <div class="container slim-container" style="margin-top: 15px;">
          <dashboard :patient="patient" :categories="category_list"
                     v-show="state == 'dashboard' || state == 'graph-category-chooser'"/>
          <report :patient="patient" :categories="category_list" :data="data" v-show="state == 'report'"/>
          <conclusion-editor :patient="patient" v-show="state == 'conclusion'"/>
        </div>
        <graph-presenter :patient="patient" v-show="state == 'graph'"/>
      </div>

    </div>
  </div>
</template>

<script>
import DashboardHeader from "./components/parts/DashboardHeader";
import Loading from "./components/parts/Loading";
import Report from "./components/Report";
import Dashboard from "./components/Dashboard";
import LoadError from "./components/LoadError";
import GraphPresenter from "./components/GraphPresenter";
import * as moment from "moment/moment";
import ConclusionEditor from "./components/ConclusionEditor";
import ActionDone from "./components/ActionDone";

export default {
  name: 'App',
  components: {
    ActionDone,
    ConclusionEditor,
    GraphPresenter,
    LoadError,
    Dashboard,
    Report,
    Loading,
    DashboardHeader,
  },
  data() {
    return {
      state: "loading",
      patient: undefined,
      data: undefined,
      loaded: false
    }
  },
  methods: {
    load: function () {
      this.loaded = false
      this.axios.get(this.url('/api/settings/get_patient')).then(response => {
            this.patient = response.data
            this.axios.get(this.url('/api/categories')).then(this.process_load_answer);
          }
      );
    },
    process_load_answer: function (response) {
      this.category_list = response.data;

      if (this.window_mode == 'settings') {
        this.state = 'dashboard';
      }

      if (this.window_mode == 'conclusion') {
        this.state = 'conclusion';
      }

      if (this.window_mode == 'graph') {
        this.state = 'graph-category-chooser'
      }

      if (this.window_mode == 'log') {
        this.state = 'log'
        let params = {
          title: 'История назначений',
          categories: ['doctor_action'],
          filters: undefined
        }
        Event.fire('load-report', params)
      }
      this.loaded = true
    },
    process_load_error: function (response) {
      this.state = 'load-error'
    },

    load_records: function (data) {
      this.data = null

      this.axios.post(this.url('/api/report'), data).then(response => {
        this.data = {
          records: response.data.dates,
          page: data.page,
          pages: response.data.page_cnt,
          report: data.report
        }

        this.state = 'report'
      });
    },
  },
  created() {
    this.window_mode = window.MODE

    console.log("running created");
    this.load();

    Event.listen('back-to-dashboard', () => this.state = 'dashboard');
    Event.listen('load-error', () => this.state = 'load-error')
    Event.listen('action-done', () => this.state = 'done')

    Event.listen('load-records', (data) => {
      this.load_records(data)
    })

    Event.listen('load-report', (report) => {
      this.loaded = false
      let end_date = new Date(moment(this.patient.end_date).set({
        hour: 23,
        minute: 59,
        second: 59
      }).format('YYYY-MM-DD HH:mm:ss'))
      let today = new Date(moment().set({
        hour: 23,
        minute: 59,
        second: 59
      }).format('YYYY-MM-DD HH:mm:ss'))
      let end_filter_date = end_date < today ? end_date : today

      let data = {
        dates: [undefined, +end_filter_date / 1000],
        page: 0,
        categories: report.categories,
        report: report
      }

      this.load_records(data)
      this.state = 'report'
      this.loaded = true
    })

    Event.listen('load-graph', (params) => {
      this.state = 'graph'
    });
    Event.listen('load-day-graph', (params) => {
      this.state = 'graph'
    });
    Event.listen('load-heatmap', (params) => {
      this.state = 'graph'
    });

  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.container {
  max-width: 95%;
}

h1, h2 {
  font-weight: normal;
}

a {
  color: #006c88;
  font-weight: bold;
}

body {
  background-color: #fcfcfc;
  font-family: Roboto;
}

@media screen and (max-width: 900px) {
  .slim-container {
    max-width: 100% !important;
    padding-left: 10px;
    padding-right: 10px;
  }
}

.col, .col-1, .col-10, .col-11, .col-12, .col-2, .col-3, .col-4, .col-5, .col-6, .col-7, .col-8, .col-9, .col-auto, .col-lg, .col-lg-1, .col-lg-10, .col-lg-11, .col-lg-12, .col-lg-2, .col-lg-3, .col-lg-4, .col-lg-5, .col-lg-6, .col-lg-7, .col-lg-8, .col-lg-9, .col-lg-auto, .col-md, .col-md-1, .col-md-10, .col-md-11, .col-md-12, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6, .col-md-7, .col-md-8, .col-md-9, .col-md-auto, .col-sm, .col-sm-1, .col-sm-10, .col-sm-11, .col-sm-12, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6, .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-auto, .col-xl, .col-xl-1, .col-xl-10, .col-xl-11, .col-xl-12, .col-xl-2, .col-xl-3, .col-xl-4, .col-xl-5, .col-xl-6, .col-xl-7, .col-xl-8, .col-xl-9, .col-xl-auto {
  padding-right: 5px;
  padding-left: 5px;
}

.row {
  margin: 5px -5px;
}

.card {
  border-color: rgba(0, 108, 136, 0.3);
}

.btn-primary, .btn-primary:active, .btn-primary:hover, .btn-primary:focus, .btn-primary:disabled {
  border-color: #006c88;
  background-color: #006c88;
}

.btn-success, .btn-success:active, .btn-success:hover, .btn-success:focus, .btn-success:disabled {
  border-color: #24a8b4;
  background-color: #24a8b4;
}

.btn-danger, .btn-danger:active, .btn-danger:hover, .btn-danger:focus {
  border-color: #ff5763;
  background-color: #ff5763;
}

h5, h4, h3 {
  color: #006c88;
  margin-bottom: 15px;
  margin-top: 15px;
}

strong {
  font-weight: 500;
}

input[type=checkbox] {
  /* Double-sized Checkboxes */
  -ms-transform: scale(1.2); /* IE */
  -moz-transform: scale(1.2); /* FF */
  -webkit-transform: scale(1.2); /* Safari and Chrome */
  -o-transform: scale(1.2); /* Opera */
  transform: scale(1.2);
  margin: 10px;
}

</style>