<!--<link type="text/css" href="../common/OBJLoader.js" rel="stylesheet">-->

<template>
  <div id="container"></div>
  <el-slider
      v-model="value"
      show-input
      :min="1"
      :max="100"
      style="
      width: 50%;
      position: absolute;
      bottom: 10%;
      left: 50%;
      transform: translateX(-50%);
    "
      @change="onChange"
  />
</template>

<script>
import * as THREE from 'three'
import Stats from 'stats.js'
import * as Dat from 'dat.gui'
import TrackballControls from 'three-trackballcontrols'  //npm install three-trackballcontrols --save
// import {OBJLoader} from 'three/addons/loaders/OBJLoader.js';
// import {MTLLoader} from 'three/addons/loaders/MTLLoader.js';
import {OBJLoader} from 'three/examples/jsm/loaders/OBJLoader';
import {MTLLoader} from 'three/examples/jsm/loaders/MTLLoader';
import { useRouter } from 'vue-router'

export default {
  name: "Model3D2",
  data() {
    return {
      step: 0,
      value: 0,
    };
  },
  mounted() {
    this.router = useRouter()

    this.renderer = "";
    this.camera = "";
    this.scene = "";
    this.light = "";
    this.gui = "";
    this.axesHelper = "";
    this.stats = "";
    this.trackballControls = "";
    this.clock = "";
    this.jeepCar = "";
    this.controls = "";
    this.objLoader = "";
    this.mtlLoader = "";
    this.gltfLoader = "";
    this.plane = "";
    this.obj="",
    this.mtl=""

    // 执行
    this.execute();
    // 窗口大小变化
    window.onresize = this.onWindowResize;
  },
  methods: {
    // 初始化场景
    initScene() {
      this.scene = new THREE.Scene();
    },
    //初始化相机
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(
          45,
          window.innerWidth / window.innerHeight,
          0.1,
          20000
      );
      // 设置相机位置
      this.camera.position.x = -400;
      this.camera.position.y = 400;
      this.camera.position.z = 400;
      // 设置相机指向的位置 默认0，0，0
      this.camera.lookAt(this.scene.position);
    },
    // 初始化三维坐标系
    initHelper() {
        this.axesHelper = new THREE.AxesHelper(10000);
        this.scene.add(this.axesHelper);
    },
    initRender() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      // 告诉渲染器需要阴影效果
      this.renderer.shadowMap.enabled = true;
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      // 设置背景色
      this.renderer.setClearColor(new THREE.Color("rgb(61,61,61)"));
      document
          .querySelector("#container")
          .appendChild(this.renderer.domElement);
    },
    //初始化性能检测器
    initStats() {
      this.stats = new Stats();
      document.body.appendChild(this.stats.dom);
    },
    getWorldCenterPosition(box, scalar = 0.5) {
      return new THREE.Vector3()
          .addVectors(box.max, box.min)
          .multiplyScalar(scalar);
    },
    explodeModel(model, scalar) {
      model.traverse(function (value) {
        // @ts-ignore
        if (!value.isMesh || !value.userData.originPosition) return;
        const distance = value.userData.worldDir
            .clone()
            .multiplyScalar(value.userData.worldDistance.length() * scalar);
        const offset = new THREE.Vector3().subVectors(
            value.userData.meshCenter,
            value.userData.originPosition
        );
        const center = value.userData.explodeCenter;
        const newPos = new THREE.Vector3()
            .copy(center)
            .add(distance)
            .sub(offset);
        const localPosition = value.parent?.worldToLocal(newPos.clone());
        localPosition && value.position.copy(localPosition);
      });
    },
    initExplodeModel(modelObject) {
      if (!modelObject) return;

      // 计算模型中心
      const explodeBox = new THREE.Box3();
      explodeBox.setFromObject(modelObject);
      const explodeCenter = this.getWorldCenterPosition(explodeBox);
      const meshBox = new THREE.Box3();
      const that = this;

      // 遍历整个模型，保存数据到userData上，以便爆炸函数使用
      modelObject.traverse(function (value) {
        if (
            value.isMark ||
            value.isMarkChild ||
            value.isLine ||
            value.isSprite
        ) {
          return;
        }
        if (value.isMesh) {
          meshBox.setFromObject(value);
          const meshCenter = that.getWorldCenterPosition(meshBox);
          // 爆炸方向
          value.userData.worldDir = new THREE.Vector3()
              .subVectors(meshCenter, explodeCenter)
              .normalize();
          // 爆炸距离 mesh中心点到爆炸中心点的距离
          value.userData.worldDistance = new THREE.Vector3().subVectors(
              meshCenter,
              explodeCenter
          );
          // 原始坐标
          value.userData.originPosition = value.getWorldPosition(
              new THREE.Vector3()
          );
          // mesh中心点
          value.userData.meshCenter = meshCenter.clone();
          value.userData.explodeCenter = explodeCenter.clone();
        }
      });
    },

    initModel() {
      const router = useRouter();
      console.log("router",router)
      console.log(this.$route.query.obj)
      console.log(this.$route.query.mtl)

      const obj = this.$route.query.obj;
      var objArr = obj.split('/');
      var lastObj = objArr[objArr.length-1];
      console.log("lastObj为",lastObj)
      var preObj = obj.replace(lastObj,"");
      console.log("preObj为",preObj)

      const mtl = this.$route.query.mtl;
      var mtlArr = mtl.split('/');
      var lastMtl = mtlArr[mtlArr.length-1];
      console.log("lastMtl为",lastMtl)
      var preMtl = mtl.replace(lastMtl,"");
      console.log("preMtl为",preMtl)


      // 实例化mtl loader
      this.mtlLoader = new MTLLoader();
      // 实例化obj loader
      this.objLoader = new OBJLoader();
      const that = this;

      // 火箭
     /* const modelpath="/static/Hawk/";
      const modelname="Hawk.obj";
      const mtlpath="/static/Hawk/";
      const mtlname="Hawk.mtl";*/
      const modelpath= preObj;
      const modelname= lastObj;
      const mtlpath= preMtl;
      const mtlname= lastMtl;

      //小桌子和圆盘
      // const modelpath="/static/my/";
      // const modelname="my.obj";
      // const mtlpath="/static/my/";
      // const mtlname="my.mtl";

      //地质模型
      // const modelpath="/static/jingxie/";
      // const modelname="jingxie.obj";
      // const mtlpath="/static/jingxie/";
      // const mtlname="jingxie.mtl";

      // const modelpath="/static/jingguiji/";
      // const modelname="jingguiji.obj";
      // const mtlpath="/static/jingguiji/";
      // const mtlname="jingguiji.mtl";

      this.objLoader.setPath(modelpath);
      this.mtlLoader.setPath(mtlpath);

       // 加载mtl和obj
      that.mtlLoader.load(mtlname, function(materials) {
          materials.preload();
          that.objLoader.setMaterials(materials)
            // 加载obj
          that.objLoader.load(modelname,
              function(obj) {
              console.log(obj);
              obj.position.set(0,0,0);
              // obj.position.set(1000,1000,0);
              // obj.scale.set(0.001, 0.001, 0.001);
              // obj.scale.set(0.0001, 0.0001, 0.0001);
              obj.scale.set(10, 10, 10);
              obj.children.forEach((item) => {
                item.castShadow = true;
                item.receiveShadow = true;
              });
              obj.castShadow = true;
              obj.receiveShadow = true;
              that.jeepCar = obj;
              // 添加到场景
              that.scene.add(that.jeepCar);
              // that.initExplodeModel(that.jeepCar);
          },
              function(xhr) {
                console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
              },
              //   called when loading has errors
              function(error) {
                console.log("An error happened");
              }
          )
        })


      // 只加载obj
      // this.objLoader.load(modelname, function (obj) {
      //  console.log(obj);
      //
      //   obj.traverse((node) => {
      //     node.material = new THREE.MeshLambertMaterial({
      //       color: "#72757A",
      //       emissive: "#000000",
      //       // shininess: 150
      //     });
      //   });
      //   obj.scale.set(100, 100, 100);
      //   // 给 OBJ 模型设置纹理贴图
      //   obj.children[0].material.color.set(0xccd072);
      //   // obj.children[4].material.color.set(0xe1a07f);
      //
      //   obj.children.forEach((item) => {
      //     item.castShadow = true;
      //     item.receiveShadow = true;
      //   });
      //   obj.castShadow = true;
      //   obj.receiveShadow = true;
      //
      //   that.jeepCar = obj;
      //   // 添加到场景
      //   that.scene.add(that.jeepCar);
      //   that.initExplodeModel(that.jeepCar);
      // });

      // // 创建一个几何平面，作为地板，400，400
      // const planeGeometry = new THREE.PlaneGeometry(600, 600);
      // // 创建带颜色材质,更换为MeshLambertMaterial材质，去掉网格结构
      // const planeMaterial = new THREE.MeshLambertMaterial({
      //   color: "rgb(200,110,110)",
      // });
      // this.plane = new THREE.Mesh(planeGeometry, planeMaterial);
      //
      // // 平面开启接收阴影效果
      // this.plane.receiveShadow = true;
      //
      // // 设置平面角度和位置
      // this.plane.rotation.x = -0.5 * Math.PI;
      // this.plane.position.x = 0;
      // this.plane.position.y = 0;
      // this.plane.position.z = 0;
      // // 添加平面
      // this.scene.add(this.plane);

      //   this.gltfLoader = new GLTFLoader();
      //   this.gltfLoader.load("static/gltf/scene.gltf", function (gltf) {
      //     console.log(gltf);
      //     gltf.scene.scale.set(80, 80, 80);
      //     that.jeepCar = gltf.scene;
      //     // 添加到场景
      //     that.scene.add(gltf.scene);
      //     that.initExplodeModel(gltf.scene);
      //   });
    },
    onChange(val) {
      console.log(val);
      this.explodeModel(this.jeepCar, val);
    },
    //初始化灯光
    initLight() {
      // 为场景所有物体添加基础光源
      const ambientLight = new THREE.AmbientLight(0x72757a, 2);
      this.scene.add(ambientLight);

      // 添加聚光灯光源
      const spotLight = new THREE.SpotLight(0xffffff, 2, 10000);
      spotLight.shadow.mapSize.set(2048, 2048);
      spotLight.position.set(-300, 0, -200);
      // spotLight.position.set(-0, 0, 2200);

      // 开启投影
      spotLight.castShadow = true;
      this.scene.add(spotLight);
      const anotherSpotLight = new THREE.SpotLight(0xffffff, 2, 1000);
      anotherSpotLight.shadow.mapSize.set(2048, 2048);
      anotherSpotLight.position.set(300, 0, 300);
      this.scene.add(anotherSpotLight);

      const spotLightHelper = new THREE.SpotLightHelper(spotLight);
      this.scene.add(spotLightHelper);

      //   const light = new THREE.DirectionalLight(0xe8b73b, 2, 1000); // 光源颜色
      //   light.shadow.mapSize.set(2048, 2048);
      //   light.position.set(100, 100, 0);
      //   this.scene.add(light);

      //   const directionalLightHelper = new THREE.SpotLightHelper(light);
      //   this.scene.add(directionalLightHelper);
    },
    initGui() {
      // 为带贴图MTL的OBJ模型添加UI控制（xy坐标，xyz角度）
      this.controls = {
        positionX: 0,
        positionY: 0,
        positionZ: 0,
        rotationX: 0,
        rotationY: 0,
        rotationZ: 0,
      };

      const gui = new Dat.GUI();
      // 设置允许操作范围
      gui.add(this.controls, "positionX", -500, 500);
      gui.add(this.controls, "positionY", -500, 500);
      gui.add(this.controls, "positionZ", -500, 500);
      gui.add(this.controls, "rotationX", -360, 360);
      gui.add(this.controls, "rotationY", -360, 360);
      gui.add(this.controls, "rotationZ", -360, 360);
    },
    //初始化控制器
    initControls() {
      this.trackballControls = new TrackballControls(
          this.camera,
          this.renderer.domElement
      );
      this.trackballControls.rotateSpeed = 1.0;
      this.trackballControls.zoomSpeed = 1;
      this.trackballControls.panSpeed = 1;
      this.trackballControls.noZoom = false;
      this.trackballControls.noPan = false;
      this.trackballControls.staticMoving = true;
      this.trackballControls.dynamicDampingFactor = 0.3;
      this.trackballControls.keys = [65, 83, 68];
    },
    // 初始化时钟工具
    initClock() {
      this.clock = new THREE.Clock();
    },
    render() {
      this.trackballControls.update(this.clock.getDelta());
      this.stats.update();

      // UI事件
      if (this.jeepCar) {
        this.jeepCar.position.x = this.controls.positionX;
        this.jeepCar.position.y = this.controls.positionY;
        this.jeepCar.position.z = this.controls.positionZ;

        // 这里设置的角度，需要转换为弧度
        this.jeepCar.rotation.x = this.angle2Radian(this.controls.rotationX);
        this.jeepCar.rotation.y = this.angle2Radian(this.controls.rotationY);
        this.jeepCar.rotation.z = this.angle2Radian(this.controls.rotationZ);
      }

      // render using requestAnimationFrame
      requestAnimationFrame(this.render);
      this.renderer.render(this.scene, this.camera);
    },
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    // 角度转弧度（弧度 = π / 180 * 角度）
    angle2Radian(angle) {
      return (Math.PI / 180) * angle;
    },
    execute() {
      // 初始化场景
      this.initScene();
      // 初始化摄像头
      this.initCamera();
      // 初始化三维坐标系
      this.initHelper();
      // 初始化辅助UI
      this.initGui();
      // 初始化帧数显示工具
      this.initStats();
      // 初始化时钟工具
      this.initClock();
      // 初始化模型
      this.initModel();
      // 初始化渲染器
      this.initRender();
      // 初始化光源
      this.initLight();
      // 初始化控制器
      this.initControls();
      // 执行渲染
      this.render();
    },
  },
};
</script>