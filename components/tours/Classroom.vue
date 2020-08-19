<template>
	<div id="sceneSpace" class="w-screen h-screen">
		<div class="flex justify-center absolute w-screen text-gray-500">
			<div class="text-center">
				<h1>3D Model</h1>
				<p>Author: <a target="_blank" href="https://sketchfab.com/GlobalDigitalHeritage"><span class="text-indigo-500">Global Digital Heritage</span></a></p>
				<p>Source: <a target="_blank" href="https://sketchfab.com/3d-models/montemor-o-novos-castle-c3c42b291c9a42e5882cacd3330c04ef"><span class="text-indigo-500">Montemor-o-Novo's Castle</span></a></p>
			</div>
		</div>
	</div>
</template>


<script>
import * as THREE from 'three';
import gsap from 'gsap';
import Stats from '../../node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '../../node_modules/three/examples/jsm/loaders/GLTFLoader.js';
import {DRACOLoader} from '../../node_modules/three/examples/jsm/loaders/DRACOLoader.js';

export default {
	name: 'Classroom',
	data() {
		return {
			camera: null,
			scene: null,
			renderer: null,
			controls: null,
			mesh: null,

			mouse: null,
			mouseDown: null,
			raycaster: null,
			isShifDown: false,
			stats: null,

			rollOverMesh: null,
			rollOverGeo: null,
			rollOverMaterial: null,

			outLineMesh1: null,
			outLineMaterial1: null,


			plane: null,
			// objects for intersection
			// objects: [],
			intersects: null
		}
	},
	computed: {

	},
	methods: {
		init: function() {
			const sceneSpace = document.getElementById('sceneSpace');

			this.camera = new THREE.PerspectiveCamera(75, sceneSpace.clientWidth / sceneSpace.clientHeight, 0.1, 1000);
			this.camera.position.set(0,10,10);
			this.camera.rotateX(10);
			this.camera.lookAt(0,0,0);

			this.scene = new THREE.Scene();

			const ambient = new THREE.AmbientLight(0x555555, 2);
    		this.scene.add(ambient);


			// Instantiate a loader
			const loader = new GLTFLoader();

			// Load a glTF resource
			loader.load(
				// resource URL
				'../../gltf/lowpoly_stylized_classroom/scene.gltf',

				// called when the resource is loaded
				( gltf )=> {
					// console.log(gltf);
					this.scene.add(gltf.scene);
					this.mesh = gltf.scene.children[0];
					// this.plane = gltf.scene.children[0];
					// this.objects.push(this.plane);
					// this.mesh.position.y += 140;
					// this.mesh.rotation.z = -Math.PI;
					// console.log(this.mesh);

				},

				// called while loading is progressing
				function ( xhr ) {
					console.log( ( xhr.loaded / xhr.total * 100 ) + '% loaded' );
				},

				// called when loading has errors
				function ( error ) {
					console.log( 'An error happened' );
				}
			);

			// roll-over circle
			let rollOverGeo = new THREE.CircleBufferGeometry(
				// radius: float
				1,
				// segments: integer
				50,
				// thetaStart: float,
				// thetaLength: float
			);
			let rollOverMaterial = new THREE.LineBasicMaterial({
				color: 0xffffff,
				opacity: .6,
				transparent: true
			});

			this.rollOverMesh = new THREE.Mesh(
				// geometry
				rollOverGeo,
				// material
				rollOverMaterial
			);
			this.rollOverMesh.rotateX(-Math.PI / 2);

			this.scene.add(this.rollOverMesh);

			let outLineMaterial1 = new THREE.LineBasicMaterial({
				color: 0xffffff,
				opacity: .5,
				side: THREE.DoubleSide
			});

			this.outLineMesh1 = new THREE.Mesh(rollOverGeo, outLineMaterial1);
			this.outLineMesh1.position = this.rollOverMesh.position;
			this.outLineMesh1.scale.multiplyScalar(.7);
			this.rollOverMesh.add(this.outLineMesh1);

			// raycaster
			this.raycaster = new THREE.Raycaster();
			this.mouse = new THREE.Vector2();

			this.renderer = new THREE.WebGLRenderer({antialias: true});
			this.renderer.outputEncoding = THREE.sRGBEncoding;
			this.renderer.setSize(sceneSpace.clientWidth, sceneSpace.clientHeight);
			this.renderer.shadowMap.enabled = true;
			this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
			this.renderer.setPixelRatio(sceneSpace.devicePixelRatio);

			sceneSpace.appendChild(this.renderer.domElement);

			this.stats = new Stats();
			sceneSpace.appendChild( this.stats.domElement );

			this.initControls();

			this.renderer.setAnimationLoop(() => {
				this.renderer.render(this.scene, this.camera);
				this.stats.update();
			});
		},

		animate: function() {
			requestAnimationFrame(this.animate);
			this.renderer.render(this.scene, this.camera);
			this.stats.update();
		},

		onWindowResize: function(){
			this.renderer.setSize(sceneSpace.clientWidth, sceneSpace.clientHeight);
			this.camera.aspect = sceneSpace.clientWidth / sceneSpace.clientHeight;
			this.camera.updateProjectionMatrix();
		},

		initControls: function() {
			this.camera.rotation.order = "YXZ";

			// mouse down event
			document.addEventListener("mousedown", (evt)=>{
				this.mouseDown = true;
				document.getElementsByTagName("body")[0].style.cursor = "grab";
			});

			// mouse up event
			document.addEventListener("mouseup", (evt)=>{
				document.getElementsByTagName("body")[0].style.cursor = "default";
				this.mouseDown = false;
			});

			document.addEventListener(
				"mousemove",
				(evt)=>{
					let movementX = evt.movementX || evt.mozMovementX || evt.webkitMovementX || 0;
					let movementY = evt.movementY || evt.mozMovementY || evt.webkitMovementY || 0;

					if (this.mouseDown) {
						document.getElementsByTagName("body")[0].style.cursor = "grabbing";

						this.camera.rotation.y -= -movementX / 600;
						this.camera.rotation.x -= -movementY / 600;
					}
				},
				false
			);
		},

		// on mouse move
		onDocumentMouseMove: function(event){
			event.preventDefault();
			let objects=[];
			objects.push(this.mesh.children[0].children[0].children[53]);
			this.mouse.set(
				// X (this is the way we get x position)
				(event.clientX / window.innerWidth) * 2 - 1,
				// Y (this is the way we get y position)
				-(event.clientY / window.innerHeight) * 2 + 1
			);

			this.raycaster.setFromCamera(this.mouse, this.camera);

			// ray will intersect with objects added to array named objects
			// let intersects = raycaster.intersectObjects(objects);
			this.intersects = this.raycaster.intersectObjects(objects);
			// console.log("Objects: ",objects[0].children[0]);
			console.log("Intersects", this.intersects);
			/*
			the picking ray will intersects with one or more of all objects in the scene
			let intersects = raycaster.intersectObjects(scene.children);
			*/

			if(this.intersects.length > 0){
				let intersect = this.intersects[0];

				console.log(intersect.point);
				this.rollOverMesh.position.copy(intersect.point).add(intersect.face.normal);
			}

		},

		// mouse down
		onDocumentMouseDown: function(event){
			event.preventDefault();
			let objects=[];
			objects.push(this.mesh.children[0].children[0].children[53]);
			this.mouse.set(
				(event.clientX / window.innerWidth) * 2 - 1,
				-(event.clientY / window.innerHeight) * 2 + 1
			);

			this.raycaster.setFromCamera(this.mouse, this.camera);

			this.intersects = this.raycaster.intersectObjects(objects);

			if(this.intersects.length > 0){
				let intersect = this.intersects[0];

				// here is the magic 🤩🤩
				// Move camera 🎉🎉🎉🎉🎉🎉🎉
				gsap.to(this.camera.position, 2, {
					x: intersect.point.x,
					z: intersect.point.z,
					ease: "Power1.easeInOut",
					onUpdate: () => {
						this.camera.updateProjectionMatrix();
					},
					onComplete: () =>{
						console.log('animatin ends');
					}
				});

			}
		}

	},

	mounted() {
		this.init();
		document.addEventListener("mousemove", this.onDocumentMouseMove, false);
    	document.addEventListener("mousedown", this.onDocumentMouseDown, false);
		window.addEventListener('resize', () => {this.onWindowResize()}, false);
	},

	created(){
	}
}
</script>

<style lang="css">
*{
	margin: 0;
	padding: 0;
}
</style>
