<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

interface Download {
  id: string
  filename: string
  status: string
  progress: number
  created_at: number
  stopped: boolean
}

const downloads = ref<Download[]>([])

onMounted(() => {
  requestDownloadStatus()
})

const sortedDownloads = computed(() => {
  return Object.values(downloads.value).sort((a, b) => b.created_at - a.created_at)
})

const formatTimestamp = (timestamp: number) => {
  const date = new Date(timestamp)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  return `${year}-${month}-${day} ${hours}:${minutes}`
}

const requestDownloadStatus = () => {
  const response = window.prompt('VERSO::DOWNLOAD_STATUS_GET')
  if (response !== null) {
    const status = JSON.parse(response) as { [id: string]: Download }
    downloads.value = Object.values(status)
  }
}

const onclickCancel = (id: string) => {
  window.prompt('VERSO::ABORT_DOWNLOAD::' + id)
}

setInterval(() => {
  requestDownloadStatus()
}, 2000)
</script>

<template>
  <div class="download-history">
    <h2 class="download-title">Download History</h2>
    <div class="download-container">
      <div v-for="download in sortedDownloads" :key="download.id" class="download-item">
        <div class="name-date-row">
          <div class="filename">{{ download.filename }}</div>
          <div class="timestamp">{{ formatTimestamp(download.created_at) }}</div>
        </div>
        <div class="progress-bar">
          <div class="progress" :style="{ width: `${download.progress}%` }"></div>
        </div>
        <div class="status-row">
          <div class="status">
            {{ download.status }}
          </div>
          <button v-if="!download.stopped" @click="onclickCancel(download.id)" class="cancel-btn">
            Cancel
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.download-history {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
}

.download-container {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.download-title {
  font-size: 2rem;
  font-weight: bold;
  margin-bottom: 1rem;
}

.download-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 0.75rem;
  background-color: #2a2a2a;
  border-radius: 0.5rem;
  color: #ffffff;

  .name-date-row {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
  }

  .filename {
    flex: 1;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .timestamp {
    flex: 1;
    font-size: 0.875rem;
    color: #9ca3af;
    text-align: right;
  }

  .progress-bar {
    flex: 1;
    width: 100%;
    height: 0.5rem;
    background-color: #404040;
    border-radius: 0.25rem;
    overflow: hidden;
  }

  .progress {
    height: 0.5rem;
    background-color: #ffffff;
    transition: width 0.3s ease;
  }

  .status-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .status {
    color: #9ca3af;
    text-transform: capitalize;
    font-size: 0.875rem;
  }

  .cancel-btn {
    padding: 0.25rem 0.75rem;
    background-color: #dc2626;
    color: white;
    border: none;
    border-radius: 0.25rem;
    cursor: pointer;
    font-size: 0.875rem;
    transition: background-color 0.2s;

    &:hover {
      background-color: #b91c1c;
    }
  }
}
</style>
