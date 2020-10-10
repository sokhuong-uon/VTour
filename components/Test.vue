<template>
	<div id="sceneSpace" class="w-screen h-screen">
		<div class="absolute right-4 top-4 text-gray-200">
			<button @click="juphBetweenCube(0)" class="w-10 h-8 bg-gray-700">0</button>
			<button @click="juphBetweenCube(1)" class="w-10 h-8 bg-gray-700">1</button>
		</div>
	</div>
</template>


<script>

import {
	PerspectiveCamera, AmbientLight, Scene, WebGLRenderer,
	AxesHelper, GridHelper, Raycaster, Vector2,
	PlaneBufferGeometry, MeshBasicMaterial, Mesh, BoxBufferGeometry, BackSide, DoubleSide,
	DirectionalLight, MeshLambertMaterial, TextureLoader, CubeTextureLoader, CircleBufferGeometry,
	CylinderBufferGeometry, SphereBufferGeometry
} from 'three';
import gsap from 'gsap';
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
			circleOverHelper: null,
			raycaster: null,
			mouse: null,
			targetObjects: [],

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
				this.camera.position.set(0, 4, 4);
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
			// {
			// 	this.controls = new OrbitControls(this.camera, this.renderer.domElement);
			// 	this.controls.target.set(0,0,0);
			// 	this.controls.minDistance = 0.5;
			// 	this.controls.maxDistance = 100;
			// 	this.controls.update();
			// }

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

			// Circle Helper
			// {
			// 	let geometry = new CylinderBufferGeometry(0.3, 0.3, 0.01, 60);
			// 	let material = new MeshBasicMaterial({color: 0x353535});
			// 	this.circleOverHelper = new Mesh(geometry, material);
			// 	this.circleOverHelper.rotateX(-Math.PI / 2);
			// 	const outline = new Mesh(
			// 		new CylinderBufferGeometry(0.2, 0.2, 0.02, 60),
			// 		new MeshBasicMaterial({color: 0xfffffa, side: DoubleSide})
			// 	);
			// 	// outline.rotateX(-Math.PI/2);
			// 	this.circleOverHelper.add(outline);
			// 	this.circleOverHelper.rotateX(-Math.PI/2);
			// 	this.scene.add(this.circleOverHelper);
			// }

			// Plane
			{
				const geometry = new PlaneBufferGeometry(10, 10);
				const material = new MeshBasicMaterial({color: 0x233568, opacity: 0.001, transparent: true});
				this.plane = new Mesh(geometry, material);
				this.plane.name = "Plane";
				this.plane.rotateX(-Math.PI / 2);
				this.plane.position.set(0,0.0000001,0)
				this.scene.add(this.plane);
			}

			// Cube
			{
				const geometry = new BoxBufferGeometry(2, 2, 2);
				// const geometry = new SphereBufferGeometry(2, 64, 64);
				// const material = new MeshLambertMaterial({color: 0x353535, side: BackSide});
				// const loader = new TextureLoader();
				// const materials = [
				// 	new MeshBasicMaterial({map: loader.load('/cubemaps/group1/px.png'), side: BackSide}),
				// 	new MeshBasicMaterial({map: loader.load('/cubemaps/group1/nx.png'), side: BackSide}),
				// 	new MeshBasicMaterial({map: loader.load('/cubemaps/group1/py.png'), side: BackSide}),
				// 	new MeshBasicMaterial({map: loader.load('/cubemaps/group1/ny.png'), side: BackSide}),
				// 	new MeshBasicMaterial({map: loader.load('/cubemaps/group1/pz.png'), side: BackSide}),
				// 	new MeshBasicMaterial({map: loader.load('/cubemaps/group1/nz.png'), side: BackSide}),
				// ];
				const loader = new CubeTextureLoader();
				loader.setPath( '/cubemaps/group1/' );

				let textureCube = loader.load( [
					'px.png',
					'nx.png',
					'py.png',
					'ny.png',
					'pz.png',
					'nz.png'
				] );

				let material = new MeshBasicMaterial( { color: 0xffffff, envMap: textureCube, side: BackSide} );
				// for(let i=-4; i<=4; i+=2){
				// 	for(let j=-4; j<=4; j+=2){
				// 		let cube = new Mesh(geometry, materials);
				// 		cube.position.set(i, 1, j);
				// 		this.scene.add(cube);
				// 		this.targetObjects.push(cube);
				// 	}
				// }
				let cube = new Mesh(geometry, material);
				cube.position.set(0, 1, 0);
				this.scene.add(cube);
				this.targetObjects.push(cube);
				this.renderer.background = material;

			}

			this.targetObjects.push(this.plane);


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

		juphBetweenCube(cubeID){
			gsap.to(this.camera.position,{
				x: 0,
				y: 1,
				z: cubeID,
				ease: "Power3.InOut",
				duration: 2,
			})
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
		window.addEventListener('resize', this.onWindowResize, false);
	},

	created(){
	}
}
</script>
