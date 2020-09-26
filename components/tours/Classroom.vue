<template>
<div id="sceneSpace" class="w-screen h-screen">
    <div class="absolute flex justify-center w-screen text-gray-500 select-none">
        <div class="text-center">
            <h1>3D Model</h1>
            <p>Author: <a target="_blank" href="https://sketchfab.com/GlobalDigitalHeritage"><span class="text-indigo-500">Global Digital Heritage</span></a></p>
            <p>Source: <a target="_blank" href="https://sketchfab.com/3d-models/montemor-o-novos-castle-c3c42b291c9a42e5882cacd3330c04ef"><span class="text-indigo-500">Montemor-o-Novo's Castle</span></a></p>
        </div>
    </div>
	<div class="absolute right-4 bottom-4 w-18 h-10 bg-gray-700">
		<button v-if="inRoom" @click="exitRoon"  class="w-18 h-10 text-gray-400 font-semibold">Exit</button>
		<button v-else @click="enterRoon" class="w-18 h-10 text-gray-400 font-semibold">Enter</button>
	</div>
</div>
</template>

<script>

import {
	PerspectiveCamera, AmbientLight, Scene, WebGLRenderer, Mesh, Raycaster,
	LineBasicMaterial, CircleBufferGeometry, MeshBasicMaterial, Vector2,
	DoubleSide, sRGBEncoding, PCFSoftShadowMap, Group, AxesHelper
} from 'three';
import gsap from 'gsap';
import Stats from '../../node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '../../node_modules/three/examples/jsm/loaders/GLTFLoader.js';

export default {
    name: 'Classroom',
    data() {
        return {
			inRoom: false,
			sceneSpace: null,
            renderer: null,
            stats: null,
            camera: null,
            scene: null,
            raycaster: null,
			targetObjects: null,
			outLine: null,
            circleOverHelper: null,

            mouse: null,
            mouseDown: null,
			tl:null
        }
    },

    methods: {
        init() {
			// Container
            this.sceneSpace = document.getElementById('sceneSpace');
			this.sceneSpace.name = "Scene Space";

			// Camera
			{
				const fov = 75;
				const aspect = this.sceneSpace.clientWidth / this.sceneSpace.clientHeight;
				const near = 0.1;
				const far = 1000;
				this.camera = new PerspectiveCamera(fov, aspect, near, far);
				this.camera.name = "Camera";
				this.camera.position.set(0,5,20);
				this.camera.destination = {x: 0, y: 5, z: 20}
			}

			// Scene
			{
				this.scene = new Scene();
				this.scene.name  = "Scene";
			}

			// Helper
			{
				let helper = new AxesHelper(100);
				this.scene.add(helper);
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
				this.stats.name = "Stats Bar"
				this.sceneSpace.appendChild(this.stats.domElement);
			}

			// Light
			{
				const color = 0x555555;
				const intensity = 2;
				const ambientLight = new AmbientLight(color, intensity);
				ambientLight.name = "Ambient Light";
				this.scene.add(ambientLight);
			}

            // Load GLTF
			{
				const loader = new GLTFLoader();

				loader.load(
					'../../gltf/lowpoly_stylized_classroom/scene.gltf',

					// Call once loaded
					(gltf) => {
						gltf.scene.name = "GLTF Scene";
						gltf.scene.scale.set(0.2,0.2, 0.2);
						this.scene.add(gltf.scene);
						this.targetObjects = gltf.scene.children[0].children[0].children[0].children;
						console.log(this.targetObjects);
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
				this.circleOverHelper.rotateX(Math.PI / 2);

				const outLineMaterial = new MeshBasicMaterial({
					color: 0xffffff,
					opacity: 1,
					side: DoubleSide
				});
				this.outLine = new Mesh(circleRollerGeo, outLineMaterial);
				this.outLine.position = this.circleOverHelper.position;
				this.outLine.scale.multiplyScalar(.7);

				this.circleOverHelper.add(this.outLine);
				this.circleOverHelper.name = "Circle Helper";
				this.scene.add(this.circleOverHelper);
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

        onMouseMove(event) {
			event.preventDefault();
            this.mouse.set(
                (event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
                -(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
            );

            this.raycaster.setFromCamera(this.mouse, this.camera);
            let intersects = this.raycaster.intersectObjects(this.targetObjects);

            if (intersects.length) {
                let intersect = intersects[0];

                if (intersect.object.id == 344){ // Not a good approach becuase id may change based on order of object
                    this.circleOverHelper.position = intersect.point
                }
            }
        },

        onMouseDown(event) {
			event.preventDefault();
            this.mouse.set(
                (event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
                -(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
            );

            this.raycaster.setFromCamera(this.mouse, this.camera);
            let intersects = this.raycaster.intersectObjects(this.targetObjects);

            if (intersects.length) {
                let intersect = intersects[0];

                // Here is the magic 🤩🤩
                // Move camera 🎉🎉🎉🎉🎉🎉🎉
                if (intersect.object.id == 344){ // Not a good approach becuase id may change based on order of object

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

		enterRoon(){
			this.inRoom = true;
			this.tl = gsap.timeline({});
			this.tl.to(this.camera.position,{
				duration: 1,
				y: 2,
				z: 2,
				ease:"Power1.InOut",
				onUpdate: ()=>this.camera.updateProjectionMatrix(),
				onComplete:()=>{}
			}).to(this.scene.rotation,{
				duration: 1,
				y: 10,
				ease:"Power1",
			}, "-=0.5")
		},
		exitRoon(){
			this.inRoom = false;
			this.tl.reverse();
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

	computed: {
    },

    created() {}
}
</script>
