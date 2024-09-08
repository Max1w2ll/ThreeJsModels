<script lang="ts" setup>
/* eslint-disable */
import * as THREE from 'three'
import { onMounted, ref } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { TransformControls } from "three/examples/jsm/controls/TransformControls"
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import Stats from 'stats.js'
import {
  models,
  albedoTextures,
  metalnessTextures,
  normalTextures,
  roughnessTextures,
  sheenTextures
} from "@/types/enums"

// Scene
const stats = new Stats()
const renderer = new THREE.WebGLRenderer()
const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 10000 )
const controls = new OrbitControls( camera, renderer.domElement )
const axesHelper = new THREE.AxesHelper( 999 )

// Lighting
const spotLight = new THREE.SpotLight( 0xffffff, 100 )
const lightHelper = new THREE.SpotLightHelper( spotLight )
const shadowCameraHelper = new THREE.CameraHelper( spotLight.shadow.camera )

// Selected values
const selectedMesh = ref<string>('')
const selectedAlbedo = ref<string>('')
const selectedMetalness = ref<string>('')
const selectedNormal = ref<string>('')
const selectedRoughness = ref<string>('')
const selectedSheen = ref<string>('')

// Select and move
const raycaster = new THREE.Raycaster()
const selectableGroup = new THREE.Group()
const selectedObject = ref()
const transformControls = new TransformControls( camera, renderer.domElement )

onMounted(() => {
  createFPSCounter()
  createScene()
  createCamera()
  createGrid()
  createXYZ()
  createAmbientLight()
  createSpotLight()
  createPivot()
})

// Setting up scene //
const createFPSCounter = () => {
  stats.showPanel(0)
  document.body.appendChild(stats.dom)
}

const createScene = () => {
  renderer.setSize( window.innerWidth, window.innerHeight )
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.setAnimationLoop( animate )
  document.body.appendChild( renderer.domElement )
  scene.background = new THREE.Color( 0x3f3f3f )
}

const createCamera = () => {
  camera.position.x = 5
  camera.position.y = 4
  camera.position.z = 5
  camera.rotation.x = -0.5
  camera.rotation.y = 0.7
  camera.rotation.z = 0.45
}

const createGrid = () => {
  const size = 999
  const divisions = 999
  const gridHelper = new THREE.GridHelper( size, divisions )
  scene.add( gridHelper )
}

const createXYZ = () => {
  scene.add( axesHelper )
}

const createAmbientLight = () => {
  const ambient = new THREE.AmbientLight( 0xffffff, 0.1 )
  scene.add( ambient )
}

const createSpotLight = () => {
  spotLight.position.set( 3, 5, -5 )
  spotLight.angle = Math.PI / 4
  spotLight.penumbra = 0.05
  spotLight.distance = 1000
  spotLight.castShadow = true
  spotLight.shadow.mapSize.width = 1024
  spotLight.shadow.mapSize.height = 1024
  spotLight.shadow.camera.near = 2
  scene.add( spotLight )

  setLightHelper()
  setShadowHelper()
}

const setLightHelper = () => {
  lightHelper.visible = false
  scene.add( lightHelper )
}

const setShadowHelper = () => {
  shadowCameraHelper.visible = false
  scene.add( shadowCameraHelper )
}

const createPivot = () => {
  transformControls.addEventListener('dragging-changed', function(e) {
    controls.enabled = !e.value
  })
  scene.add( transformControls )
  scene.add( selectableGroup )
}

const select = (event: any) => {
  const coords = new THREE.Vector2(
    (event.clientX / renderer.domElement.clientWidth) * 2 - 1,
    -((event.clientY / renderer.domElement.clientHeight) * 2 - 1),
  )
  raycaster.setFromCamera(coords, camera)
  const instersections = raycaster.intersectObjects(selectableGroup.children, true)
  if (instersections.length > 0) {
    selectedObject.value = instersections[0].object
    transformControls.attach(selectedObject.value)
  }
}
document.addEventListener('mousedown', select)

const animate = () => {
  stats.begin()

  lightHelper.update()
  shadowCameraHelper.update()
  renderer.render( scene, camera )

  stats.end()
}

// Right menu //
const hideSettings = () => {
  const settings = document.getElementById('settings')
  const showSettingsTab = document.getElementById('show-settings')
  settings?.classList.add("hidden")
  showSettingsTab?.classList.remove("hidden")
}

const showSettings = () => {
  const settings = document.getElementById('settings')
  const showSettingsTab = document.getElementById('show-settings')
  settings?.classList.remove("hidden")
  showSettingsTab?.classList.add("hidden")
}

const getCoords = () => {
  return selectedObject.value?.position
}

// Mesh functions //
const createMesh = () => {
  if (!selectedMesh.value) return
  const loader = new GLTFLoader()
  loader.load( `/geometries/${selectedMesh.value}.glb`, function ( gltf: any ) {
    scene.add( gltf.scene )
    selectableGroup.add( gltf.scene )
  }, undefined, function ( error: any ) {
    console.error( error )
  })
}

const removeMesh = () => {
  transformControls.detach()
  scene.remove( selectedObject.value );
  selectedObject.value.removeFromParent();
  selectedObject.value = {}
}

const applyTextures = () => {
  const texture = THREE.TextureLoader().load(`/textures/${selectedMesh.value}.glb`)
}

</script>

<template>
  <div class="show-settings hidden" id="show-settings">
    <img @click="showSettings()" class="arrow-left" src="@/assets/images/arrow_left.svg" alt="">
  </div>
  <div class="main-settings" id="settings">
      <div class="title-icon">
        <p class="title"> Settings </p>
        <img @click="hideSettings()" class="arrow-right" src="@/assets/images/arrow_right.svg" alt="">
      </div>
      <div class="settings">
        <div class="setting">
          <div class="selected-delete">
            <span class="title"> Selected: {{ selectedObject?.name ? selectedObject?.name : 'none' }} </span>
            <img v-if="selectedObject?.name" @click="removeMesh()" class="delete" src="@/assets/images/delete.svg" alt="">
          </div>
          <p class="title"> Location </p>
          <input :value="getCoords()?.x.toFixed(2)" type="number" placeholder="x"/>
          <input :value="getCoords()?.y.toFixed(2)" type="number" placeholder="y"/>
          <input :value="getCoords()?.z.toFixed(2)" type="number" placeholder="z"/>
        </div>
        <div class="setting">
          <p class="title"> Materials </p>
          <select v-model="selectedAlbedo">
            <option value="" disabled selected>albedo</option>
            <option v-for="texture in albedoTextures"> {{ texture.label }} </option>
          </select>
          <select v-model="selectedRoughness">
            <option value="" disabled selected>roughness</option>
            <option v-for="texture in roughnessTextures"> {{ texture.label }} </option>
          </select>
          <select v-model="selectedMetalness">
            <option value="" disabled selected>metalness</option>
            <option v-for="texture in metalnessTextures"> {{ texture.label }} </option>
          </select>
          <select v-model="selectedNormal">
            <option value="" disabled selected>normal</option>
            <option v-for="texture in normalTextures"> {{ texture.label }} </option>
          </select>
          <select v-model="selectedSheen">
            <option value="" disabled selected>sheen</option>
            <option v-for="texture in sheenTextures"> {{ texture.label }} </option>
          </select>
          <button type="button"> Apply materials </button>
        </div>
        <div class="setting">
          <p class="title"> Create mesh </p>
          <select v-model="selectedMesh">
            <option value="" disabled selected>Select mesh</option>
            <option v-for="model in models"> {{ model.value }} </option>
          </select>
          <button @click="createMesh()" type="button">Create</button>
        </div>
      </div>
  </div>
</template>

<style scoped>
.main-settings {
  position: absolute;
  top: 64px;
  right: 5px;
  height: 600px;
  width: 200px;
  z-index: 9999;
  background: #393939;
  border-radius: 8px;
  color: #fefefe;
  padding: 0 12px;
  text-align: left;
  font-size: 12px;
  font-family: system-ui;
  transition: 0.5s;
  transform: translateX(0);

  .title-icon {
    .arrow-right {
      position: absolute;
      top: 12px;
      right: 12px;
      cursor: pointer;
    }
  }

  .selected-delete {
    position: relative;
    display: inline-block;
    word-break: break-word;

    .title {
      width: 170px !important;
      display: inline-grid;
    }

    .delete {
      position: absolute;
      right: 0;
      cursor: pointer;
    }
  }

  .settings {
    display: grid;
    gap: 20px;

    .setting {
      display: inline-grid;
      gap: 8px;
      width: 100%;

      .title {
        font-size: 12px;
        color: #c3c3c3;
      }

      input, button {
        background: #545454;
        border: none;
        border-radius: 4px;
        color: #fefefe;
      }

      button {
        cursor: pointer;
      }

      input::-webkit-outer-spin-button,
      input::-webkit-inner-spin-button {
        -webkit-appearance: none;
        margin: 0;
      }

      textarea:focus, input:focus {
        outline: none;
      }

      select {
        background: #282828;
        border: none;
        border-radius: 4px;
        color: #fefefe;
      }
    }
  }
}

.show-settings {
  position: absolute;
  top: 64px;
  right: 10px;
  padding: 20px 0 10px;
  height: 30px;
  width: 30px;
  z-index: 9999;
  background: #393939;
  border-radius: 8px;
  color: #fefefe;
  transition: 0.5s;
  transform: translateX(0);

  .arrow-left {
    cursor: pointer;
  }
}

.hidden {
  transform: translateX(300px);
}

</style>


