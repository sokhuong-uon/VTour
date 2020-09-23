<template>
<div id="sceneSpace" class="w-screen h-screen">
	<div class="absolute flex justify-center w-screen text-gray-500">
		<div class="text-center">
			<h1>Example</h1>
			<p>Source: <a target="_blank" href="https://threejsfundamentals.org/threejs/lessons/threejs-fundamentals.html"><span class="text-indigo-500">Threejs Fundamentals</span></a></p>
		</div>
	</div>
</div>
</template>

<script>
import * as THREE from 'three';

export default {
	name: 'HelloCube',
	data() {
		return {
			camera: null,
			scene: null,
			renderer: null,
			mesh: null,
		}
	},
	methods: {
		init: function() {
			const sceneSpace = document.getElementById('sceneSpace');
			this.camera = new THREE.PerspectiveCamera(75, sceneSpace.clientWidth / sceneSpace.clientHeight, 0.1, 1000);
			this.camera.position.set(0,0,5);

			this.scene = new THREE.Scene();

			const light = new THREE.AmbientLight();
			// this.scene.add(light)

			this.renderer = new THREE.WebGLRenderer({antialias: true});
			this.renderer.setSize(sceneSpace.clientWidth, sceneSpace.clientHeight);
			this.renderer.setPixelRatio(sceneSpace.devicePixelRatio);

			sceneSpace.appendChild(this.renderer.domElement);

			const boxWidth = 1;
			const boxHeight = 1;
			const boxDepth = 1;
			const geometry = new THREE.BoxGeometry(boxWidth, boxHeight, boxDepth);

			const material = new THREE.MeshBasicMaterial({color: 0x44aa88});  // greenish blue

			const cube = new THREE.Mesh(geometry, material);
			this.scene.add(cube);
		},

		animate: function() {
			requestAnimationFrame(this.animate);
			this.renderer.render(this.scene, this.camera);
		},

		onWindowResize(){
			this.renderer.setSize(sceneSpace.clientWidth, sceneSpace.clientHeight);
			this.camera.aspect = sceneSpace.clientWidth / sceneSpace.clientHeight;
			this.camera.updateProjectionMatrix();
		}

	},

	mounted() {
		this.init();
		this.animate();
		window.addEventListener('resize', () => {this.onWindowResize()}, false);
	},

	created(){
	}
}
</script>

<style>

</style>