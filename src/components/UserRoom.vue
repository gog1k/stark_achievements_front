<template>
    <div class="container-fluid">
        <div class="row">
            <div class="room" ref="canvasWrapper">
                <template
                    v-for="item in items"
                    :key="item"
                >
                    <room-item
                        :uid="item.uid"
                        :object="item.object"
                        :template="item.template"
                        :coordinates="item.coordinates"
                        :rotation="item.rotation"
                    >
                    </room-item>
                </template>
            </div>
        </div>
    </div>
</template>
<script>
import * as THREE from 'three'
import * as TWEEN from '@tweenjs/tween.js'
import RoomItem from '@/components/RoomItem.vue'

export default {
    name: 'user-room',
    components: {
        RoomItem,
    },
    data() {
        return {
            target: new THREE.Vector3(100, 100, 100),
            distance: 1500,
            phi: Math.PI / 2,
            theta: Math.PI / 2,
            minPhi: 0.1,
            maxPhi: Math.PI / 2,
            minTheta: 0,
            maxTheta: Math.PI / 2,
            loading: true,
            mouseX: 0,
            mouseY: 0,
            move: false,
            mouse: new THREE.Vector2(),
            raycaster: new THREE.Raycaster(),
            isDragging: false,
            currentObject: false,
            allowMoving: false,
            items: [
                // {
                //     coordinates: {
                //         x: 0,
                //         y: 0,
                //         z: 0,
                //     },
                //     rotation: {
                //         x: 0,
                //         y: 0,
                //         z: 0,
                //     },
                //     object: '/images/sky.obj',
                //     template: '/images/sky.mtl',
                // },
                {
                    coordinates: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    rotation: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    object: '/images/grass.obj',
                    template: '/images/grass.mtl',
                },
                {
                    coordinates: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    rotation: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    object: '/images/room.obj',
                    template: '/images/room.mtl',
                },
                {
                    coordinates: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    rotation: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    object: '/images/shelf.obj',
                    template: '/images/shelf.mtl',
                },
                // {
                //     coordinates: {
                //         x: 0,
                //         y: 0,
                //         z: 200,
                //     },
                //     rotation: {
                //         x: 0,
                //         y: 0,
                //         z: 0,
                //     },
                //     object: '/images/shelf.obj',
                //     template: '/images/shelf.mtl',
                // },
                {
                    coordinates: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    rotation: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    object: '/images/window.obj',
                    template: '/images/window.mtl',
                },
                {
                    coordinates: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    rotation: {
                        x: 0,
                        y: 0,
                        z: 0,
                    },
                    object: '/images/door.obj',
                    template: '/images/door.mtl',
                },
            ],
        }
    },
    mounted() {
        this.scene = new THREE.Scene()

        this.camera = new THREE.PerspectiveCamera(45, this.$el.clientWidth / this.$el.clientHeight, 1, 10000)
        this.rotateCamera(75, 30)
        this.camera.lookAt(0, 0, 0)

        this.renderer = new THREE.WebGLRenderer({ antialias: true })
        this.renderer.setSize(this.$el.clientWidth, this.$el.clientHeight)
        this.renderer.setClearColor(0x3B3B3B, 1)
        this.$refs.canvasWrapper.appendChild(this.renderer.domElement)

        let light = new THREE.DirectionalLight(0xFFFFFF, 1)
        light.position.set(900, 600, 900)
        this.scene.add(light)

        // let earthGeometry = new THREE.SphereGeometry(1, 32, 32);
        // let earthMaterial = new THREE.MeshPhongMaterial({
        //     map: new THREE.TextureLoader().load('https://threejs.org/examples/textures/earth_atmos_2048.jpg'),
        //     specularMap: new THREE.TextureLoader().load('https://threejs.org/examples/textures/earth_specular_2048.jpg'),
        //     specular: new THREE.Color('grey'),
        //     shininess: 5
        // });
        // let earthMesh = new THREE.Mesh(earthGeometry, earthMaterial);
        // this.scene.add(earthMesh);

        // const skyGeometry = new THREE.SphereGeometry(10000, 32, 32);
        // const skyMaterial = new THREE.MeshBasicMaterial({
        //     map: new THREE.TextureLoader().load('https://threejs.org/examples/textures/2k_stars.jpg'),
        //     side: THREE.BackSide
        // });
        // const skyMesh = new THREE.Mesh(skyGeometry, skyMaterial);
        // this.scene.add(skyMesh);

        this.$refs.canvasWrapper.addEventListener('mousedown', this.handleMouseDown)
        this.$refs.canvasWrapper.addEventListener('mousemove', this.handleMouseMove)
        this.$refs.canvasWrapper.addEventListener('mouseup', this.handleMouseUp)
        this.$refs.canvasWrapper.addEventListener('click', this.handleMouseClick)

        this.render()

        this.loading = false
    },
    beforeUnmount() {
        this.$refs.canvasWrapper.removeEventListener('mousedown', this.handleMouseDown)
        this.$refs.canvasWrapper.removeEventListener('mousemove', this.handleMouseMove)
        this.$refs.canvasWrapper.removeEventListener('mouseup', this.handleMouseUp)
    },
    methods: {
        handleMouseClick(event) {
            this.mouse = new THREE.Vector2()
            this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1
            this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1

            this.raycaster.setFromCamera(this.mouse, this.camera)

            let intersects = this.raycaster.intersectObjects(this.scene.children, true)

            if (intersects.length) {
                let object = this.scene.getObjectById(intersects[0].object.id)
                console.log('click', intersects[0].object)
                console.log('object', object)
            }
        },
        handleMouseDown(event) {
            this.isDragging = true
            this.mouseX = event.clientX
            this.mouseY = event.clientY
        },
        handleMouseMove(event) {
            if (
                this.allowMoving
                && !this.currentObject
            ) {
                console.log('moving')
                // this.currentObject = object
            }

            if (this.isDragging) {
                if (this.currentObject) {
                    const deltaX = event.clientX - this.mouseX
                    const deltaY = event.clientY - this.mouseY
                    this.moveElement(-deltaX / 2, deltaY / 2)
                    this.mouseX = event.clientX
                    this.mouseY = event.clientY
                } else {
                    const deltaX = event.clientX - this.mouseX
                    const deltaY = event.clientY - this.mouseY
                    this.rotateCamera(-deltaX / 2, deltaY / 2)
                    this.mouseX = event.clientX
                    this.mouseY = event.clientY
                }
            }
        },
        handleMouseUp() {
            this.isDragging = false
            this.currentObject = false
        },
        moveElement(deltaX, deltaY) {
            console.log(deltaX)
            console.log(deltaY)
        },
        rotateCamera(deltaX, deltaY) {
            this.phi -= deltaY * 0.01
            this.theta -= deltaX * 0.01

            this.phi = Math.max(this.minPhi, Math.min(this.maxPhi, this.phi))

            this.theta = Math.min(this.maxTheta, Math.max(this.minTheta, this.theta))

            this.camera.position.x = this.target.x + this.distance * Math.sin(this.phi) * Math.cos(this.theta)
            this.camera.position.y = this.target.y + this.distance * Math.cos(this.phi)
            this.camera.position.z = this.target.z + this.distance * Math.sin(this.phi) * Math.sin(this.theta)

            this.camera.lookAt(this.target)
        },

        render() {
            this.renderer.render(this.scene, this.camera)
            requestAnimationFrame(this.render)
            TWEEN.update()
        },
    },
}
</script>

<style>
.room {
    display: flex;
    align-items: center;
    width: 100%;
    height: 100vh;
    margin-top: -56px;
    padding-top: 56px;
    color: #9e4c4c
}
</style>