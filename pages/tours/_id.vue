<template>
<div class="bg-gray-100">
	<SwampLocation v-if="$route.params.id == 'swamplocation'"/>
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
