<template>
	<div id="sceneSpace" class="w-screen h-screen">
		<div class="flex justify-center absolute w-screen text-gray-500">
			<div class="text-center">
				<h1>3D Model</h1>
				<p>Author: <a target="_blank" href="https://sketchfab.com/tafar165"><span class="text-indigo-500">MD Ashfaq Bin Arif</span></a></p>
				<p>Source: <a target="_blank" href="https://sketchfab.com/models/a95a81f32bb1461da296d1e2351e7093"><span class="text-indigo-500">Office Interior</span></a></p>
			</div>
		</div>
	</div>
</template>


<script>
import * as THREE from 'three';
import {GLTFLoader} from '../../node_modules/three/examples/jsm/loaders/GLTFLoader.js';
import {OrbitControls} from '../../node_modules/three/examples/jsm/controls/OrbitControls.js';
import {DRACOLoader} from '../../node_modules/three/examples/jsm/loaders/DRACOLoader.js';

export default {
	name: 'Interior',
	data() {
		return {
			camera: null,
			scene: null,
			renderer: null,
			controls: null,
			mesh: null,
		}
	},
	methods: {
		init: function() {
			const sceneSpace = document.getElementById('sceneSpace');
			this.camera = new THREE.PerspectiveCamera(75, sceneSpace.clientWidth / sceneSpace.clientHeight, 0.1, 1000);
			this.camera.position.set(0,0,100);

			this.scene = new THREE.Scene();

			const ambient = new THREE.AmbientLight(0x777777, 1);
    		this.scene.add(ambient);



			// Instantiate a loader
			const loader = new GLTFLoader();

			// Load a glTF resource
			loader.load(
				// resource URL
				'../../gltf/office_interior/scene.gltf',

				// called when the resource is loaded
				( gltf )=> {
					// console.log(gltf);
					this.scene.add(gltf.scene);
					this.mesh = gltf.scene.children[0]
					// this.mesh.position.y += 140;
					// this.mesh.rotation.z = -Math.PI;
					console.log(this.mesh);

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



			this.renderer = new THREE.WebGLRenderer({antialias: true});
			this.renderer.setSize(sceneSpace.clientWidth, sceneSpace.clientHeight);
			this.renderer.setPixelRatio(sceneSpace.devicePixelRatio);
			this.renderer.outputEncoding = THREE.sRGBEncoding;
			this.renderer.shadowMap.enabled = true;
			this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;

			sceneSpace.appendChild(this.renderer.domElement);

			this.controls = new OrbitControls(this.camera, this.renderer.domElement);
			this.controls.target.set(0,0,0);
			this.controls.update();

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
<style lang="css">
*{
	margin: 0;
	padding: 0;
}
</style>
