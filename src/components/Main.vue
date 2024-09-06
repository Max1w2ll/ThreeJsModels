<script lang="ts">
/* eslint-disable */
import * as THREE from 'three';
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { TransformControls } from "three/examples/jsm/controls/TransformControls";
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import Stats from 'stats.js'
import { ref } from "vue";
import { string } from "mathjs";

// const selectedMesh = ref<string>()

export default {
  name: 'MainPage',

  data() {
    return {
      selectedMesh: string
    }
  },

  methods: {
    hideSettings() {
      const settings = document.getElementById('mesh-settings');
      const showSettings = document.getElementById('show-settings');
      settings?.classList.add("hidden");
      showSettings?.classList.remove("hidden");
    },

    showSettings() {
      const settings = document.getElementById('mesh-settings');
      const showSettings = document.getElementById('show-settings');
      settings?.classList.remove("hidden");
      showSettings?.classList.add("hidden");
    },

    createMesh() {
      const loader = new GLTFLoader();

      loader.load( "/geometries/chair.glb", function ( gltf: any ) {
        scene.add( gltf.scene );
      }, undefined, function ( error: any ) {
        console.error( error );
      });
    }
  }
}
// FPS counter
const stats = new Stats()
stats.showPanel(0)
document.body.appendChild(stats.dom)

// Set scene
const renderer = new THREE.WebGLRenderer();
const scene = new THREE.Scene();
renderer.setSize( window.innerWidth, window.innerHeight );
renderer.setAnimationLoop( animate );
document.body.appendChild( renderer.domElement );
scene.background = new THREE.Color( 0x3f3f3f );

// Set camera and controls
const camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 10000 );
camera.position.x = 15;
camera.position.y = 12;
camera.position.z = 15;
camera.rotation.x = -0.5
camera.rotation.y = 0.7
camera.rotation.z = 0.45
const controls = new OrbitControls( camera, renderer.domElement );

// Grid
const size = 999;
const divisions = 999;
const gridHelper = new THREE.GridHelper( size, divisions );
scene.add( gridHelper );

// XYZ Axes
const axesHelper = new THREE.AxesHelper( 999 );
scene.add( axesHelper );

// Raycaster (select mesh)
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
const movePounter = new THREE.Vector2();
let draggable: THREE.Object3D;

window.addEventListener('click', event => {

})

renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;

var geometry = new THREE.BoxGeometry( 3, 3, 3 );
var material = new THREE.MeshStandardMaterial( { color: 0x8f9092 } );
var cube = new THREE.Mesh( geometry, material );
cube.position.set(0, 1.5, 0);
cube.castShadow = true;
cube.receiveShadow = true;
scene.add( cube );

// Ambient light
var ambient = new THREE.AmbientLight( 0xffffff, 0.1 );
scene.add( ambient );

// Spot light
let spotLight = new THREE.SpotLight( 0xffffff, 100 );
spotLight.position.set( 3, 5, -5 );
spotLight.angle = Math.PI / 4;
spotLight.penumbra = 0.05;
/* spotLight.decay = 2; */
spotLight.distance = 1000;
spotLight.castShadow = true;
  spotLight.shadow.mapSize.width = 1024;
spotLight.shadow.mapSize.height = 1024;
spotLight.shadow.camera.near = 2;
/* spotLight.shadow.camera.far = 15; */
scene.add( spotLight );

let lightHelper = new THREE.SpotLightHelper( spotLight );
lightHelper.visible = false;
scene.add( lightHelper );

let shadowCameraHelper = new THREE.CameraHelper( spotLight.shadow.camera );
shadowCameraHelper.visible = false;
scene.add( shadowCameraHelper );

// Transform controls (move mesh)
const transformControls = new TransformControls( camera, renderer.domElement );
transformControls.addEventListener('dragging-changed', function(e) {
  controls.enabled = !e.value;
});
transformControls.attach(cube)
scene.add( transformControls );

// Render
function animate() {
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
          <select :value="selectedMesh">
            <option value="" disabled selected>Select mesh</option>
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


