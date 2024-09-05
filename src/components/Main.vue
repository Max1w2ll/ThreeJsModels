<script lang="ts">
/* eslint-disable */
import { defineComponent } from "vue";
import * as THREE from 'three';
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { TransformControls } from "three/examples/jsm/controls/TransformControls";
import { GridHelper, AxesHelper, Raycaster } from "three";
import Stats from 'stats.js'

export default {
  name: 'MainPage',
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
scene.background = new THREE.Color( 0xaaa5a0 );

// Set camera and controls
const camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 10000 );
camera.position.x = 10;
camera.position.y = 8;
camera.position.z = 10;
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

// Create cubes
const geometry = new THREE.BoxGeometry( 1, 1, 1 );
const material = new THREE.MeshBasicMaterial( { color: 0xe83f68 } );
const cube = new THREE.Mesh( geometry, material );
scene.add( cube );

const cube2 = new THREE.Mesh( geometry, material );
scene.add( cube2 );

// Raycaster (select mesh)
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
const movePounter = new THREE.Vector2();
let draggable: THREE.Object3D;

window.addEventListener('click', event => {

})

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
  renderer.render( scene, camera );
  stats.end()
}
</script>

<template>
  <div />
</template>


