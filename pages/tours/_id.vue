<template>
<div class="bg-gray-100">
	<Building v-if="$route.params.id == 'office_building'"/>
	<Interior v-if="$route.params.id == 'office_interior'"/>
	<SwampLocation v-if="$route.params.id == 'swamplocation'"/>
	<Castle v-if="$route.params.id == 'castle'"/>
	<Classroom v-if="$route.params.id == 'classroom'"/>
	<Gym v-if="$route.params.id == 'gym'"/>
	<GammingRoom v-if="$route.params.id == 'gameroom'"/>
	<h1 v-else>Hahahahaha</h1>
</div>
</template>

<script>
export default {
	name: 'Tours'
}
</script>

<style>

</style>
