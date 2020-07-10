<template>
  <div id="app">
    <div class="header-container">
      <div class="title-container">
        <div class="title">空氣品質指標 (AQI)</div>
        <div class="select-wrapper">
          <select class="county-select" v-model="selectedCounty">
            <option value="" selected disabled>請選擇地區</option>
            <option
              v-for="(county, index) of counties"
              :value="county"
              :key="index"
              >{{ county }}</option
            >
          </select>
        </div>
      </div>
      <div class="level-container">
        <div v-for="(level, index) of levels" :key="index" class="level">
          <div class="range" :class="[level.x]">
            {{ level.from }}~{{ level.to }}
          </div>
          <div class="desc">{{ level.desc }}</div>
        </div>
      </div>
    </div>
    <div class="county-container">
      <div class="county">{{ selectedCounty }}</div>
      <div class="spacer"></div>
      <div class="updateAt">
        {{ updateAt ? updateAt.toLocaleString() : '' }} 更新
      </div>
    </div>
    <div class="sites-container">
      <div v-for="(site, index) of filteredSites" :key="index" class="site">
        <div class="site-name">{{ site.siteName }}</div>
        <div class="aqi" :class="[site.level]">{{ site.aqi }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import * as axios from 'axios';
import { levels } from './constants/levels';

export default {
  name: 'App',
  data: function() {
    return {
      levels,
      dataset: [],
      counties: [],
      selectedCounty: undefined,
      selectedSiteData: undefined,
      updateAt: undefined,
    };
  },
  methods: {
    getLevel: function(aqi) {
      for (let i = 0, level = this.levels[0]; i < this.levels.length; i++) {
        if (level.from <= aqi && aqi <= level.to) {
          return level.x;
        }
      }

      return '';
    },
  },
  mounted: function() {
    axios
      .get(
        'https://opendata.epa.gov.tw/api/v1/AQI?%24skip=0&%24top=1000&%24format=json'
      )
      .then((res) => res.data)
      .then((records) => {
        records.forEach((record) => {
          const level = this.getLevel(Number.parseInt(record.AQI));
          console.log(Number.parseInt(record.AQI));

          this.dataset.push({
            siteName: record.SiteName,
            County: record.County,
            aqi: record.AQI,
            level,
          });

          if (!this.counties.includes(record.County)) {
            this.counties.push(record.County);
          }
          // this.counties.sort();
        });

        this.updateAt = new Date();
        this.selectedCounty = this.counties[0];

        console.log(this.dataset);
      });
  },
  computed: {
    filteredSites: function() {
      return this.selectedCounty
        ? this.dataset.filter(
            (siteData) => siteData.County === this.selectedCounty
          )
        : [];
    },
  },
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background-color: #f0f0f0;
  font-weight: 700;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#app {
  width: 100%;
  max-width: 1200px;
  margin: auto;
  padding: 3rem;

  display: flex;
  flex-direction: column;
}

/* .header-container */
.header-container {
  display: flex;
}
.header-container .title-container {
  width: 40%;
  padding: 0 15px;
  display: flex;
  flex-direction: column;
}
.header-container .level-container {
  width: 60%;
  padding: 0 15px;
  display: flex;
}

/* title-container */
.header-container .title-container .title {
  margin-bottom: 0.5rem;

  font-size: 2.5rem;
}
.header-container .title-container .select-wrapper {
  position: relative;
}
.header-container .title-container .select-wrapper select {
  width: 100%;
  height: 50px;
  padding-left: 1.5rem;

  border: 2px solid #000;
  appearance: none;
}
.header-container .title-container .select-wrapper::before {
  content: ' ';
  position: absolute;
  border-style: solid;
  top: 17px;
  right: 15px;
  border-width: 0 4px 7px 4px;
  border-color: transparent transparent #000 transparent;
}
.header-container .title-container .select-wrapper::after {
  content: ' ';
  position: absolute;
  border-style: solid;
  right: 15px;
  bottom: 17px;

  width: 0;
  height: 0;
  border-width: 7px 4px 0 4px;
  border-color: #000 transparent transparent transparent;
}

/* level-container */
.header-container .level-container .level {
  width: 100%;
  border: 3px solid #000;
  border-right: none;

  display: flex;
  flex-direction: column;

  background-color: #fff;
}
.header-container .level-container .level .range {
  height: 50%;
  padding: 0 15px;
  border-bottom: 3px solid #000;

  display: flex;
  justify-content: center;
  align-items: center;

  text-align: center;
}
.header-container .level-container .level:last-child {
  border-right: 3px solid #000;
}
.header-container .level-container .level .desc {
  height: 50%;
  padding: 0 15px;

  display: flex;
  justify-content: center;
  align-items: center;

  text-align: center;
}

/* county-container */
.county-container {
  margin: 3rem 0;
  padding: 0 15px;

  display: flex;
  align-items: center;
}
.county-container .county {
  font-size: 2.25rem;
}
.county-container .spacer {
  height: 0;
  margin: 0 1.125rem;
  border-top: 3px dashed #000;
  flex-grow: 1;
}

/* sites-container */
.sites-container {
  display: flex;
  flex-wrap: wrap;
}
.sites-container .site {
  width: calc(33.33% - 36px);
  height: 90px;
  margin: 0 15px 48px 15px;
  border: 3px solid #000;

  display: flex;

  background-color: #fff;
  font-size: 40px;
}
.sites-container .site .site-name {
  border-right: 3px solid #000;

  flex-grow: 1;

  display: flex;
  justify-content: center;
  align-items: center;
}
.sites-container .site .aqi {
  width: 150px;

  display: flex;
  justify-content: center;
  align-items: center;
}

.level-1 {
  background-color: #95f084;
}
.level-2 {
  background-color: #ffe695;
}
.level-3 {
  background-color: #ffaf6a;
}
.level-4 {
  background-color: #ff5757;
}
.level-5 {
  background-color: #9777ff;
}
.level-6 {
  background-color: #ad1774;
}
</style>
