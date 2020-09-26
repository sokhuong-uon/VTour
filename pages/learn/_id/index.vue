<template>
<div>
	<HelloCube v-if="$route.params.id == 'hellocube'"/>
	<Responsive v-if="$route.params.id == 'responsive'"/>
</div>
</template>

<script>
export default {

}
</script>

<style>

</style>