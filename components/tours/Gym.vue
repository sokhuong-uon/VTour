<template>
	<div id="sceneSpace" class="w-screen h-screen">
		<div class="flex justify-center absolute w-screen text-gray-500">
			<div class="text-center">
				<h1>3D Model</h1>
				<p>Author: <a target="_blank" href="https://sketchfab.com/y2dan"><span class="text-indigo-500">y2dan</span></a></p>
				<p>Source: <a target="_blank" href="https://sketchfab.com/models/7ae0c1028c404669826cde0656829287"><span class="text-indigo-500">Avenue Tennis - Main Gym 2018</span></a></p>
			</div>
		</div>
	</div>
</template>


<script>
import * as THREE from 'three';
import gsap from 'gsap';
import Stats from '../../node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '../../node_modules/three/examples/jsm/loaders/GLTFLoader.js';

export default {
	name: 'Gym',
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

			objects: [],
			intersects: null
		}
	},
	methods: {
		init: function() {
			const sceneSpace = document.getElementById('sceneSpace');
			this.scene = new THREE.Scene();
			this.camera = new THREE.PerspectiveCamera(75, sceneSpace.clientWidth / sceneSpace.clientHeight, 0.1, 70000);
			this.camera.position.set(0,0,500);
			this.camera.lookAt(0,0,0);
			// this.camera.rotateX(Math.PI / 2);
			this.camera.rotateY(-Math.PI / 2);
			// this.camera.rotateZ(Math.PI / 2);

			const ambient = new THREE.AmbientLight(0x777777, 2);
    		this.scene.add(ambient);

			const loader = new GLTFLoader();

			// Load a glTF resource
			loader.load(
				// resource URL
				'../../gltf/avenue_tennis_-_main_gym_2018/scene.gltf',

				// called when the resource is loaded
				( gltf )=> {
					this.scene.add(gltf.scene);
					this.mesh = gltf.scene.children[0];
					// this.mesh.scale.set(.1, .1, .1);
					this.mesh.position.y -= 1000;
					this.mesh.position.z += 5500;
					console.log(this.mesh.children[0].children[0].children[0].children[16]);

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


			// raycaster
			this.raycaster = new THREE.Raycaster();
			this.mouse = new THREE.Vector2();

			this.renderer = new THREE.WebGLRenderer({antialias: true});
			this.renderer.setPixelRatio(sceneSpace.devicePixelRatio);
			this.renderer.setSize(sceneSpace.clientWidth, sceneSpace.clientHeight);
			// this.renderer.outputEncoding = THREE.sRGBEncoding;
			this.renderer.shadowMap.enabled = true;
			// this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
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


		// mouse down
		onDocumentMouseDown: function(event){
			event.preventDefault();
			this.objects = [];
			this.objects.push(this.mesh.children[0].children[0].children[0].children[16]);
			this.mouse.set(
				(event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
				-(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
			);

			this.raycaster.setFromCamera(this.mouse, this.camera);
			this.intersects = this.raycaster.intersectObjects(this.objects);

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
    	document.addEventListener("mousedown", this.onDocumentMouseDown, false);
		window.addEventListener('resize', () => {this.onWindowResize()}, false);
	},

	created(){
	}
}
</script>
