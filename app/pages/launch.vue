<template>
    <v-container>
        <h1>SpaceX Launches</h1>

        <!-- Year Filter -->
        <v-row class="mb-4">
            <v-col cols="12" md="4">
                <v-select
                    v-model="selectedYear"
                    :items="availableYears"
                    label="Filter by Year"
                    clearable
                    variant="outlined"
                    prepend-inner-icon="mdi-calendar"
                ></v-select>
            </v-col>
            <v-col cols="12" md="8" class="d-flex align-center">
                <v-chip color="info">
                    Showing {{ filteredLaunches.length }} of {{ launches.length }} launches
                </v-chip>
            </v-col>
        </v-row>

        <v-row>
            <v-col v-for="launch in filteredLaunches" :key="launch.id" cols="12" md="6" lg="4">
                <v-card class="mb-4" variant="elevated" height="100%">
                    <v-card-title class="text-h6">{{ launch.mission_name }}</v-card-title>
                    <v-card-subtitle class="text-body-2">
                        {{ new Date(launch.launch_date_utc).toLocaleDateString() }} • 
                        {{ launch.rocket?.rocket_name }}
                    </v-card-subtitle>
                    <v-card-text>
                        <div class="mb-2">
                            <strong>Launch Site:</strong> {{ launch.launch_site?.site_name_long }}
                        </div>
                        <div class="mb-2">
                            <strong>Rocket:</strong> {{ launch.rocket?.rocket_name }}
                        </div>
                        <div class="mb-2">
                            <strong>Launch Date:</strong> {{ new Date(launch.launch_date_utc).toLocaleString() }}
                        </div>
                        <div v-if="launch.details">
                            <strong>Details:</strong> 
                            <p class="mt-1 text-body-2">{{ launch.details }}</p>
                        </div>
                        <div v-else class="text-body-2 font-italic text-grey">
                            No details provided.
                        </div>
                    </v-card-text>
                    <v-card-actions>
                    </v-card-actions>
                </v-card>
            </v-col>
            <v-col v-if="pending" cols="12" class="text-center">
                <v-progress-circular indeterminate color="primary"></v-progress-circular>
                <p class="mt-2">Loading launches...</p>
            </v-col>
            <v-col v-if="error" cols="12">
                <v-alert type="error" :text="error.message"></v-alert>
            </v-col>
            <v-col v-if="!pending && !error && filteredLaunches.length === 0" cols="12" class="text-center">
                <v-alert type="info" text="No launches found for the selected year."></v-alert>
            </v-col>
        </v-row>
    </v-container>
</template>

<script setup lang="ts">
import gql from 'graphql-tag'
import { computed, ref } from 'vue'

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

const { data, pending, error } = await useAsyncQuery<{
    launches: Launch[]
}>({
    query: gql`
        query GetLaunches {
            launches(limit: 100) {
                id
                mission_name
                launch_date_utc
                rocket {
                    rocket_name
                    rocket {
                        id
                    }
                }
                launch_site {
                    site_name_long
                }
                details
            }
        }
    `,
})

const launches = computed(() => data.value?.launches ?? [])

const selectedYear = ref<string | null>(null)

const availableYears = computed(() => {
    const years = new Set<string>()
    launches.value.forEach(launch => {
        const year = new Date(launch.launch_date_utc).getFullYear().toString()
        years.add(year)
    })
    
    return Array.from(years).sort((a, b) => b.localeCompare(a)) // Sort newest first
})

const filteredLaunches = computed(() => {
    if (!selectedYear.value) {
        return launches.value
    }
    
    return launches.value.filter(launch => {
        const launchYear = new Date(launch.launch_date_utc).getFullYear().toString()
        return launchYear === selectedYear.value
    })
})
</script>