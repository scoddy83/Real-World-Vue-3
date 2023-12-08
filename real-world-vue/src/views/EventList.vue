<script setup>
import EventCard from '../components/EventCard.vue'
import EventService from '../services/EventServices.js'
import { ref, onMounted, computed, watchEffect } from 'vue'

const props = defineProps(['page'])
const events = ref("")
const page = computed(() => props.page)
const totalEvents = ref(0)
const totalPages = ref(0)
const cardsPerPage = 2

const hasNextPage = computed(() => {
  // Calculate totalPages, based on 2 per page
  const totalPage = Math.ceil(totalEvents.value / cardsPerPage)

  // if current page is less than total pages, return true
  return page.value < totalPage
})


onMounted(() => {
  watchEffect(() => {
    events.value = null
    EventService.getEvents(cardsPerPage, page.value)
      .then((response) => {
        events.value = response.data
        // our response has total stored in the header.
        totalEvents.value = response.headers['x-total-count']
        totalPages.value = Math.ceil(totalEvents.value / cardsPerPage)
      })
      .catch((error) => {
        console.log(error)
      });
  })
});
</script>

<template>
  <h1>Events for Good</h1>
  <div class="events">
    <EventCard v-for="event in events" :key="event.id" :event="event" />

    <div class="pagination">
      <router-link
        :to="{ name: 'EventList', query: { page: page - 1 } }"
        rel="prev"
        v-if="page != 1"
      >
        &#60; Previous
      </router-link>

      <router-link v-for="n in totalPages" :key="n"
        :to="{ name: 'EventList', query: { page: n } }"
      >
        {{ n }} 
      </router-link>

      <router-link
        :to="{ name: 'EventList', query: { page: page + 1 } }"
        rel="next"
        v-if="hasNextPage"
      >
        Next &#62;
      </router-link>
    </div>
  </div>
</template>

<style scoped>
.events {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pagination {
  display: flex;
  width: 290px;
}
.pagination a {
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}
#page-prev {
  text-align: left;
}
#page-next {
  text-align: right;
}
</style>
