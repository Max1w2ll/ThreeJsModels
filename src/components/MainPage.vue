<script lang="ts" setup>
/* eslint-disable */
import * as THREE from 'three';
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { TransformControls } from "three/examples/jsm/controls/TransformControls";
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import Stats from 'stats.js'
import { models } from "@/types/enums";
import { onMounted, ref } from "vue";

const stats = new Stats()
const renderer = new THREE.WebGLRenderer();
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 10000 );
const axesHelper = new THREE.AxesHelper( 999 );
const controls = new OrbitControls( camera, renderer.domElement );

const geometry = new THREE.BoxGeometry( 3, 3, 3 );
const material = new THREE.MeshStandardMaterial( { color: 0x8f9092 } );
const cube = new THREE.Mesh( geometry, material );

const spotLight = new THREE.SpotLight( 0xffffff, 100 );
const lightHelper = new THREE.SpotLightHelper( spotLight );
const shadowCameraHelper = new THREE.CameraHelper( spotLight.shadow.camera );

const selectedMesh = ref<string>('')

onMounted(() => {
  createFPSCounter()
  createScene()
  createCamera()
  createGrid()
  createXYZ()
  createCube()
  createAmbientLight()
  createSpotLight()
  createPivot()
})

const createFPSCounter = () => {
  stats.showPanel(0)
  document.body.appendChild(stats.dom)
}

const hideSettings = () => {
  const settings = document.getElementById('mesh-settings');
  const showSettings = document.getElementById('show-settings');
  settings?.classList.add("hidden");
  showSettings?.classList.remove("hidden");
}

const showSettings = () => {
  const settings = document.getElementById('mesh-settings');
  const showSettings = document.getElementById('show-settings');
  settings?.classList.remove("hidden");
  showSettings?.classList.add("hidden");
}

const createMesh = () => {
  if (!selectedMesh.value) return
  const loader = new GLTFLoader();
  loader.load( `/geometries/${selectedMesh.value}.glb`, function ( gltf: any ) {
    scene.add( gltf.scene );
  }, undefined, function ( error: any ) {
    console.error( error );
  });
}

const createScene = () => {
  renderer.setSize( window.innerWidth, window.innerHeight );
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.setAnimationLoop( animate );
  document.body.appendChild( renderer.domElement );
  scene.background = new THREE.Color( 0x3f3f3f );
}

const createCamera = () => {
  camera.position.x = 15;
  camera.position.y = 12;
  camera.position.z = 15;
  camera.rotation.x = -0.5
  camera.rotation.y = 0.7
  camera.rotation.z = 0.45
}

const createGrid = () => {
  const size = 999;
  const divisions = 999;
  const gridHelper = new THREE.GridHelper( size, divisions );
  scene.add( gridHelper );
}

const createXYZ = () => {
  scene.add( axesHelper );
}

// Raycaster (select mesh)
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
const movePounter = new THREE.Vector2();
let draggable: THREE.Object3D;

window.addEventListener('click', event => {

})

const createCube = () => {
  cube.position.set(0, 1.5, 0);
  cube.castShadow = true;
  cube.receiveShadow = true;
  scene.add( cube );
}

const createAmbientLight = () => {
  // Ambient light
  var ambient = new THREE.AmbientLight( 0xffffff, 0.1 );
  scene.add( ambient );
}

const createSpotLight = () => {
  spotLight.position.set( 3, 5, -5 );
  spotLight.angle = Math.PI / 4;
  spotLight.penumbra = 0.05;
  spotLight.distance = 1000;
  spotLight.castShadow = true;
  spotLight.shadow.mapSize.width = 1024;
  spotLight.shadow.mapSize.height = 1024;
  spotLight.shadow.camera.near = 2;
  scene.add( spotLight );

  setLightHelper()
  setShadowHelper()
}

const setLightHelper = () => {
  lightHelper.visible = false;
  scene.add( lightHelper );
}

const setShadowHelper = () => {
  shadowCameraHelper.visible = false;
  scene.add( shadowCameraHelper );
}

const createPivot = () => {
  const transformControls = new TransformControls( camera, renderer.domElement );
  transformControls.addEventListener('dragging-changed', function(e) {
    controls.enabled = !e.value;
  });
  transformControls.attach(cube)
  scene.add( transformControls );
}

const animate = () => {
  stats.begin()

  lightHelper.update();
  shadowCameraHelper.update();
  renderer.render( scene, camera );

  stats.end()
}
</script>

<template>
  <div class="show-settings hidden" id="show-settings">
    <img @click="showSettings()" class="arrow-left" src="@/assets/images/arrow_left.svg" alt="">
  </div>
  <div class="mesh-settings" id="mesh-settings">
      <div class="title-arrow">
        <p class="title"> Mesh settings </p>
        <img @click="hideSettings()" class="arrow-right" src="@/assets/images/arrow_right.svg" alt="">
      </div>
      <div class="settings">
        <div class="setting">
          <p class="title"> Location </p>
          <input type="number" placeholder="x"/>
          <input type="number" placeholder="y"/>
          <input type="number" placeholder="z"/>
        </div>
        <div class="setting">
          <p class="title"> Materials </p>
          <select>
            <option value="" disabled selected>albedo</option>
          </select>
          <select>
            <option value="" disabled selected>roughness</option>
          </select>
          <select>
            <option value="" disabled selected>metalness</option>
          </select>
          <select>
            <option value="" disabled selected>normal</option>
          </select>
          <select>
            <option value="" disabled selected>sheen</option>
          </select>
        </div>
        <div class="setting">
          <p class="title"> Create mesh </p>
          <select v-model="selectedMesh">
            <option value="" disabled selected>Select mesh</option>
            <option v-for="model in models">
              {{ model.value }}
            </option>
          </select>
          <button @click="createMesh()" type="button">Create</button>
        </div>
      </div>
  </div>
</template>

<style scoped>
.mesh-settings {
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

  .title-arrow {
    .arrow-right {
      position: absolute;
      top: 12px;
      right: 12px;
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


