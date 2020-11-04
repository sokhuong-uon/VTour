<template>
<div id="sceneSpace" class="relative w-screen h-screen">
	<div class="absolute w-full h-10 flex items-center justify-center text-green-700 text-lg select-none">
		Drag and drop (360x180)degree panorama image
	</div>
	<div class="absolute right-0 bottom-4 w-20 h-10 flex flex-wrap bg-gray-700 items-center justify-center overflow-y-auto">
		<button @click="changePanorama(0)" class="w-6 h-6 mx-1 bg-indigo-600 text-gray-100 text-lg">1</button>
		<button @click="changePanorama(1)" class="w-6 h-6 mx-1 bg-indigo-600 text-gray-100 text-lg">2</button>
		<!--
		<button @click="changePanorama(2)" class="w-6 h-6 mx-1 bg-indigo-600 text-gray-100 text-lg">3</button>
		<button @click="changePanorama(3)" class="w-6 h-6 mx-1 bg-indigo-600 text-gray-100 text-lg">4</button>
		-->
	</div>
</div>
</template>

<script>

import {
	WebGLRenderer, PerspectiveCamera, Scene, AmbientLight,
	Mesh, CylinderBufferGeometry, SphereBufferGeometry, CircleBufferGeometry,
	MeshBasicMaterial, TextureLoader,
	AxesHelper, GridHelper,
	sRGBEncoding, PCFSoftShadowMap,
	Raycaster, Vector2, MathUtils, Vector3, BackSide, DoubleSide
} from 'three';
import gsap from 'gsap';
import Stats from '~/node_modules/three/examples/jsm/libs/stats.module.js';

export default {
	name: 'Equirectangular',
	data() {
		return {
			sceneSpace: null,
			camera: null,
			scene: null,
			renderer: null,
			stats: null,

			sphereMaterial: null,
			panoramaImageSources: [
				'/panoramas/ballroom.jpg',
				'/panoramas/fireplace.jpg',
				'/panoramas/st_fagans_interior.jpg',
				'https://upload.wikimedia.org/wikipedia/commons/e/e4/Siilinj%C3%A4rven_kirkko_360x180_astetta.jpg'
			],

			isUserInteracting: false,
			onPointerDownMouseX: 0,
			onPointerDownMouseY: 0,
			lon: 0,
			onPointerDownLon: 0,
			lat: 0,
			onPointerDownLat: 0,
			phi: 0,
			theta: 0,

		}
	},
	methods: {
		init(){
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
				// this.renderer.outputEncoding = sRGBEncoding;
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
				let fov = 75;
				let aspect = this.sceneSpace.clientWidth / this.sceneSpace.clientHeight;
				const near = 0.1;
				const far = 5000;
				this.camera = new PerspectiveCamera(fov, aspect, near, far);
				this.camera.name = "Camera";
				this.camera.target = new Vector3(0, 0, 0);
				this.camera.position.set(0,0,0);
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

			// Grid Helper
			// {
			// 	let gridHelper = new GridHelper(500, 500, 0xffffff);
			// 	gridHelper.position.set(0, 0, 0)
			// 	this.scene.add(gridHelper);
			// }

			// Sphere
			{
				let geometry = new SphereBufferGeometry(10, 60, 60);
				// invert the geometry on the x-axis so that all of the faces point inward
				geometry.scale(-1, 1, 1);

				let texture = new TextureLoader().load(
					"/panoramas/ballroom.jpg"
				);
				this.sphereMaterial = new MeshBasicMaterial({ map: texture });

				let mesh = new Mesh(geometry, this.sphereMaterial);

				this.scene.add(mesh);

				texture = new TextureLoader().load(
					"/panoramas/fireplace.jpg"
				);
				let sphere = new Mesh(geometry, new MeshBasicMaterial({ map: texture }));
				sphere.position.set(15, 0, 0);
				this.scene.add(sphere);

				// geometry = new SphereBufferGeometry(1,10,10);
				// let material = new MeshBasicMaterial({color: 0x435999});
				// let smallSphere = new Mesh(geometry, material)
				// smallSphere.position.set(5, 0, 0);
				// sphere.add(smallSphere);
			}

			this.sceneSpace.style.touchAction = "none";
			this.sceneSpace.addEventListener("pointerdown", this.onPointerDown, false);

			this.sceneSpace.addEventListener("wheel", this.onDocumentMouseWheel, false);

			// Drag Drop
			{
				this.sceneSpace.addEventListener(
					"dragover",
					(event)=> {
						event.preventDefault();
						event.dataTransfer.dropEffect = "copy";
					},false
				);

				this.sceneSpace.addEventListener(
					"dragenter",
					()=> {
						this.sceneSpace.style.opacity = 0.5;
					},
					false
				);

				this.sceneSpace.addEventListener(
					"dragleave",
					()=> {
						this.sceneSpace.style.opacity = 1;
					},
					false
				);

				this.sceneSpace.addEventListener(
					"drop",
					(event)=> {
						event.preventDefault();

						var reader = new FileReader();
						reader.addEventListener(
							"load",
							(event)=> {
								this.sphereMaterial.map.image.src = event.target.result;
								this.sphereMaterial.map.needsUpdate = true;
							},
							false
						);
						reader.readAsDataURL(event.dataTransfer.files[0]);

						this.sceneSpace.style.opacity = 1;
					},
					false
				);
			}

			this.renderer.setAnimationLoop(this.animate);
		},

		onPointerDown(event){
			// if (event.isPrimary === false) return;

			this.isUserInteracting = true;

			this.onPointerDownMouseX = event.clientX;
			this.onPointerDownMouseY = event.clientY;

			this.onPointerDownLon = this.lon;
			this.onPointerDownLat = this.lat;

			this.sceneSpace.addEventListener("pointermove", this.onPointerMove, false);
			this.sceneSpace.addEventListener("pointerup", this.onPointerUp, false);
		},

		onPointerMove(event) {
			// if (event.isPrimary === false) return;

			this.lon = (this.onPointerDownMouseX - event.clientX) * 0.1 + this.onPointerDownLon;
			this.lat = (event.clientY - this.onPointerDownMouseY) * 0.1 + this.onPointerDownLat;
		},

		onPointerUp(event) {
			// if (event.isPrimary === false) return;

			this.isUserInteracting = false;

			this.sceneSpace.removeEventListener("pointermove", this.onPointerMove, false);
			this.sceneSpace.removeEventListener("pointerup", this.onPointerUp, false);
		},

		onDocumentMouseWheel(event) {

			let fov = this.camera.fov + event.deltaY * 0.05;

			this.camera.fov = MathUtils.clamp(fov, 10, 75);

			this.camera.updateProjectionMatrix();
		},

		changePanorama(index){
			gsap.to(this.camera.position,{
				x: index*15,
				// y: 10* index,
				// z: 10* index + 15,
				ease: "power3",
				duration: 1
			});

		},

		animate() {

			setTimeout( () => {

				this.lat = Math.max(-85, Math.min(85, this.lat));
				this.phi = MathUtils.degToRad(90 - this.lat);
				this.theta = MathUtils.degToRad(this.lon);

				this.camera.target.x = 500 * Math.sin(this.phi) * Math.cos(this.theta);
				this.camera.target.y = 500 * Math.cos(this.phi);
				this.camera.target.z = 500 * Math.sin(this.phi) * Math.sin(this.theta);

				this.camera.lookAt(this.camera.target);

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
