<template>
  <div class="leagues-container">
    <div class="page-header">
      <h1>HALITE LEAGUES</h1>
      <i class="xline xline-bottom"></i>
    </div>
    <div class="hackathon-events-container">
      <div class="filter-form">
          <div class="form-header">
            <div class="filter-handler">
              <a href="#" class="handler-item" @click="clearFilter">
                <span class="handler-item-img icon-remove"></span>
                <span class="handler-item-text">Clear all</span>
              </a>
            </div>
          </div>
          <div class="filter-group">
            <div class="input-group">
              <v-select
                multiple
                placeholder="Name"
                v-model="filter_name"
                :options="nameOptions">
              </v-select>
              <v-select
                multiple
                placeholder="Category"
                v-model="filter_cat"
                :options="catOptions">
              </v-select>
              <!-- <div>
                <button class="btn"><span>APPLY FILTER</span></button>
              </div> -->
            </div>
          </div>
        </div>
      <div class="table-container">
        <table class="table table-leader leagues-table">
          <thead>
            <tr>
              <th class="league-col-name">League name</th>
              <th class="league-col-desc">Description</th>
              <th class="league-col-category">Category</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="league in finalLeagues" :key="league.id">
              <td>
                <a :href="getLink(league)">{{league.name}}</a>
              </td>
              <td>{{league.description}}</td>
              <td>{{league.category}}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
<script>
  import * as api from '../api'
  import vSelect from 'vue-select'
  import _ from 'lodash'

  export default {
    name: "LeaguesPortal",
    props: ['baseUrl'],
    components: {
      vSelect
    },
    data: function () {
      return {
        leagues: [],
        filter_name: [],
        filter_cat: [],
        catOptions: [],
        nameOptions: []
      }
    },
    mounted: function(){
      // get params 
      const params = new URLSearchParams(window.location.search)
      if (params.has('category')){
        this.filter_cat.push(decodeURI(params.get('category')))
      }

      api.getLeaguesList().then((leagues) => {
        console.log(leagues)
        this.leagues = leagues

        // update filter options
        let catOptions = [];
        let nameOptions = [];

        this.leagues.forEach((item) => {
          if (item.category && catOptions.indexOf(item.category) == -1){
            catOptions.push(item.category)
          }
          if (item.name && nameOptions.indexOf(item.name) == -1){
            nameOptions.push(item.name)
          }
        })
        this.catOptions = catOptions
        this.nameOptions = nameOptions
      });
    },
    computed: {
      finalLeagues: function(){
        return _.filter(this.leagues, (item) => {
          let nameIncluded = !this.filter_name.length || this.filter_name.indexOf(item.name) != -1;
          let catIncluded = !this.filter_cat.length || this.filter_cat.indexOf(item.category) != -1;
          return nameIncluded && catIncluded
        });
      }
    },
    methods: {
      getLink: function(league){
        let link = `/league-board?id=${league.id}&leaguename=${league.name}`
        
        link += '&' + league.query;
        return encodeURI(link);
      },
      clearFilter: function(){
        this.filter_name = [];
        this.filter_cat = [];
      }
    }
  }
</script>