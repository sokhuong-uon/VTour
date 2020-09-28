<template>
	<div class="relative w-screen h-screen flex items-center justify-center">
		<div id="sceneSpace" class="w-screen h-screen">
			<div class="flex justify-center absolute w-screen text-gray-500">
				<div class="text-center">
					<h1>3D Model</h1>
					<p>Author: <a target="_blank" href="https://sketchfab.com/y2dan"><span class="text-indigo-500">y2dan</span></a></p>
					<p>Source: <a target="_blank" href="https://sketchfab.com/models/7ae0c1028c404669826cde0656829287"><span class="text-indigo-500">Avenue Tennis - Main Gym 2018</span></a></p>
					<p>Licenses: <a target="_blank" href="http://creativecommons.org/licenses/by/4.0/"><span class="text-indigo-500">CC-BY-4.0</span></a></p>
				</div>
			</div>
		</div>
	</div>
</template>

<script>

import {
	PerspectiveCamera, AmbientLight, Scene, WebGLRenderer, Mesh, Raycaster,
	LineBasicMaterial, CircleBufferGeometry, MeshBasicMaterial, Vector2,
	DoubleSide, sRGBEncoding, PCFSoftShadowMap, Group
} from 'three';
import gsap from 'gsap';
import Stats from '~/node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '~/node_modules/three/examples/jsm/loaders/GLTFLoader.js';

export default {
	name: 'Gym',
	data() {
		return {
			sceneSpace: null,
			renderer: null,
			stats: null,
			camera: null,
			scene: null,
			raycaster: null,
			targetObjects: null,
			circleOverHelper: null,
			outLine: null,

			mouse: null,
			mouseDown: null,
		}
	},
	methods: {
		init() {
			// Container
			this.sceneSpace = document.getElementById('sceneSpace');

			// Camera
			{
				const fov = 75;
				const aspect = this.sceneSpace.clientWidth / this.sceneSpace.clientHeight;
				const near = 0.1;
				const far = 70000;
				this.camera = new PerspectiveCamera(fov, aspect, near, far);
				this.camera.name = "Camera";
				this.camera.position.set(0,0,500);
				this.camera.rotateY(-Math.PI / 2);
			}

			// Scene
			{
				this.scene = new Scene();
				this.scene.name  = "Scene";
			}

			// WebGL Renderer
			{
				this.renderer = new WebGLRenderer({antialias: true});
				this.renderer.name = "Renderer";
				this.renderer.setPixelRatio(this.sceneSpace.devicePixelRatio);
				this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
				this.renderer.outputEncoding = sRGBEncoding;
				this.renderer.shadowMap.enabled = true;
				this.renderer.shadowMap.type = PCFSoftShadowMap;
				this.sceneSpace.appendChild(this.renderer.domElement);
			}

			// Stats
			{
				this.stats = new Stats();
				this.stats.name = "Stats";
				this.sceneSpace.appendChild(this.stats.domElement);
			}

			// Light
			{
				const color = 0x555555;
				const intensity = 2;
				const ambientLight = new AmbientLight(color, intensity);
				this.scene.add(ambientLight);
			}

			// Load GLTF
			{
				const loader = new GLTFLoader();

				loader.load(
					'../../gltf/avenue_tennis_-_main_gym_2018/scene.gltf',

					// Call once loaded
					( gltf )=> {
						gltf.scene.name = "GLTF Scene";
						gltf.scene.position.y -= 1000;
						gltf.scene.position.z += 6500;
						this.scene.add(gltf.scene);

						this.targetObjects = gltf.scene.children[0].children[0].children[0].children[0].children;
						// console.log(this.targetObjects);
						// console.log(this.mesh.children[0].children[0]); // Gym
						// console.log(this.mesh.children[0].children[0].children[0]); // Models
						// console.log(this.mesh.children[0].children[0].children[0].children[15]); // red floor
						// console.log(this.mesh.children[0].children[0].children[0].children[16]); // white floor

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

			// Circle roller
			{
				const circleRollerGeo = new CircleBufferGeometry(1, 100,);
				const circleRollerMaterial = new LineBasicMaterial({
					color: 0xffffff,
					opacity: .8,
					transparent: false
				});

				this.circleOverHelper = new Mesh(circleRollerGeo, circleRollerMaterial);
				this.circleOverHelper.name = "Cricle Helper"
				this.circleOverHelper.rotateX(Math.PI / 2);

				this.scene.add(this.circleOverHelper);

				const outLineMaterial = new MeshBasicMaterial({
					color: 0xffffff,
					opacity: 1,
					side: DoubleSide
				});

				this.outLine = new Mesh(circleRollerGeo, outLineMaterial);
				this.outLine.position = this.circleOverHelper.position;
				this.outLine.scale.multiplyScalar(.7);
				this.circleOverHelper.add(this.outLine);
			}

			// Raycaster
			this.raycaster = new Raycaster();

			// Mouse
			this.mouse = new Vector2();

			this.renderer.setAnimationLoop(this.animate);

		},

		animate() {
			setTimeout( () => {
				this.renderer.render(this.scene, this.camera);
				this.stats.update();
			}, 1000 / 60);	// 60 fps, 1000/30 for 30 fps
		},

		initControls() {
			this.camera.rotation.order = "YXZ";

			// Mouse down
			this.sceneSpace.addEventListener("mousedown", (evt) => {
				this.mouseDown = true;
				this.sceneSpace.style.cursor = "grab";
			}, false);

			// Mouse up
			this.sceneSpace.addEventListener("mouseup", (evt) => {
				this.sceneSpace.style.cursor = "default";
				this.mouseDown = false;
			}, false);

			this.sceneSpace.addEventListener("mousemove", (evt) => {
				let movementX = evt.movementX || evt.mozMovementX || evt.webkitMovementX || 0;
				let movementY = evt.movementY || evt.mozMovementY || evt.webkitMovementY || 0;

				if (this.mouseDown) {
					this.sceneSpace.style.cursor = "grabbing";
					this.camera.rotation.y -= -movementX / 600;
					this.camera.rotation.x -= -movementY / 600;
				}
			},false);
		},

		onMouseMove(event){
			event.preventDefault();
			this.mouse.set(
				(event.clientX / this.renderer.domElement.innerWidth) * 2 - 1,
				-(event.clientY / this.renderer.domElement.innerHeight) * 2 + 1
			);

			this.raycaster.setFromCamera(this.mouse, this.camera);
			let intersects = this.raycaster.intersectObjects(this.targetObjects);

			if(intersects.length){
				let intersect = intersects[0];
				this.circleOverHelper.position = intersect.point
			}
		},

		onMouseDown(event){
			event.preventDefault();
			this.mouse.set(
				(event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
				-(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
			);

			this.raycaster.setFromCamera(this.mouse, this.camera);
			let intersects = this.raycaster.intersectObjects(this.targetObjects);

			if(intersects.length){
				let intersect = intersects[0];
				// console.log(intersect.object.name);

				if(intersect.object.name === 'red_floor' || intersect.object.name === 'white_floor'){
					// here is the magic 🤩🤩
					// Move camera 🎉🎉🎉🎉🎉🎉🎉
					gsap.to(this.camera.position, {
						x: intersect.point.x,
						z: intersect.point.z,
						ease: "Power1.easeInOut",
						onUpdate: () => {
							this.camera.updateProjectionMatrix();
						},
						onComplete: () => {
							console.log('animation ends');
						}
					}).duration(2);
				}
			}
		},

		onWindowResize() {
			this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
			this.camera.aspect = this.sceneSpace.clientWidth / this.sceneSpace.clientHeight;
			this.camera.updateProjectionMatrix();
		},
	},

	mounted() {
		this.init();
		this.initControls();
		this.sceneSpace.addEventListener("mousemove", this.onMouseMove, false);
		this.sceneSpace.addEventListener("mousedown", this.onMouseDown, false);
		window.addEventListener('resize', this.onWindowResize, false);
	},

	created(){
	}
}
</script>
