<template>
    <v-container>
        <h1>SpaceX Launches</h1>
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
            <v-col cols="12" md="4">
                <v-select
                    v-model="sortOrder"
                    :items="sortOptions"
                    label="Sort by Launch Date"
                    variant="outlined"
                    prepend-inner-icon="mdi-sort"
                ></v-select>
            </v-col>
            <v-col cols="12" md="4" class="d-flex align-center">
                <v-chip color="info">
                    Showing {{ startIndex + 1 }}-{{ endIndex }} of {{ sortedAndFilteredLaunches.length }} launches
                </v-chip>
            </v-col>
        </v-row>

        <v-row v-if="pending">
            <v-col cols="12" class="text-center">
                <v-progress-circular indeterminate color="primary"></v-progress-circular>
                <p class="mt-2">Loading launches...</p>
            </v-col>
        </v-row>

        <v-row v-else-if="error">
            <v-col cols="12">
                <v-alert type="error" :text="error.message"></v-alert>
            </v-col>
        </v-row>

        <v-row v-else>
            <template v-if="paginatedLaunches.length > 0">
                <v-col v-for="launch in paginatedLaunches" :key="launch.id" cols="12" md="6" lg="4">
                    <LaunchCard :launch="launch" />
                </v-col>
            </template>
            <v-col v-else cols="12" class="text-center">
                <v-alert type="info" text="No launches found for the selected criteria."></v-alert>
            </v-col>
        </v-row>

        <v-row class="mt-4" v-if="totalPages > 1">
            <v-col cols="12" class="d-flex justify-center">
                <v-pagination
                    v-model="currentPage"
                    :length="totalPages"
                    :total-visible="7"
                    color="primary"
                    variant="elevated"
                ></v-pagination>
            </v-col>
        </v-row>
    </v-container>
</template>

<script setup lang="ts">
import gql from 'graphql-tag'
import { computed, ref, watch } from 'vue'
import LaunchCard from '~~/components/LaunchCard.vue'

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

const { data, pending, error } = await useAsyncQuery<{
    launches: Launch[]
}>({
    query: gql`
        query GetLaunches {
            launches {
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
                    site_name
                }
                details
            }
        }
    `,
})

const launches = computed(() => data.value?.launches ?? [])

const selectedYear = ref<string | null>(null)
const sortOrder = ref<string>('newest')

const currentPage = ref(1)
const itemsPerPage = 12

const sortOptions = [
    { title: 'Newest First', value: 'newest' },
    { title: 'Oldest First', value: 'oldest' }
]

const availableYears = computed(() => {
    const years = new Set<string>()
    launches.value.forEach(launch => {
        const year = new Date(launch.launch_date_utc).getFullYear().toString()
        years.add(year)
    })
    
    return Array.from(years).sort((a, b) => b.localeCompare(a))
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

const sortedAndFilteredLaunches = computed(() => {
    const launchesToSort = [...filteredLaunches.value]
    
    return launchesToSort.sort((a, b) => {
        const dateA = new Date(a.launch_date_utc).getTime()
        const dateB = new Date(b.launch_date_utc).getTime()
        
        if (sortOrder.value === 'newest') {
            return dateB - dateA
        } else {
            return dateA - dateB
        }
    })
})

const totalPages = computed(() => {
    return Math.ceil(sortedAndFilteredLaunches.value.length / itemsPerPage)
})

const startIndex = computed(() => {
    return (currentPage.value - 1) * itemsPerPage
})

const endIndex = computed(() => {
    return Math.min(startIndex.value + itemsPerPage, sortedAndFilteredLaunches.value.length)
})

const paginatedLaunches = computed(() => {
    return sortedAndFilteredLaunches.value.slice(startIndex.value, endIndex.value)
})

watch([selectedYear, sortOrder], () => {
    currentPage.value = 1
})
</script>