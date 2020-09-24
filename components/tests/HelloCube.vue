<template>
<div id="sceneSpace" class="w-screen h-screen">
	<div class="absolute flex flex-col items-center justify-center w-screen text-gray-500 select-none">
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
		<div class="text-green-600">Move your mouse over the Cubes 😊</div>
	</div>
</div>
</template>

<script>

import {
	Scene, WebGLRenderer, PerspectiveCamera, BoxGeometry,
	Mesh, AxesHelper, Raycaster, Vector2, CubeTextureLoader,
	DirectionalLight, MeshPhongMaterial
} from 'three';
import _ from 'lodash';

export default {
	name: 'HelloCube',
	data() {
		return {
			sceneSpace: null,
			camera: null,
			scene: null,
			renderer: null,
			light: null,
			raycaster: null,
			mouse: null,
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
				const far = 1000;
				this.camera = new PerspectiveCamera(fov, aspect, near, far);
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
				const intensity = 1;
				this.light = new DirectionalLight(color, intensity);
				this.light.name = "directional light"
				this.light.position.set(0, 0, 20);
				this.light.lookAt(0,0,0);
				this.scene.add(this.light);
			}

			// WebGL renderer
			{
				this.renderer = new WebGLRenderer({antialias: true});
				this.renderer.name = "Renderer";
				this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
				this.renderer.setPixelRatio(this.sceneSpace.devicePixelRatio);
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

				for(let i=0; i<=5; i++){
					const cube = new Mesh(geometry, material);
					cube.position.set(_.random(-2, 1), _.random(-2, 2), _.random(-2, 2));
					cube.name = `Cube ${i}`;
					this.scene.add(cube);
				}
			}

			// Raycaster
			this.raycaster = new Raycaster();

			// Mouse
			this.mouse = new Vector2(); // x, y

			this.renderer.setAnimationLoop(this.animate);
		},

		animate() {
			requestAnimationFrame(this.animate);
			this.renderer.render(this.scene, this.camera);
		},
		onMouseMove(event){
			this.mouse.set(
                (event.clientX / this.renderer.domElement.clientWidth) * 2 - 1, 	// x
                -(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1	// y
            );

			this.raycaster.setFromCamera(this.mouse, this.camera);
			const intersects = this.raycaster.intersectObjects(this.scene.children);

			// What to do with object/s that intersect with the mouse
			if(intersects.length){
				// Take all object
				// intersects.forEach((intersectObj)=>{
				// 	intersectObj.object.rotation.y += 0.05;
				// 	intersectObj.object.rotation.z += 0.02;
				// })

				// Take only the front object
				intersects[0].object.rotation.y += 0.05;
				intersects[0].object.rotation.z += 0.02;
			}
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
		window.addEventListener('resize', this.onWindowResize, false);
		window.addEventListener('mousemove', this.onMouseMove, false);
	},

	created(){
	}
}
</script>
