<template>
<div id="sceneSpace" class="w-full h-screen">
	<div class="absolute break-words w-full flex-shrink min-w-0 flex flex-col items-center justify-center text-gray-500 select-none">
		<div class="text-center">
			<h1>Example</h1>
			<p>Reference:
				<a target="_blank" href="https://threejs.org/">
					<span class="text-indigo-500 hover:underline">Three.js</span>
				</a>
				<a target="_blank" href="https://threejsfundamentals.org/threejs/lessons/threejs-fundamentals.html">
					<span class="text-indigo-500 hover:underline">Threejs Fundamentals</span>
				</a>
			</p>
		</div>
	</div>
</div>
</template>

<script>

import {
	Scene, WebGLRenderer, PerspectiveCamera, BoxGeometry,
	Mesh, DirectionalLight, MeshPhongMaterial
} from 'three';
import _ from 'lodash';

export default {
	name: 'VideoTeextured',
	data() {
		return {
			sceneSpace: null,
			camera: null,
			scene: null,
			renderer: null,
			cubes: []
		}
	},
	methods: {
		init() {
			// Container
			this.sceneSpace = document.getElementById('sceneSpace');
			this.sceneSpace.name = "Scene Space";

			// Camera
			{
				const fov = 75;
				const aspect = this.sceneSpace.clientWidth / this.sceneSpace.clientHeight;
				const near = 0.1;
				const far = 100;
				this.camera = new PerspectiveCamera(fov, aspect, near, far);
				this.camera.name = "Main Camera";
				this.camera.position.set(0,0,5);
			}

			// Scene
			{
				this.scene = new Scene();
				this.scene.name  = "Scene";
			}

			// Directional light
			{
				const color = 0xb5edf5;
				const intensity = 1.5;
				const light = new DirectionalLight(color, intensity);
				light.name = "Directional light"
				light.position.set(0, 10, 20);
				this.scene.add(light);
			}

			// WebGL renderer
			{
				this.renderer = new WebGLRenderer({antialias: true});
				this.renderer.name = "Renderer";
				this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
				this.renderer.setPixelRatio(window.devicePixelRatio);
			}

			// Add renderer to the DOM
			this.sceneSpace.appendChild(this.renderer.domElement);

			// Cube
			{
				const boxWidth = 1;
				const boxHeight = 1;
				const boxDepth = 1;
				const geometry = new BoxGeometry(boxWidth, boxHeight, boxDepth);
				const material = new MeshPhongMaterial({color: 0x851141}); //Materila that affected by lights
				const cube = new Mesh(geometry, material);
				cube.name = "Cube";
				this.scene.add(cube);
			}

			this.renderer.setAnimationLoop(this.animate);
		},

		animate(time) {
			this.renderer.render(this.scene, this.camera);
		},

		onWindowResize(){
			let width = this.sceneSpace.clientWidth;
			let height = this.sceneSpace.clientHeight;
			this.renderer.setSize(width, height);
			this.camera.aspect = width / height;
			this.camera.updateProjectionMatrix();
		}
	},

	mounted() {
		this.init();
		window.addEventListener('resize', this.onWindowResize);
	},

	created(){
	}
}
</script>
