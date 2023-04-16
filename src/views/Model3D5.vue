<template>
  <div id="academy">
  </div>
</template>

<script>
import * as THREE from "three";
// import {OBJLoader,MTLLoader} from "three-obj-mtl-loader";
import {OBJLoader} from 'three/examples/jsm/loaders/OBJLoader';
import {MTLLoader} from 'three/examples/jsm/loaders/MTLLoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
// const OrbitControls = require('three-orbit-controls')(THREE);
export default {
  name:'Model3D5',
  data(){
    return{
      // scene:'',
      // light:'',
      // camera:'',
      // renderer:'',
    }
  },

  methods: {
    /**初始化 */
    initScene() {
      this.scene = new THREE.Scene();
      const ambientLight = new THREE.AmbientLight(0xcccccc, 0.4);
      this.scene.add(ambientLight);
      var axesHelper = new THREE.AxesHelper(500);
      this.scene.add(axesHelper);
    },
    initCamera() {
      const aspect = window.innerWidth / innerHeight; //宽高可根据实际项目要求更改 如果是窗口高度改为innerHeight
      this.camera = new THREE.PerspectiveCamera(45, aspect, 1, 2000);
      this.camera.position.set(-400, 400, 400);
      this.camera.lookAt(new THREE.Vector3(0, 0, 0)); // 让相机指向原点

      const pointLight = new THREE.PointLight(0xffffff,1,100);
      pointLight.position.set(0,0,20100);
      this.scene.add(pointLight);
      this.scene.add(this.camera);
    },
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, innerHeight);
      this.container = document.getElementById("academy");
      this.container.appendChild(this.renderer.domElement);
    },
    // 初始化控制器
    initOrbitControls(){
      let controls = new OrbitControls(this.camera,this.renderer.domElement);
      controls.enableDamping = true;
      controls.enableZoom = true;
      controls.autoRotate = false;
      controls.autoRotateSpeed = 3;
      controls.enablePan = true;
      controls.enableKeys = true;
      controls.keyPanSpeed = 7;
      controls.keys = {
        LEFT:37,
        UP:38,
        RIGHT:39,
        BOTTOM:40
      }
      this.controls =controls;

    },
    animate() {
      this.renderer.render(this.scene, this.camera);
      this.requestId = requestAnimationFrame(this.animate);
    },
    init() {
      this.group = new THREE.Group();
      this.initScene();
      this.initCamera();
      this.initRenderer();
      this.initOrbitControls();
    },
    /**加载模型 */
    loadPlant() {
      let that = this;
      let objLoader = new OBJLoader();
      let mtlLoader = new MTLLoader();
      mtlLoader.load("/static/jingxie/jingxie.mtl", function(materials) {
        // materials.preload();
        objLoader.setMaterials(materials);
        objLoader.load(
            "/static/jingxie/jingxie.obj",
            function(obj) {
              obj.position.set(-1000, 0,0);
              obj.scale.set(0.0001, 0.0001, 0.0001);
              that.scene.add(obj);
            },
            //   called while loading is progressing
            function(xhr) {
              console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
            },
            //   called when loading has errors
            function(error) {
              console.log("An error happened");
            }
        );
      });
    },

    loadPlant2() {
      let that = this;
      let objLoader = new OBJLoader();
      let mtlLoader = new MTLLoader();
      mtlLoader.load("/static/jingguiji/jingguiji.mtl", function(materials) {
        // materials.preload();
        objLoader.setMaterials(materials);
        objLoader.load(
            "/static/jingguiji/jingguiji.obj",
            function(obj) {
              obj.position.set(-1000, 1000,-600);
              obj.scale.set(1, 1, 1);
              that.scene.add(obj);
            },
            //   called while loading is progressing
            function(xhr) {
              console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
            },
            //   called when loading has errors
            function(error) {
              console.log("An error happened");
            }
        );
      });
    },

    loadPlant3() {
      let that = this;
      let objLoader = new OBJLoader();
      let mtlLoader = new MTLLoader();
      mtlLoader.load("/static/Hawk/Hawk.mtl", function(materials) {
        // materials.preload();
        objLoader.setMaterials(materials);
        objLoader.load(
            "/static/Hawk/Hawk.obj",
            function(obj) {
              obj.position.set(10, 10,0);
              obj.scale.set(1, 1, 1);
              that.scene.add(obj);
            },
            //   called while loading is progressing
            function(xhr) {
              console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
            },
            //   called when loading has errors
            function(error) {
              console.log("An error happened");
            }
        );
      });
    },
  },
  mounted() {
    this.renderer = "";
    this.camera = "";
    this.scene = "";
    this.light = "";
    this.init();
    this.loadPlant();
    // this.loadPlant2();
    // this.loadPlant3();
    this.animate();
    document.getElementsByTagName("canvas")[0].style.verticalAlign = "bottom"; //解决canvas底部留白问题
    // window.addEventListener("click", this.clickModel, false);
    // window.addEventListener("resize", this.onResize, false);
  },
}
</script>

<style scoped>
#academy{
  height: 600px;
}
</style>
