<script setup>
import { ref, computed, onBeforeMount, onMounted } from 'vue';
import router from '../../router';
import { useArticlesStore } from '../../stores/articles.js';
import { client } from '../../contentful';

const articlesStore = useArticlesStore();
const spaceID = import.meta.env.VITE_SPACE_ID;
const environmentID = import.meta.env.VITE_ENVIRONMENT_ID;

const blogPoll = ref([])
const selectedAnswer = ref(null);
const selectedPoll = ref(null);
const latestFourArticles = ref([]);
let categoryVotes = ref(0)

const displayedArticles = computed(() => {
  return articlesStore.searchedArticles.length > 0
    ? articlesStore.searchedArticles
    : articlesStore.articles;
});

function voteCountPercentage(updatedEntry, selectedCategory){
    console.log(selectedCategory)
    let votes = 0;
    if(selectedCategory === 'developers') {
        updatedEntry.Developers.options.forEach(entry => votes += entry.votes)
    }
    else if(selectedCategory === 'gaming') {
        updatedEntry.Gaming.options.forEach(entry => votes += entry.votes)
    }
    else {
        updatedEntry.Tech.options.forEach(entry => votes += entry.votes)
    }
    categoryVotes.value = votes;
}

//here we update the votes of a selected answer by:
//1. getting the selected answer(option) by id
//2. updating its vote number by 1(+1)
//3. and based on the selected answers category(provided on the pollEntry variable(object) e.g Developers, Gaming, Tech), we update the poll answer(option
// * and OPTIONALLY, we set the width of the selected poll(by percentage) for styling purposes
function selectAnswer(answerID, answerIndex, pollID, question, answer, poll) {
  selectedAnswer.value = answerID;
  selectedPoll.value = pollID;
  let selectedCategory;

  //we get the selected topic by using the object answer ...in array
  client.getSpace(spaceID)
 .then((space) => space.getEnvironment('master'))
  .then((environment) => environment.getEntry(environmentID))
  .then((entry) => {
    let pollEntry = entry.fields.poll['en-US'];
    let pollEntryCategoryToUpdate = pollEntry.Developers.question === poll.question ? pollEntry.Developers : pollEntry.Gaming.question === poll.question ? pollEntry.Gaming : pollEntry.Tech;
    let selectedPollEntryToUpdate = pollEntryCategoryToUpdate.options[answerIndex];
    selectedPollEntryToUpdate.votes = selectedPollEntryToUpdate.votes + 1;

    if(pollEntry.Developers.question === pollEntryCategoryToUpdate.question){
        // console.log('developers')
        // console.log(entry.fields.poll['en-US'])
        selectedCategory = 'developers';
        entry.fields.poll['en-US'].Developers.options[answerIndex] = selectedPollEntryToUpdate;
    } else if (pollEntry.Gaming.question === pollEntryCategoryToUpdate.question) {
        selectedCategory = 'gaming';
        entry.fields.poll['en-US'].Gaming.options[answerIndex] = selectedPollEntryToUpdate;
    } else {
        selectedCategory = 'tech';
        entry.fields.poll['en-US'].Tech.options[answerIndex] = selectedPollEntryToUpdate;
    }

    return entry.update()
  })
  .then((entry) => {
    entry.publish();
    console.log(`Entry ${entry.sys.id} updated.`)
    console.log(entry)
    const updatedEntry = entry.fields.poll['en-US'];
    voteCountPercentage(updatedEntry, selectedCategory)
  })

}

function fetchArticle(id) {
    router.push(`/read-article/${id}`)
}

onBeforeMount(() => {
    articlesStore.clearSearchArticles();
})

//fetch all articles from CMS
onMounted(async () => {
    const art = await articlesStore.fetchAllArticles();
    const pollQuestions = articlesStore.pollQuestions.poll;
    const { Developers, Gaming, Tech } = pollQuestions;

    const poll = [Developers, Gaming, Tech];
    blogPoll.value = poll;
    const fourArticles = art.filter((article) => article.id > 1);
    latestFourArticles.value = fourArticles
})
</script>

<template>
  <Navbar/>  
  <main id="home-wrapper">
    <!-- LATEST 5 ARTICLES / Top of the month -->
    <!-- <section id="latest-articles">
        <div class="main-latest">
          <div class="overlay"></div>
          <img :src="articlesStore.articles[0].img" alt="" class="article-image">
          <p class="title main-title">{{ articlesStore.articles[0].title }}</p>  
        </div>
        <div class="sub-latest" v-if="articlesStore.articles.length  > articlesStore.articles.length - 1">
            <div class="sub-article" v-for="(article, index) in latestFourArticles">
                <div class="overlay"></div>
                <img :src="article.img ? article.img : ''" alt="" class="latest-article-img">
                <p class="title">{{ article.title }}</p> 
                <p class="article-date">{{ article.date }}</p>
            </div>
        </div>
    </section> -->

    <!-- <div id="heading-wrapper">
       <h2 id="new-articles-heading">New Articles</h2>
       <div class="heading-underline"></div>
    </div> -->

    <!-- NEW ARTICLES -->
    <!-- <section class="all-new-articles">
      <div class="new-article" v-for="article in displayedArticles" @click="fetchArticle(article.id)">
        <img :src="article.img" alt="" class="new-article-img">
        <p class="new-article-title"><span class="span-new-article-title">{{ article.title }}</span></p>
        <p class="date">{{ article.date }}</p>
        <p
          :class="{
            category: true,
            developers: article.category === 'Developers',
            gaming: article.category === 'Gaming',
            tech: article.category === 'Tech'
          }"
        >
          {{ article.category }}
        </p>
      </div>
    </section>

    <div id="heading-wrapper">
       <h2 id="new-articles-heading">POLL'S</h2>
       <div class="heading-underline"></div>
    </div> -->


    <!-- POLL -->
    <!-- <section id="poll">
        <div class="poll-area" v-for="(poll, index) in blogPoll">
            <p class="poll-question">{{ poll.question }}</p>
            <div class="opt" v-for="answer, answerIndex in poll.options" @click="selectAnswer(answer.id, answerIndex,  poll.id, poll.question, answer.answer, poll)">
              <div
               id="percentage"
               v-if="answer.id === selectedAnswer && poll.id === selectedPoll"
               :style="{ 'width' : answer.votes * 100 / categoryVotes + '%' }"
               >
            </div>
              <p class="option-name">{{ answer.answer }}</p>
            </div>
        </div>
    </section> -->
  </main>
</template>

<style scoped>
</style>
