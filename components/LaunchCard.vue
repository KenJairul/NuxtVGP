<template>
    <v-card class="mb-4" variant="elevated" height="100%">
        <v-card-title class="text-h6">{{ launch.mission_name }}</v-card-title>
        <v-card-text>
            <div class="mb-2">
                <strong>Launch Date:</strong>
                {{ formattedDate }}
            </div>
            <div class="mb-2">
                <strong>Launch Site:</strong>
                {{ launchSiteName }}
            </div>
            <div class="mb-2">
                <strong>Rocket Name:</strong>
                {{ launch.rocket?.rocket_name }}
            </div>
            <div v-if="launch.details" class="mt-3">
                <strong>Details:</strong>
                <p class="mt-1 text-body-2">{{ launch.details }}</p>
            </div>
            <div v-else class="mt-3 text-body-2 font-italic text-grey">
                No details available
            </div>
        </v-card-text>
    </v-card>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Launch {
    id: string
    mission_name: string
    launch_date_utc: string
    rocket: {
        rocket_name: string
        rocket?: { id: string }
    }
    launch_site?: {
        site_name_long?: string
        site_name?: string
    } | null
    details?: string | null
}

const props = defineProps<{ launch: Launch }>()

const formattedDate = computed(() => new Date(props.launch.launch_date_utc).toLocaleString())

const launchSiteName = computed(() => {
    if (!props.launch.launch_site) {
        return 'Launch site not available'
    }

    return (
        props.launch.launch_site.site_name_long ??
        props.launch.launch_site.site_name ??
        'Launch site not available'
    )
})
</script>