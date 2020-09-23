<template>
<div id="sceneSpace" class="w-screen h-screen">
    <div class="absolute flex justify-center w-screen text-gray-500 select-none">
        <div class="text-center">
            <h1>3D Model</h1>
            <p>Author: <a target="_blank" href="https://sketchfab.com/GlobalDigitalHeritage"><span class="text-indigo-500">Global Digital Heritage</span></a></p>
            <p>Source: <a target="_blank" href="https://sketchfab.com/3d-models/montemor-o-novos-castle-c3c42b291c9a42e5882cacd3330c04ef"><span class="text-indigo-500">Montemor-o-Novo's Castle</span></a></p>
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
import Stats from '../../node_modules/three/examples/jsm/libs/stats.module.js';
import {GLTFLoader} from '../../node_modules/three/examples/jsm/loaders/GLTFLoader.js';

export default {
    name: 'Classroom',
    data() {
        return {
			sceneSpace: null,
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

            circleRoller: null,
            circleRollerGeo: null,
            circleRollerMaterial: null,

            outLineMaterial: null,
            outLine: null,

            objects: [],
            intersects: null,
            group: null,

			delta: 1000 / 60,
			timeTarget: 0
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
				this.camera.position.set(0,10,10);
			}

			// Scene
			{
				this.scene = new Scene();
				this.scene.name  = "Scene";
			}

			// WebGL Renderer
			{
				this.renderer = new WebGLRenderer({antialias: true});
				this.renderer.setPixelRatio(this.sceneSpace.devicePixelRatio);
				this.renderer.setSize(this.sceneSpace.clientWidth, this.sceneSpace.clientHeight);
				this.renderer.outputEncoding = sRGBEncoding;
				this.renderer.shadowMap.enabled = true;
				this.renderer.shadowMap.type = PCFSoftShadowMap;
				this.sceneSpace.appendChild(this.renderer.domElement);

				this.stats = new Stats();
				this.sceneSpace.appendChild(this.stats.domElement);
			}

			// Group
			{
				this.group = new Group();
				this.scene.add(this.group);
			}

			// Light
			{
				const ambientLight = new AmbientLight(0x555555, 2);
				this.scene.add(ambientLight);
			}

            // Load GLTF
			{
				const loader = new GLTFLoader();

				loader.load(
					'../../gltf/lowpoly_stylized_classroom/scene.gltf',

					// called when the resource is loaded
					(gltf) => {
						this.scene.add(gltf.scene);
						this.mesh = gltf.scene.children[0].children[0].children[0];
					},

					// called while loading is progressing
					function (xhr) {
						console.log((xhr.loaded / xhr.total * 100) + '% loaded');
					},

					// called when loading has errors
					function (error) {
						console.log('An error happened');
					}
				);
			}

            // Circle roller
			{
				this.circleRollerGeo = new CircleBufferGeometry(1, 100,);
				this.circleRollerMaterial = new LineBasicMaterial({
					color: 0xffffff,
					opacity: .8,
					transparent: false
				});

				this.circleRoller = new Mesh(this.circleRollerGeo, this.circleRollerMaterial);
				this.circleRoller.rotateX(Math.PI / 2);

				this.scene.add(this.circleRoller);

				this.outLineMaterial = new MeshBasicMaterial({
					color: 0xffffff,
					opacity: 1,
					side: DoubleSide
				});

				this.outLine = new Mesh(this.circleRollerGeo, this.outLineMaterial);
				this.outLine.position = this.circleRoller.position;
				this.outLine.scale.multiplyScalar(.7);
				this.circleRoller.add(this.outLine);
			}

            // Raycaster
            this.raycaster = new Raycaster();

			// Mouse
            this.mouse = new Vector2();

        },

        animate() {
			setTimeout( () => {
				requestAnimationFrame( this.animate );
				this.renderer.render(this.scene, this.camera);
				this.stats.update();
			}, 1000 / 60);	// 60 fps, 1000/30 for 30 fps
        },

        initControls() {
            this.camera.rotation.order = "YXZ";

            // mouse down event
            document.addEventListener("mousedown", (evt) => {
                this.mouseDown = true;
                this.sceneSpace.style.cursor = "grab";
            });

            // mouse up event
            document.addEventListener("mouseup", (evt) => {
                this.sceneSpace.style.cursor = "default";
                this.mouseDown = false;
            });

            document.addEventListener("mousemove", (evt) => {
				let movementX = evt.movementX || evt.mozMovementX || evt.webkitMovementX || 0;
				let movementY = evt.movementY || evt.mozMovementY || evt.webkitMovementY || 0;

				if (this.mouseDown) {
					this.sceneSpace.style.cursor = "grabbing";
					this.camera.rotation.y -= -movementX / 600;
					this.camera.rotation.x -= -movementY / 600;
				}
            },false);
        },

        onDocumentMouseMove(event) {

            this.mouse.set(
                (event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
                -(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
            );

            this.raycaster.setFromCamera(this.mouse, this.camera);

            // ray will intersect with objects added to array named objects
            // this.intersects = this.raycaster.intersectObjects(this.objects);
            this.intersects = this.raycaster.intersectObjects(this.scene.children[3].children[0].children[0].children[0].children);

            /*
            the picking ray will intersects with one or more of all objects in the scene
            let intersects = raycaster.intersectObjects(scene.children);
            */

            if (this.intersects.length) {

                let intersect = this.intersects[0];

                if (intersect.object.id == 72){
                    // this.circleRoller.position.copy(intersect.point).add(intersect.face.normal);
                    this.circleRoller.position = intersect.point
                }
            }
        },

        // mouse down
        onDocumentMouseDown(event) {
            this.mouse.set(
                (event.clientX / this.renderer.domElement.clientWidth) * 2 - 1,
                -(event.clientY / this.renderer.domElement.clientHeight) * 2 + 1
            );

            this.raycaster.setFromCamera(this.mouse, this.camera);
            // this.intersects = this.raycaster.intersectObjects(this.objects);
            this.intersects = this.raycaster.intersectObjects(this.scene.children[3].children[0].children[0].children[0].children);

            if (this.intersects.length) {

                let intersect = this.intersects[0];

                // here is the magic 🤩🤩
                // Move camera 🎉🎉🎉🎉🎉🎉🎉
                if (intersect.object.id == 72){

                    gsap.to(this.camera.position, 2, {
                        x: intersect.point.x,
                        z: intersect.point.z,
                        ease: "Power1.easeInOut",
                        onUpdate: () => {
                            this.camera.updateProjectionMatrix();
                        },
                        onComplete: () => {
                            console.log('animation ends');
                        }
                    });
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
		this.animate();
        document.addEventListener("mousemove", this.onDocumentMouseMove, false);
        document.addEventListener("mousedown", this.onDocumentMouseDown, false);
        window.addEventListener('resize', this.onWindowResize, false);
    },

	computed: {
    },

    created() {}
}
</script>
