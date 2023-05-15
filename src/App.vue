<script setup lang="ts">

import { ref, onMounted } from 'vue';

interface Job {
  id: number;
  company: string;
  logo: string;
  new: boolean;
  featured: boolean;
  position: string;
  role: string;
  level: string;
  postedAt: string;
  contract: string;
  location: string;
  languages: string[];
  tools: string[];
}
const jobData = ref<Job[] | null>(null)
const filterData = ref<string[]>([])
// fetching jsondata on mounted

onMounted(async () => {
  try {
    const response = await fetch('/data.json');
    jobData.value = await response.json()
  } catch (error) {
    console.error('error')
  }
})
// dynamic logo image fetch
const getLogoImage = (logo:string):string => {
  return `${logo}`
}

const filter = (filter:string) => {
  // preventing duplicate value on filter
  if (!filterData.value.includes(filter)) {
    filterData.value = [...filterData.value, filter];
  }
}

const clearFilter = () => {
  filterData.value = [];
}
const removeFilter = (filter: string) => {
  filterData.value = filterData.value.filter((f) => f !== filter);
};

const displayCard = (item:Job):boolean => {
  const { tools,languages, role, level } = item;
  if (filterData.value.length === 0) {
    return true; //initial state, show all cards
  }
  const filtersMatch = filterData.value.every((filter) => {
    if (filter === role || filter === level) {
      return true;
    }
    if (tools.includes(filter) || languages.includes(filter)) {
      return true;
    }
    return false;
  });

  return filtersMatch;
} 
</script>

<template>
  <header>
    <picture>
      <source media="(max-width: 750px)" srcset="/images/bg-header-mobile.svg">
      <source media="(min-width: 750px)" srcset="/images/bg-header-desktop.svg">
      <img src="/images/bg-header-mobile.svg" alt="Header Background">
    </picture>
  </header>

  <main>
    <div v-if="filterData.length !== 0" class="filter_container">
      <div class="filter-pill-container">
        <div v-for="(filter,index) in filterData" :key="index" class="filter_pill">
          <p>{{ filter }}</p>
          <button @click="removeFilter(filter)">
            <img src="./images/icon-remove.svg" alt="close-icon">
          </button>
        </div>
      </div>
      <button @click="clearFilter">Clear</button>
    </div>
    <div class="main_job_container">
      <!-- Looping job cards -->
      <div v-for="(item) in jobData" :key="item.id">
        <div v-show="displayCard(item)" :class="{'featured': item.featured}"  class="main_job_card">
          <img class="image-logo" :src='getLogoImage(item.logo)' :alt="item.company">
          <div class="job_card_info">
            <div class="job_info_header">
              <p>{{ item.company }}</p>
              <div class="new_pill" v-if="item.new">
                <p>NEW!</p>
              </div>
              <div class="featured_pill" v-if="item.featured">
                <p>FEATURED</p>
              </div>
            </div>
            <h2>{{ item.position }}</h2>
            <div class="job_info_placement">
              <p>{{ item.postedAt }}</p>
              <div class="dot"></div>
              <p>{{ item.role }}</p>
              <div class="dot"></div>
              <p>{{ item.location }}</p>
            </div>
          </div>
          <hr>
          <div class="job_card_footer">
            <button @click="filter(item.role)">{{ item.role }}</button>
            <button @click="filter(item.level)">{{ item.level }}</button>
            <button @click="filter(lang)" v-for="lang in item.languages" :key="lang">{{ lang }}</button>
            <button @click="filter(tool)" v-for="tool in item.tools" :key="tool">{{ tool }}</button>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>
