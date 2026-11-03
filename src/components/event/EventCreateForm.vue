<script setup>
import { reactive, ref } from 'vue'
import AppCard from '@/components/ui/AppCard.vue'
import AppButton from '@/components/ui/AppButton.vue'

const props = defineProps({
  submitting: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['submit', 'cancel'])

// STATE FORMULIR CREATE EVENT
const form = reactive({
  title: '',
  cat: 'Workshop',
  date: '',
  loc: '',
  quota: 100,
  price: 'Free',
  desc: '',
})

// STATE ERROR SETIAP KOLOM (FIELD-LEVEL ERROR STATE)
const errors = reactive({
  title: '',
  date: '',
  loc: '',
  quota: '',
  desc: '',
})

const errorMessage = ref('')
const successMessage = ref('')

// Format helper to convert YYYY-MM-DDTHH:mm into "Nov 25, 2026 • 10:00 AM"
const formatEventDate = (dt) => {
  if (!dt) return ''
  const dateObj = new Date(dt)
  if (isNaN(dateObj.getTime())) return dt
  const datePart = dateObj.toLocaleDateString('en-US', {
    month: 'short',
    day: '2-digit',
    year: 'numeric',
  })
  const timePart = dateObj.toLocaleTimeString('en-US', {
    hour: '2-digit',
    minute: '2-digit',
    hour12: true,
  })
  return `${datePart} • ${timePart}`
}

// VALIDASI PER KOLOM MASUKAN
const validateField = (field) => {
  switch (field) {
    case 'title':
      if (!form.title.trim()) {
        errors.title = 'Event title is required.'
      } else if (form.title.trim().length < 3) {
        errors.title = 'Event title must be at least 3 characters.'
      } else {
        errors.title = ''
      }
      break

    case 'date':
      if (!form.date) {
        errors.date = 'Event date and time is required.'
      } else {
        errors.date = ''
      }
      break

    case 'loc':
      if (!form.loc.trim()) {
        errors.loc = 'Event location is required.'
      } else {
        errors.loc = ''
      }
      break

    case 'quota':
      if (!form.quota || Number(form.quota) < 1) {
        errors.quota = 'Quota must be at least 1 attendee.'
      } else {
        errors.quota = ''
      }
      break

    case 'desc':
      if (!form.desc.trim()) {
        errors.desc = 'Event description is required.'
      } else if (form.desc.trim().length < 10) {
        errors.desc = 'Please write at least 10 characters for the description.'
      } else {
        errors.desc = ''
      }
      break
  }
}

// VALIDASI SELURUH FORMULIR SEBELUM SUBMIT
const validateAll = () => {
  validateField('title')
  validateField('date')
  validateField('loc')
  validateField('quota')
  validateField('desc')

  return !errors.title && !errors.date && !errors.loc && !errors.quota && !errors.desc
}

// HANDLER SUBMIT FORM CREATE EVENT KE REST API
const handleSubmitEvent = () => {
  const isValid = validateAll()

  if (!isValid) {
    errorMessage.value = 'Please correct the highlighted fields below before publishing the event.'
    successMessage.value = ''
    return
  }

  errorMessage.value = ''

  emit('submit', {
    title: form.title,
    cat: form.cat,
    date: formatEventDate(form.date),
    loc: form.loc,
    quota: form.quota,
    price: form.price,
    desc: form.desc,
  })
}
</script>

<template>
  <AppCard class="create-event-card">
    <div class="form-header">
      <h3>Create New Event</h3>
      <p class="subtitle">Fill in the details to publish a new event to the catalog.</p>
    </div>

    <!-- ERROR BANNER -->
    <div v-if="errorMessage" class="error-banner">⚠️ {{ errorMessage }}</div>

    <form @submit.prevent="handleSubmitEvent" class="create-form">
      <div class="form-group">
        <label for="title">Event Title *</label>
        <input
          id="title"
          v-model="form.title"
          type="text"
          placeholder="e.g. Modern UI/UX Masterclass"
          :class="{ 'has-error': errors.title }"
          @blur="validateField('title')"
        />
        <span v-if="errors.title" class="field-error">{{ errors.title }}</span>
      </div>

      <div class="form-row">
        <div class="form-group">
          <label for="cat">Event Category *</label>
          <select id="cat" v-model="form.cat">
            <option value="Workshop">Workshop</option>
            <option value="Meetup">Meetup</option>
            <option value="Competition">Competition</option>
            <option value="Seminar">Seminar</option>
            <option value="Conference">Conference</option>
          </select>
        </div>

        <div class="form-group">
          <label for="date">Date & Time *</label>
          <input
            id="date"
            v-model="form.date"
            type="datetime-local"
            :class="{ 'has-error': errors.date }"
            @blur="validateField('date')"
          />
          <span v-if="errors.date" class="field-error">{{ errors.date }}</span>
        </div>
      </div>

      <div class="form-row">
        <div class="form-group">
          <label for="loc">Event Location *</label>
          <input
            id="loc"
            v-model="form.loc"
            type="text"
            placeholder="e.g. Innovation Center, Jakarta"
            :class="{ 'has-error': errors.loc }"
            @blur="validateField('loc')"
          />
          <span v-if="errors.loc" class="field-error">{{ errors.loc }}</span>
        </div>

        <div class="form-group">
          <label for="quota">Attendee Quota *</label>
          <input
            id="quota"
            v-model="form.quota"
            type="number"
            min="1"
            placeholder="100"
            :class="{ 'has-error': errors.quota }"
            @blur="validateField('quota')"
          />
          <span v-if="errors.quota" class="field-error">{{ errors.quota }}</span>
        </div>
      </div>

      <div class="form-group">
        <label for="desc">Event Description *</label>
        <textarea
          id="desc"
          v-model="form.desc"
          rows="4"
          placeholder="Describe the agenda, speakers, and key takeaways..."
          :class="{ 'has-error': errors.desc }"
          @blur="validateField('desc')"
        ></textarea>
        <span v-if="errors.desc" class="field-error">{{ errors.desc }}</span>
      </div>

      <div class="form-actions">
        <AppButton variant="secondary" type="button" @click="emit('cancel')"> Cancel </AppButton>
        <AppButton variant="primary" type="submit" :loading="submitting">
          🚀 Publish Event Now</AppButton
        >
      </div>
    </form>
  </AppCard>
</template>

<style scoped>
.create-event-card {
  padding: var(--space-8);
  max-width: 800px;
  margin: 0 auto;
}

.form-header {
  margin-bottom: var(--space-6);
}

.form-header h3 {
  font-size: 1.6rem;
  color: var(--text-main);
  margin-bottom: var(--space-1);
}

.subtitle {
  color: var(--text-muted);
  font-size: 0.95rem;
}

.error-banner {
  background: #fee2e2;
  color: #dc2626;
  padding: var(--space-3) var(--space-4);
  border-radius: 8px;
  border: 1px solid #fca5a5;
  margin-bottom: var(--space-6);
  font-size: 0.9rem;
  font-weight: 500;
}

.create-form {
  display: flex;
  flex-direction: column;
  gap: var(--space-5);
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--space-4);
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}

.form-group label {
  font-weight: 600;
  font-size: 0.9rem;
  color: var(--text-main);
}

.form-group input,
.form-group select,
.form-group textarea {
  padding: 10px 14px;
  border-radius: 8px;
  border: 1px solid var(--border-color);
  font-size: 0.95rem;
  font-family: inherit;
  outline: none;
  transition:
    border-color 0.2s,
    box-shadow 0.2s;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(102, 68, 255, 0.15);
}

.form-group input.has-error,
.form-group select.has-error,
.form-group textarea.has-error {
  border-color: #dc2626;
}

.field-error {
  color: #dc2626;
  font-size: 0.8rem;
  font-weight: 500;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: var(--space-3);
  margin-top: var(--space-4);
}

@media (max-width: 640px) {
  .form-row {
    grid-template-columns: 1fr;
  }
}
</style>
