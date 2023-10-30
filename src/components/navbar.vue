<script setup>
// import logo from '../assets/logo-2.png';
import { ref, watch, computed } from 'vue';
import { useArticlesStore } from '../stores/articles';


const articlesStore = useArticlesStore();
const activeNavItem = ref('home');
const searchText = ref('');
const microphone = ref(null);
const mobileNav = ref(null);
const mobileNavStatus = ref(false);

function setActiveNavItem(item) {
  activeNavItem.value = item;
}

watch(searchText, () => {
  articlesStore. searchArticles(searchText.value)
})

//searching the articles by voice
function searchArticleByVoice() {
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  const recognition = new SpeechRecognition();
  recognition.start();

  recognition.onstart = function () {
    console.log('Listening, please speak...');
    microphone.value.style.color = '#fff';
  };

  recognition.onresult = function (event) {
    const transcript = event.results[0][0].transcript.toLowerCase();
    const articleName = transcript;
    // console.log(articleName, transcript)
    articlesStore.searchArticles(articleName);
    microphone.value.style.color = '#ed1d3b';
  };
}

//open//close of mobile nav
function changeMobileNavStatus() {
  mobileNavStatus.value = !mobileNavStatus.value;
}
</script>


<template>
  <nav class="navbar">
    <!-- <div class="logo nav-item-1">
      <img :src="logo" alt="" id="logo">
      <i class="fa-solid fa-bars" @click="changeMobileNavStatus"></i>
    </div> -->

    <!-- <div class="menu nav-item-2">
      <div :class="{ 'nav-item': true, 'active': activeNavItem === 'home' }" @click="setActiveNavItem('home')">
        <RouterLink to="/">Home</RouterLink>
      </div>
      <div :class="{ 'nav-item': true, 'active': activeNavItem === 'developer' }" @click="setActiveNavItem('developer')">
        <RouterLink to="/developers">Developers</RouterLink>
      </div>
      <div :class="{ 'nav-item': true, 'active': activeNavItem === 'gaming' }" @click="setActiveNavItem('gaming')">
        <RouterLink to="/gaming">Gaming</RouterLink>
      </div>
      <div :class="{ 'nav-item': true, 'active': activeNavItem === 'tech' }" @click="setActiveNavItem('tech')">
        <RouterLink to="/tech"> Tech</RouterLink>
      </div>
    </div> -->

      <!-- Search Input -->
      <!-- <div class="search-input-wrapper nav-item">
        <input class="search" v-model="searchText" type="text" placeholder="Search..." />
        <div class="search-icon-wrapper">
          <i class="fa fa-search"></i>
        </div>
        <div class="voice-icon-wrapper" @click="searchArticleByVoice">
          <i ref="microphone" class="fa-solid fa-microphone"></i>
        </div>
    </div> -->
  </nav>



  <!-- MOBILE NAV -->
  <!-- <nav :class="{'mobile-nav': mobileNavStatus, 'mobile-nav-status': true}" ref="mobileNav">
    <i :class="{'fa-solid fa-x': mobileNavStatus}" @click="changeMobileNavStatus"></i>
    <div :class="{ 'mobile-nav-item': true, 'active': activeNavItem === 'home' }" @click="setActiveNavItem('home')"><RouterLink to="/">Home</RouterLink></div>
    <div :class="{ 'mobile-nav-item': true, 'active': activeNavItem === 'developer' }" @click="setActiveNavItem('developer')"><RouterLink to="/developers">Developers</RouterLink></div>
    <div :class="{ 'mobile-nav-item': true, 'active': activeNavItem === 'gaming' }" @click="setActiveNavItem('gaming')"><RouterLink to="/gaming">Gaming</RouterLink></div>
    <div :class="{ 'mobile-nav-item': true, 'active': activeNavItem === 'tech' }" @click="setActiveNavItem('tech')"><RouterLink to="/tech"> Tech</RouterLink></div>
  </nav> -->
</template>


<style scoped>
</style>