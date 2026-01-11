<template>
  <div v-if="tickets.length === 0" class="empty-state">
    <UIcon name="i-lucide-ticket" class="empty-icon" />
    <p class="empty-text">No tickets found</p>
  </div>

  <div v-else class="tickets-list">
    <UCard
      v-for="ticket in tickets"
      :key="ticket.id || Math.random()"
      class="ticket-card"
    >
      <div class="ticket-content">
        <div class="ticket-header">
          <h3 class="ticket-title">
            {{ ticket.title || "Untitled Ticket" }}
          </h3>
          <div class="flex items-center gap-2">
            <UBadge
              :color="getStatusColor(ticket.status)"
              variant="subtle"
              class="status-badge"
            >
              {{ formatStatus(ticket.status) }}
            </UBadge>
            <UButton
              class="cursor-pointer"
              icon="i-lucide-trash-2"
              color="red"
              variant="ghost"
              size="xs"
              :loading="props.deletingTicketId === ticket.id"
              @click="handleDelete(ticket)"
            />
          </div>
        </div>

        <p v-if="ticket.text" class="ticket-text">
          {{ ticket.text }}
        </p>

        <div class="ticket-footer">
          <div class="ticket-meta">
            <span v-if="ticket.task_id" class="meta-item">
              <UIcon name="i-lucide-clipboard-list" class="meta-icon" />
              <span class="meta-label">Task:</span>
              <span class="meta-value">
                {{ formatId(ticket.task_id) }}
              </span>
            </span>
            <span v-if="ticket.person_id" class="meta-item">
              <UIcon name="i-lucide-user-plus" class="meta-icon" />
              <span class="meta-label">Created by:</span>
              <span class="meta-value">
                {{ formatId(ticket.person_id) }}
              </span>
            </span>
            <span v-if="ticket.assignee_id" class="meta-item">
              <UIcon name="i-lucide-user-check" class="meta-icon" />
              <span class="meta-label">Assigned to:</span>
              <span class="meta-value">
                {{ formatId(ticket.assignee_id) }}
              </span>
            </span>
            <span v-if="ticket.project_id" class="meta-item">
              <UIcon name="i-lucide-factory" class="meta-icon" />
              <span class="meta-label">Production:</span>
              <span class="meta-value">
                {{ formatId(ticket.project_id) }}
              </span>
            </span>
            <span v-if="ticket.episode_id" class="meta-item">
              <UIcon name="i-lucide-tv" class="meta-icon" />
              <span class="meta-label">Episode:</span>
              <span class="meta-value">
                {{ formatId(ticket.episode_id) }}
              </span>
            </span>
          </div>
          <div class="ticket-dates">
            <div v-if="ticket.created_at" class="ticket-date">
              <UIcon name="i-lucide-calendar-plus" class="meta-icon" />
              <span class="meta-label">Created:</span>
              <span>{{ formatDate(ticket.created_at) }}</span>
            </div>
            <div
              v-if="
                ticket.updated_at && ticket.updated_at !== ticket.created_at
              "
              class="ticket-date"
            >
              <UIcon name="i-lucide-calendar-clock" class="meta-icon" />
              <span class="meta-label">Updated:</span>
              <span>{{ formatDate(ticket.updated_at) }}</span>
            </div>
          </div>
        </div>
      </div>
    </UCard>
    <div class="footer flex items-center">
      <p class="tickets-count text-center">{{ tickets.length }} tickets</p>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  tickets: {
    type: Array,
    required: true,
    default: () => [],
  },
  deletingTicketId: {
    type: String,
    default: null,
  },
});

const emit = defineEmits(["delete"]);

const handleDelete = (ticket) => {
  if (!ticket.id) {
    console.error("Ticket ID is missing");
    return;
  }
  emit("delete", ticket);
};

const getStatusColor = (status) => {
  if (!status) return "info";
  const statusLower = status.toLowerCase();
  if (statusLower === "open") return "success";
  if (statusLower === "on hold") return "warning";
  if (statusLower === "closed") return "neutral";
  return "info";
};

const formatStatus = (status) => {
  if (!status) return "Unknown";
  return status
    .split(" ")
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(" ");
};

const formatDate = (dateString) => {
  if (!dateString) return "";
  try {
    const date = new Date(dateString);
    return date.toLocaleDateString("en-US", {
      year: "numeric",
      month: "short",
      day: "numeric",
      hour: "2-digit",
      minute: "2-digit",
    });
  } catch {
    return dateString;
  }
};

const formatId = (id) => {
  if (!id) return "";
  return id.substring(0, 8) + "...";
};
</script>
