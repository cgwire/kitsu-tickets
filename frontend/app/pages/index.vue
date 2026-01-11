<template>
  <div v-if="isLoading">
    <div class="flex items-center gap-4">
      <div class="flex items-center gap-2">Loading...</div>
      <UIcon name="i-lucide-loader-circle" class="animate-spin" />
      <div class="grid gap-2">
        <USkeleton class="h-4 w-[250px]" />
        <USkeleton class="h-4 w-[200px]" />
        <USkeleton class="h-4 w-[200px]" />
        <USkeleton class="h-4 w-[200px]" />
      </div>
    </div>
  </div>

  <div v-else-if="!isLoggedIn">
    <div class="empty-state">Not logged in</div>
  </div>

  <div class="tickets-container" v-else>
    <p v-if="productionId">Production ID: {{ productionId }}</p>
    <p v-if="episodeId">Episode ID: {{ episodeId }}</p>
    <p v-if="isStudioPage">Studio Page</p>
    <div class="tickets-header flex">
      <span class="flex-1"></span>
      <div class="cursor-pointer">
        <CreateTicketModal
          ref=""
          :is-loading="isCreating"
          :production-id="productionId"
          :episode-id="episodeId"
          @submit="handleCreateTicket"
          @close="closeModal"
          v-model="isCreateModalOpen"
        />
      </div>
    </div>

    <TicketList
      v-if="!isLoading"
      :tickets="filteredTickets"
      :deleting-ticket-id="deletingTicketId"
      @delete="handleDeleteTicket"
    />
  </div>
</template>

<script setup>
const route = useRoute();
const colorMode = useColorMode();

const { client, fetchTickets, createTicket, deleteTicket } = useKitsu();

const isLoggedIn = ref(null);
const tickets = ref([]);
const isLoading = ref(true);
const isCreateModalOpen = ref(false);
const isCreating = ref(false);
const deletingTicketId = ref(null);

const productionId = computed(() => route.query.production_id);
const episodeId = computed(() => route.query.episode_id);
const isDarkTheme = computed(() => route.query.dark_theme === "true");
const isStudioPage = computed(() => !productionId.value && !episodeId.value);

colorMode.value = "light";

onMounted(() => {
  fetchData();
});

const filteredTickets = computed(() => {
  return tickets.value.filter((ticket) => {
    if (isStudioPage.value) {
      return true;
    } else if (productionId.value) {
      return ticket.project_id === productionId.value;
    } else if (episodeId.value) {
      return (
        ticket.project_id === productionId.value &&
        ticket.episode_id === episodeId.value
      );
    }
  });
});

const fetchData = async () => {
  try {
    isLoading.value = true;
    const isLoggedInResponse = await client.isLoggedIn();
    isLoggedIn.value = isLoggedInResponse.isLoggedIn;

    tickets.value = await fetchTickets(productionId.value, episodeId.value);
  } catch (error) {
    console.error("Error fetching data:", error);
    tickets.value = [];
  } finally {
    isLoading.value = false;
  }
};

const handleCreateTicket = async (ticketData) => {
  try {
    isCreating.value = true;

    const createdTicket = await createTicket(ticketData);
    tickets.value.unshift(createdTicket);
  } catch (error) {
    console.error("Error creating ticket:", error);
    alert("Failed to create ticket: " + (error.message || "Unknown error"));
    isCreateModalOpen.value = true;
  } finally {
    isCreating.value = false;
  }
};

const closeModal = () => {
  isCreateModalOpen.value = false;
};

const handleDeleteTicket = async (ticket) => {
  if (!ticket.id) {
    console.error("Ticket ID is missing");
    return;
  }

  deletingTicketId.value = ticket.id;

  try {
    await deleteTicket(ticket.id);
    tickets.value = tickets.value.filter((t) => t.id !== ticket.id);
  } catch (error) {
    console.error("Error deleting ticket:", error);
    alert("Failed to delete ticket: " + (error.message || "Unknown error"));
  } finally {
    deletingTicketId.value = null;
  }
};
</script>
