<template>


<!--  My Projects -->
    <section class="pb-5" id="projects">
        <h1 class="mt-5 mb-5 pb-4 text-center">My Projects</h1>

        <div
          v-for="(group, index) in chunkedProjects"
          :key="index"
          class="row my-5 justify-content-center"
        >
          <ProjectCard
            v-for="project in group"
            :key="project.id"
            :project="project"
            class="col-md-4"
          />
        </div>
    </section>
</template>
<script setup>
    
import { computed } from "vue"; // import computed for dynamic calculated state

import ProjectCard from "./ProjectCard.vue"; // import ProjectCard component
import projects from "../data/projects.json"; // import projects data

const chunkSize = 3; // number of projects per row

// compute projects in chunks of 3
const chunkedProjects = computed(() => {
    const chunks = [] // array to store project groups

    for (let i = 0; i < projects.length; i += chunkSize) {
        chunks.push(projects.slice(i, i + chunkSize)) // take 3 projects at a time
    }

    return chunks // return grouped projects
})
</script>