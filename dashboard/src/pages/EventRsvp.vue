<template>
  <div v-if="event.doc" class="w-full z-0 min-h-screen">
    <EventHeader
      class="px-4 py-8 md:p-8"
      :event="event"
      :form_exists="rsvp_exists"
      :form="event_rsvp"
    />
    <TabsWithRoute :tabs="tabs.options" />
    <RouterView @rsvp-created="rsvpCreated" :event_rsvp="event_rsvp" />
  </div>
</template>
Show
<script setup>
import EventHeader from '@/components/EventHeader.vue'
import { createDocumentResource, createResource } from 'frappe-ui'
import { useRoute, useRouter } from 'vue-router'
import TabsWithRoute from '@/components/TabsWithRoute.vue'
import { reactive, ref, watch } from 'vue'

const route = useRoute()
const router = useRouter()
const event = createDocumentResource({
  doctype: 'FOSS Chapter Event',
  name: route.params.id,
  fields: ['*'],
  auto: true,
})

let tabs = reactive({
  options: [
    {
      label: 'Overview',
      route: `/event/${route.params.id}/rsvp`,
    },
    {
      label: 'Web Form',
      route: `/event/${route.params.id}/rsvp/create`,
    },
  ],
})

const replaceCreateOption = () => {
  // TODO: simplify this
  tabs.options = tabs.options.filter((d) => d.label !== 'Web Form')

  if (!tabs.options.find((d) => d.label === 'Web Form')) {
    tabs.options.push({
      label: 'Web Form',
      route: `/event/${route.params.id}/rsvp/edit`,
    })
  }

  if (!tabs.options.find((d) => d.label === 'Insights')) {
    tabs.options.push({
      label: 'Insights',
      route: `/event/${route.params.id}/rsvp/insights`,
    })
  }
}

let rsvp_exists = ref(false)
let event_rsvp = reactive({})

watch(event, (newEvent) => {
  event_rsvp = createResource({
    url: 'frappe.client.get',
    params: {
      doctype: 'FOSS Event RSVP',
      filters: {
        event: newEvent.doc.name,
      },
    },
    auto: true,
    onError(error) {
      if (error.response.status === 404) {
        rsvp_exists.value = false
      }
    },
    onSuccess(response) {
      rsvp_exists.value = true
    },
  })
  if (rsvp_exists.value) {
    replaceCreateOption()
  }
})

const rsvpCreated = () => {
  replaceCreateOption()
  router.replace(`/event/${route.params.id}/rsvp/edit`)
}
</script>
