<template>
	<v-card class="mb-4" variant="elevated" height="100%">
		<v-card-title class="text-h6">{{ launch.mission_name }}</v-card-title>
		<v-card-text>
			<div class="mb-2">
				<strong>Mission Name:</strong>
				{{ launch.mission_name }}
			</div>
			<div class="mb-2">
				<strong>Launch Date:</strong>
				{{ formattedDate }}
			</div>
			<div class="mb-2">
				<strong>Launch Site:</strong>
				{{ launch.launch_site?.site_name_long }}
			</div>
			<div class="mb-2">
				<strong>Rocket Name:</strong>
				{{ launch.rocket?.rocket_name }}
			</div>
			<div v-if="launch.details">
				<strong>Details:</strong>
				<p class="mt-1 text-body-2">{{ launch.details }}</p>
			</div>
			<div v-else class="text-body-2 font-italic text-grey">No details available.</div>
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
	launch_site: { site_name_long: string }
	details?: string | null
}

const props = defineProps<{ launch: Launch }>()
const formattedDate = computed(() => new Date(props.launch.launch_date_utc).toLocaleString())
</script>
