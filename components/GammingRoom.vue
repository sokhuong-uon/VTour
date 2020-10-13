<template>
	<div id="sceneSpace" class="relative w-screen h-screen">
		<div class="flex justify-center absolute w-screen text-gray-500">
			<div class="text-center">
				<h1>3D Model. Takes around 30 seconds to load on netlify</h1>
				<p>Author: <a target="_blank" href="https://sketchfab.com/vboichut"><span class="text-indigo-500">Vincent</span></a></p>
				<p>Source: <a target="_blank" href="https://sketchfab.com/3d-models/gaming-room-47c7c39f767e40eb84d1b1d857af525b"><span class="text-indigo-500">Gaming Room</span></a></p>
				<p>Licenses: <a target="_blank" href="http://creativecommons.org/licenses/by/4.0/"><span class="text-indigo-500">CC-BY-4.0</span></a></p>
			</div>
		</div>
		<button v-if="!inRoom" @click="enterRoom" class="absolute w-18 h-10 bottom-4 right-4 rounded-md bg-gray-700 text-gray-300">Enter</button>
	</div>
</template>


<script>

import {
	PerspectiveCamera, AmbientLight, Scene, WebGLRenderer, Mesh, AxesHelper,
	sRGBEncoding, PCFSoftShadowMap, CircleBufferGeometry, MeshBasicMaterial,
	Raycaster, Vector2, CylinderBufferGeometry, DoubleSide, AnimationMixer,
	Clock
} from 'three';
import gsap from 'gsap';
import Stats from '~/node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '~/node_modules/three/examples/jsm/loaders/GLTFLoader.js';

export default {
	name: 'GamingRoom',
	data() {
		return {
			inRoom: false,
			sceneSpace: null,
			camera: null,
			scene: null,
			renderer: null,
			stats: null,
			raycaster: null,
			circleOverHelper: null,
			targetObjects: [],

			mouse: null,
			mouseDown: null,
			intersect: {
				point:{
					x: 0,
					y: 0,
					z:0
				}
			},

			enterRoomTween: null,
			exitRoomTween: null,
			moveInRoomTween: null,
			mixer: null,
			clock: new Clock()
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
				// this.renderer.setClearColor(0x888888);
				this.renderer.setPixelRatio(window.devicePixelRatio);
				this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
				this.renderer.outputEncoding = sRGBEncoding;
				this.renderer.shadowMap.enabled = true;
				this.renderer.shadowMap.type = PCFSoftShadowMap;
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
				const far = 5000;
				this.camera = new PerspectiveCamera(fov, aspect, near, far);
				this.camera.name = "Camera";
				this.camera.position.set(150,300,600);
			}

			// Light
			{
				const color = 0xffffff;
				const intensity = 1;
				const ambientLight = new AmbientLight(color, intensity);
				ambientLight.name = "Ambient Light";
				this.scene.add(ambientLight);
			}

			// Helper
			{
				let helper = new AxesHelper(50);
				this.scene.add(helper);
			}

			// Circle Helper
			{
				let geometry = new CylinderBufferGeometry(8, 8, 0.02, 60);
				let material = new MeshBasicMaterial({color: 0x353535});
				this.circleOverHelper = new Mesh(geometry, material);
				this.circleOverHelper.rotateX(-Math.PI / 2);
				const outline = new Mesh(
					new CylinderBufferGeometry(6, 6, 0.05, 60),
					new MeshBasicMaterial({color: 0xfffffa, side: DoubleSide})
				);
				this.circleOverHelper.add(outline);
				this.circleOverHelper.rotateX(-Math.PI/2);
				this.scene.add(this.circleOverHelper);
			}


			// Load GLTF
			{
				const loader = new GLTFLoader();

				loader.load(
					'../../gltf/gaming_room/scene.gltf',

					// Call once loaded
					(gltf) => {
						gltf.scene.name = "GLTF Scene";
						this.mixer = new AnimationMixer( gltf.scene );
						let action = this.mixer.clipAction( gltf.animations[ 0 ] );
						action.play();
						gltf.scene.children[0].position.set(1000,-10,300);
						this.targetObjects.push(gltf.scene.children[0].children[0].children[0].children[0].children[0].children[63].children[0]);
						this.scene.add(gltf.scene.children[0]);
					},

					// Call while loading
					(xhr) => {
						console.log((xhr.loaded / xhr.total * 100) + '% loaded');
					},

					// Call when errors
					(error) => {
						console.log('An error happened');
						console.log(error);
					}
				);
			}

			this.moveInRoomTween = gsap.to(this.camera.position, {
				x: this.intersect.point.x,
				y: 110,
				z: this.intersect.point.z,
				ease: "Power3.InOut",
				duration: 2,
				paused: true,
				reversed: true,
			});

			// Raycaster
			this.raycaster = new Raycaster();

			// Mouse
			this.mouse = new Vector2();

			this.renderer.setAnimationLoop(this.animate);
		},

		animate() {
			setTimeout( () => {
				this.renderer.render(this.scene, this.camera);
				let delta = this.clock.getDelta();

				if ( this.mixer ) this.mixer.update( delta );
				this.stats.update();
			}, 1000 / 60);	// 60 fps, 1000/30 for 30 fps
		},

		enterRoom(){
			this.inRoom = true;
			gsap.to(this.camera.position,{
				x: 0,
				y: 110,
				z: 150,
				ease: "power3.InOut",
				duration: 3,
			});
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

		onMouseMove(event) {
			event.preventDefault();
			if(!this.moveInRoomTween.isActive()){
				this.mouse.set(
					(event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
					-(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
				);

				this.raycaster.setFromCamera(this.mouse, this.camera);
				let intersects = this.raycaster.intersectObjects(this.targetObjects);

				if (intersects.length) {
					this.intersect = intersects[0];

					if (this.intersect.object.name == 'SOL1polySurface627_lambert68_0'){
						this.circleOverHelper.position.copy(this.intersect.point).add(this.intersect.face.normal);
					}
				}

			}
		},

		onMouseDown(event) {
			event.preventDefault();
			if(!this.moveInRoomTween.isActive()){
				this.mouse.set(
					(event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
					-(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
				);

				this.raycaster.setFromCamera(this.mouse, this.camera);
				let intersects = this.raycaster.intersectObjects(this.targetObjects);

				if (intersects.length) {
					this.inRoom = true;
					this.intersect = intersects[0];
					console.log(this.intersect.object.name);

					if (this.intersect.object.name == 'SOL1polySurface627_lambert68_0'){
						this.moveInRoomTween.play();
						gsap.to(this.camera.position, {
							x: this.intersect.point.x,
							z: this.intersect.point.z,
							ease: "Power3.InOut",
							duration: 2,
						});
					}
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
