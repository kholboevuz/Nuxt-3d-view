<template>
  <div>
    <div ref="modelContainer" class="model-container"></div>

    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <h2>Model Data</h2>
        <pre>{{ modelData }}</pre>
      </div>
    </div>
  </div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js'

export default {
  data() {
    return {
      moveForward: false,
      moveBackward: false,
      moveLeft: false,
      moveRight: false,
      moveSpeed: 0.1,
      raycaster: new THREE.Raycaster(),
      mouse: new THREE.Vector2(),
      showModal: false,
      modelData: null,
      model: null, // Initialize the model variable
    }
  },
  mounted() {
    this.initThree()
    this.initKeyboardControls()
    window.addEventListener('click', this.onMouseClick)
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.onKeyDown)
    window.removeEventListener('keyup', this.onKeyUp)
    window.removeEventListener('click', this.onMouseClick)
  },
  methods: {
    initThree() {
      const container = this.$refs.modelContainer
      const width = container.clientWidth
      const height = container.clientHeight

      const scene = new THREE.Scene()

      const camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000)
      camera.position.z = 5

      const renderer = new THREE.WebGLRenderer()
      renderer.setSize(width, height)
      renderer.setClearColor(new THREE.Color(0xffffff))
      container.appendChild(renderer.domElement)

      const controls = new OrbitControls(camera, renderer.domElement)
      controls.enableDamping = true
      controls.dampingFactor = 0.25
      controls.enableZoom = true

      const loader = new OBJLoader()
      loader.load(
        '/model/container.obj',
        obj => {
          this.model = obj
          scene.add(this.model)
          this.animate(scene, camera, renderer, controls)
        },
        undefined,
        error => {
          console.error(error)
        }
      )

      const ambientLight = new THREE.AmbientLight(0xcccccc, 0.4)
      scene.add(ambientLight)

      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
      directionalLight.position.set(1, 1, 0).normalize()
      scene.add(directionalLight)
    },
    animate(scene, camera, renderer, controls) {
      const animate = () => {
        requestAnimationFrame(animate)

        this.updateCameraPosition(camera)
        controls.update()

        renderer.render(scene, camera)
      }
      animate()
    },
    initKeyboardControls() {
      window.addEventListener('keydown', this.onKeyDown)
      window.addEventListener('keyup', this.onKeyUp)
    },
    onKeyDown(event) {
      switch (event.code) {
        case 'ArrowUp':
        case 'KeyW':
          this.moveForward = true
          break
        case 'ArrowLeft':
        case 'KeyA':
          this.moveLeft = true
          break
        case 'ArrowDown':
        case 'KeyS':
          this.moveBackward = true
          break
        case 'ArrowRight':
        case 'KeyD':
          this.moveRight = true
          break
      }
    },
    onKeyUp(event) {
      switch (event.code) {
        case 'ArrowUp':
        case 'KeyW':
          this.moveForward = false
          break
        case 'ArrowLeft':
        case 'KeyA':
          this.moveLeft = false
          break
        case 'ArrowDown':
        case 'KeyS':
          this.moveBackward = false
          break
        case 'ArrowRight':
        case 'KeyD':
          this.moveRight = false
          break
      }
    },
    updateCameraPosition(camera) {
      if (this.moveForward) camera.position.z -= this.moveSpeed
      if (this.moveBackward) camera.position.z += this.moveSpeed
      if (this.moveLeft) camera.position.x -= this.moveSpeed
      if (this.moveRight) camera.position.x += this.moveSpeed
    },
    onMouseClick(event) {
      this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1
      this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1

      this.raycaster.setFromCamera(this.mouse, this.camera)

      const intersects = this.raycaster.intersectObject(this.model, true)

      if (intersects.length > 0) {
        this.modelData = {
          name: intersects[0].object.name,
          position: intersects[0].object.position,
        }
        this.showModal = true
      }
    },
    closeModal() {
      this.showModal = false
    },
  },
}
</script>

<style scoped>
.model-container {
  width: 100%;
  height: 500px;
  position: relative;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
  width: 80%;
  max-width: 600px;
  position: relative;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 24px;
  cursor: pointer;
}
</style>
