<template>
	<div id="sceneSpace" class="w-screen h-screen">

	</div>
</template>


<script>

import {
	PerspectiveCamera, AmbientLight, Scene, WebGLRenderer,
	AxesHelper, GridHelper,
	PlaneBufferGeometry, MeshBasicMaterial, Mesh, BoxBufferGeometry, BackSide,
	DirectionalLight, MeshLambertMaterial, TextureLoader
} from 'three';
import Stats from '~/node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '~/node_modules/three/examples/jsm/loaders/GLTFLoader.js';
import {OrbitControls} from '~/node_modules/three/examples/jsm/controls/OrbitControls.js';

export default {
	name: 'Test',
	data() {
		return {
			sceneSpace: null,
			camera: null,
			scene: null,
			renderer: null,
			stats: null,
			controls: null,

			plane: null,
			cubes: []
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
				this.camera.position.set(0.5, 2, 0.5);
				// this.camera.rotation.y = -Math.PI;
			}

			// Light
			{
				const color = 0x777777;
				const intensity = 4;
				const ambientLight = new AmbientLight(color, intensity);
				ambientLight.name = "Ambient Light";
				this.scene.add(ambientLight);
			}

			// Directional Light
			{
				const color = 0x777777;
				const intensity = 1;
				const directionalLight = new DirectionalLight(color, intensity);
				directionalLight.name = "Directional Light";
				directionalLight.position.set(20, 60, 0);
				this.scene.add(directionalLight);
			}

			// Orbit Controls
			{
				this.controls = new OrbitControls(this.camera, this.renderer.domElement);
				this.controls.target.set(0,0.5,0);
				this.controls.minDistance = 0.5;
				this.controls.maxDistance = 100;
				this.controls.update();
			}

			// Axes Helper
			{
				const helper = new AxesHelper(10, 10, 10);
				this.scene.add(helper);
			}

			// Grid Helper
			{
				const size = 50;
				const divisions = 50;
				const gridHelper = new GridHelper( size, divisions );
				this.scene.add( gridHelper );
			}

			// Plane
			// {
			// 	const geometry = new PlaneBufferGeometry(10, 10);
			// 	const material = new MeshBasicMaterial({transparent: true});
			// 	this.plane = new Mesh(geometry, material);
			// 	this.plane.rotateX(-Math.PI / 2);
			// 	this.scene.add(this.plane);
			// }

			// Cube
			{
				const geometry = new BoxBufferGeometry(1, 1, 1);
				const material = new MeshLambertMaterial({color: 0x353535, side: BackSide});
				const loader = new TextureLoader();
				const materials = [
					new MeshBasicMaterial({map: loader.load('https://images.unsplash.com/photo-1495366691023-cc4eadcc2d7e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=4&w=128&h=128&q=100'), side: BackSide}),
					new MeshBasicMaterial({map: loader.load('https://images.unsplash.com/photo-1494790108377-be9c29b29330?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=3&w=128&h=128&q=100'), side: BackSide}),
					new MeshBasicMaterial({map: loader.load('https://images.unsplash.com/photo-1494790108377-be9c29b29330?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=3&w=128&h=128&q=100'), side: BackSide}),
					new MeshBasicMaterial({map: loader.load('https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=3&w=128&h=128&q=100'), side: BackSide}),
					new MeshBasicMaterial({map: loader.load('https://images.unsplash.com/photo-1581803118522-7b72a50f7e9f?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=3&w=128&h=128&q=100'), side: BackSide}),
					new MeshBasicMaterial({map: loader.load('https://images.unsplash.com/photo-1495366691023-cc4eadcc2d7e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=4&w=128&h=128&q=100'), side: BackSide}),
				];
				for(let i=-5.5; i<5; i++){
					for(let j=-5.5; j<5; j++){
						let cube = new Mesh(geometry, materials);
						cube.position.set(i, 0.5, j);
						this.scene.add(cube);
					}
				}
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
