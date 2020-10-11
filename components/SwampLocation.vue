<template>
	<div id="sceneSpace" class="w-screen h-screen">
		<div class="flex justify-center absolute w-screen text-gray-500">
			<div class="text-center">
				<h1>3D Model</h1>
				<p>Author: <a target="_blank" href="https://sketchfab.com/apsnu"><span class="text-indigo-500">bocharova</span></a></p>
				<p>Source: <a target="_blank" href="https://sketchfab.com/models/7ede00c0dfe14574bd8f06ee36020609"><span class="text-indigo-500">swamp location</span></a></p>
				<p>Licenses: <a target="_blank" href="http://creativecommons.org/licenses/by-nc/4.0/"><span class="text-indigo-500">CC-BY-NC-4.0</span></a></p>
			</div>
		</div>
	</div>
</template>


<script>

import {
	PerspectiveCamera, AmbientLight, Scene, WebGLRenderer, Mesh
} from 'three';
import Stats from '~/node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '~/node_modules/three/examples/jsm/loaders/GLTFLoader.js';
import {OrbitControls} from '~/node_modules/three/examples/jsm/controls/OrbitControls.js';

export default {
	name: 'SwampLocation',
	data() {
		return {
			sceneSpace: null,
			camera: null,
			scene: null,
			renderer: null,
			stats: null,
			controls: null,
		}
	},
	methods: {
		init() {
			// Container
			this.sceneSpace = document.getElementById('sceneSpace');
			this.sceneSpace.name = "Scene Space";

			// WebGL Renderer
			{
				this.renderer = new WebGLRenderer({antialias: true});
				this.renderer.name = "Renderer";
				this.renderer.setPixelRatio(window.devicePixelRatio);
				this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
				this.renderer.shadowMap.enabled = true;
				this.sceneSpace.appendChild(this.renderer.domElement);
			}

			// Stats
			{
				this.stats = new Stats();
				this.stats.name = "Stats Bar"
				this.sceneSpace.appendChild(this.stats.domElement);
			}

			// Scene
			{
				this.scene = new Scene();
				this.scene.name  = "Scene";
			}

			// Camera
			{
				const fov = 75;
				const aspect = this.sceneSpace.clientWidth / this.sceneSpace.clientHeight;
				const near = 0.1;
				const far = 1000;
				this.camera = new PerspectiveCamera(fov, aspect, near, far);
				this.camera.name = "Camera";
				this.camera.position.set(0,10,50);
				this.camera.rotation.y = -Math.PI;
			}

			// Light
			{
				const color = 0x777777;
				const intensity = 4;
				const ambientLight = new AmbientLight(color, intensity);
				ambientLight.name = "Ambient Light";
				this.scene.add(ambientLight);
			}

			// Load GLTF
			{
				const loader = new GLTFLoader();

				loader.load(
					'../../gltf/swamp_location/scene.gltf',

					// Call once loaded
					(gltf) => {
						gltf.scene.name = "GLTF Scene";
						this.scene.add(gltf.scene);
					},

					// Call while loading
					(xhr) => {
						console.log((xhr.loaded / xhr.total * 100) + '% loaded');
					},

					// Call when errors
					(error) => {
						console.log('An error happened');
					}
				);
			}

			// Orbit Controls
			{
				this.controls = new OrbitControls(this.camera, this.renderer.domElement);
				this.controls.target.set(0,0,0);
				this.controls.minDistance = 10;
				this.controls.maxDistance = 100;
				this.controls.update();
			}

			this.renderer.setAnimationLoop(this.animate);
		},

		animate() {
			setTimeout( () => {
				this.renderer.render(this.scene, this.camera);
				this.stats.update();
			}, 1000 / 60);	// 60 fps, 1000/30 for 30 fps
		},

		onWindowResize() {
			this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
			this.camera.aspect = this.sceneSpace.clientWidth / this.sceneSpace.clientHeight;
			this.camera.updateProjectionMatrix();
		},
	},

	mounted() {
		this.init();
		window.addEventListener('resize', this.onWindowResize, false);
	},

	created(){
	}
}
</script>
