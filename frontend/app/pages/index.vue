<template>
  <div v-if="isLoading" class="loading-state">
    <UIcon name="i-lucide-loader-circle" class="animate-spin" size="24" />
    <span>{{ $t('tickets.loading') }}</span>
  </div>

  <div v-else-if="!isLoggedIn" class="empty-state">
    <UIcon name="i-lucide-log-in" size="48" />
    <p class="empty-text">{{ $t('tickets.not_logged_in') }}</p>
  </div>

  <div v-else class="tickets-container">
    <div class="tickets-header">
      <div class="header-content">
        <div>
          <h1 class="tickets-title">{{ $t('tickets.title') }}</h1>
          <p class="tickets-count">
            {{ $t('tickets.count', filteredTickets.length) }}
            <span v-if="isStudioPage"> {{ $t('tickets.all_productions') }}</span>
          </p>
        </div>
        <UButton
          icon="i-lucide-plus"
          class="cursor-pointer"
          color="neutral"
          :label="$t('tickets.create.submit')"
          variant="subtle"
          @click="openCreateModal"
        />
        <TicketFormModal
          :is-loading="isSaving"
          :production-id="productionId"
          :episode-id="episodeId"
          :edit-ticket="editingTicket"
          v-model="isFormModalOpen"
          @submit="handleSubmitTicket"
          @close="closeModal"
        />
      </div>
    </div>

    <TicketList
      :tickets="filteredTickets"
      :productions="productions"
      :people="people"
      :deleting-ticket-id="deletingTicketId"
      @edit="handleEditTicket"
      @delete="handleDeleteTicket"
    />
  </div>
</template>

<script setup>
const route = useRoute()

const { client, getOpenProductions, getPeople, fetchTickets, createTicket, updateTicket, deleteTicket } = useKitsu()

const isLoggedIn = ref(null)
const tickets = ref([])
const productions = ref([])
const people = ref([])
const isLoading = ref(true)
const isFormModalOpen = ref(false)
const isSaving = ref(false)
const editingTicket = ref(null)
const deletingTicketId = ref(null)

const productionId = computed(() => route.query.production_id)
const episodeId = computed(() => route.query.episode_id)
const isStudioPage = computed(() => !productionId.value && !episodeId.value)

onMounted(() => {
  fetchData()
})

const filteredTickets = computed(() => {
  return tickets.value
    .filter((ticket) => {
      if (isStudioPage.value) {
        return true
      } else if (episodeId.value) {
        return (
          ticket.project_id === productionId.value
          && ticket.episode_id === episodeId.value
        )
      } else if (productionId.value) {
        return ticket.project_id === productionId.value
      }
      return true
    })
    .sort((a, b) => new Date(b.created_at) - new Date(a.created_at))
})

const fetchData = async () => {
  try {
    isLoading.value = true
    const isLoggedInResponse = await client.isLoggedIn()
    isLoggedIn.value = isLoggedInResponse.isLoggedIn
    const [ticketsData, productionsData, peopleData] = await Promise.all([
      fetchTickets(productionId.value, episodeId.value),
      getOpenProductions(),
      getPeople()
    ])
    tickets.value = ticketsData
    productions.value = productionsData
    people.value = peopleData
  } catch (error) {
    console.error('Error fetching data:', error)
    tickets.value = []
  } finally {
    isLoading.value = false
  }
}

const handleSubmitTicket = async (ticketData) => {
  const ticketToEdit = editingTicket.value
  try {
    isSaving.value = true
    if (ticketToEdit) {
      const response = await updateTicket(ticketToEdit.id, ticketData)
      console.log('updateTicket response:', response)
      const index = tickets.value.findIndex((t) => t.id === ticketToEdit.id)
      if (index !== -1) {
        tickets.value[index] = { ...tickets.value[index], ...ticketData }
      }
    } else {
      const created = await createTicket(ticketData)
      tickets.value.unshift(created)
    }
  } catch (error) {
    console.error('Error saving ticket:', error)
  } finally {
    isSaving.value = false
  }
}

const openCreateModal = () => {
  editingTicket.value = null
  isFormModalOpen.value = true
}

const handleEditTicket = (ticket) => {
  editingTicket.value = ticket
  isFormModalOpen.value = true
}

const closeModal = () => {
  isFormModalOpen.value = false
  editingTicket.value = null
}

const handleDeleteTicket = async (ticket) => {
  deletingTicketId.value = ticket.id
  try {
    await deleteTicket(ticket.id)
    tickets.value = tickets.value.filter((t) => t.id !== ticket.id)
  } catch (error) {
    console.error('Error deleting ticket:', error)
  } finally {
    deletingTicketId.value = null
  }
}
</script>
