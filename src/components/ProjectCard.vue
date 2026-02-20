<template>
	
	<div class="card col-md-3">
	  <img: src="project.image" class="card-img-top pt-3" alt="..."/>
	  <div class="card-body">
	  	<h5 class="card-title">{{project.title}}</h5>
	    <p class="card-text">{{project.description}}</p>
	  </div>
	</div>

</template>

<script setup>
	import {defineProps} from "vue"

	defineProps({
		project:Object
	})
</script>