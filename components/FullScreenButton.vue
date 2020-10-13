<template>
<div class="w-full h-full flex items-center justify-center" @mouseover="hovering = true" @mouseleave="hovering = false">
	<svg v-if="hovering" width="24" height="24" fill="none" xmlns="http://www.w3.org/2000/svg">
		<path d="M20 20h-4M4 8V4v4zm0-4h4-4zm0 0l5 5-5-5zm16 4V4v4zm0-4h-4 4zm0 0l-5 5 5-5zM4 16v4-4zm0 4h4-4zm0 0l5-5-5 5zm16 0l-5-5 5 5zm0 0v-4 4z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
	</svg>
	<svg v-if="!hovering" width="20" height="20" fill="none" xmlns="http://www.w3.org/2000/svg">
		<path fill-rule="evenodd" clip-rule="evenodd" d="M6.414 5l2.293 2.293a1 1 0 01-1.414 1.414L5 6.414V8a1 1 0 01-2 0V4a.997.997 0 011-1h4a1 1 0 010 2H6.414zM12 5a1 1 0 110-2h4a.998.998 0 01.705.29l.004.005a.997.997 0 01.291.702V8a1 1 0 11-2 0V6.414l-2.293 2.293a1 1 0 11-1.414-1.414L13.586 5H12zm-9 7a1 1 0 112 0v1.586l2.293-2.293a1 1 0 011.414 1.414L6.414 15H8a1 1 0 110 2H3.997a.996.996 0 01-.702-.29l-.004-.005A.997.997 0 013 16v-4z" fill="currentColor"/>
		<path d="M12 17a1 1 0 110-2h1.586l-2.293-2.293a1 1 0 011.414-1.414L15 13.586V12a1 1 0 112 0v4.003a.996.996 0 01-.29.702l-.005.004A.997.997 0 0116 17h-4z"
		fill="currentColor"/>
	</svg>
</div>
</template>

<script>
export default {
	name: 'FullScreen',
	data() {
		return{
			hovering: false
		}
	},
}
</script>
