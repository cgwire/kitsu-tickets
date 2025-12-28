<template>
  <div>
    <h1>Hello World</h1>


  </div>
</template>

<script setup lang="ts">
const route = useRoute()

const { client, fetchTickets } = useKitsu()

const isLoggedIn = ref(null)
const openProductions = ref([])
const tickets = ref([])

const productionId = computed(() => route.query.production_id)
const episodeId = computed(() => route.query.episode_id)
const isDarkTheme = computed(() => route.query.dark_theme === 'true')
const isStudioPage = computed(() => !productionId.value && !episodeId.value)

onMounted(() => {
  console.log(productionId.value)
  fetchData()
})

const fetchData = async () => {
  console.log('fetching data')
  const isLoggedInResponse = await client.isLoggedIn()
  isLoggedIn.value = isLoggedInResponse.isLoggedIn
  console.log(isLoggedInResponse)

  const openProductionsResponse = await client.getOpenProductions()
  console.log(openProductionsResponse)
  openProductions.value = openProductionsResponse
  console.log(openProductionsResponse)

  const ticketsResponse = await fetchTickets(productionId.value, episodeId.value)
  console.log(ticketsResponse)
  tickets.value = ticketsResponse
  console.log(ticketsResponse)
}
</script>
