<template>
  <div ref="threeRef" :style="{position:'absolute',left:0,top:0}"></div>
  <div ref="statsDivRef"></div>
</template>

<script lang="ts">
import {defineComponent, onMounted,ref} from "vue";
import {Color, SpotLight,PerspectiveCamera,
  Scene, WebGLRenderer,AxesHelper,
  PlaneGeometry,MeshLambertMaterial,Mesh,BoxGeometry,SphereGeometry}
  from 'three'
import States from 'stats.js'
export default  defineComponent({
  setup(){
    const threeRef =ref<HTMLDivElement>()
    const statsDivRef=ref<HTMLDivElement>()
    const statsRef=ref<Stats>()
    function initStats(){
      statsRef.value=new States()
      statsRef.value.showPanel(0)
      statsDivRef.value?.appendChild(statsRef.value.dom)
    }

    function init(){
      initStats()
      const scene=new Scene()
      //坐标系
      const axes=new AxesHelper(20);
      scene.add(axes);

      //底部平面
      const planeGeometry=new PlaneGeometry(60,20);
      const planeMaterial=new MeshLambertMaterial({color:0xcccccc});
      const plane=new Mesh(planeGeometry,planeMaterial);
      plane.receiveShadow=true;//接收阴影
      plane.rotation.x=-0.5*Math.PI;//旋转
      plane.position.x=0;//移动
      plane.position.y=0;
      plane.position.z=0;
      scene.add(plane)

      //方块
      const cubeGeometry=new BoxGeometry(4,4,4);
      const cubeMaterial=new MeshLambertMaterial({color:0xff0000});
      const cube=new Mesh(cubeGeometry,cubeMaterial);
      cube.castShadow=true//添加阴影
      cube.position.x=2;//移动
      cube.position.y=2;
      cube.position.z=2;
      scene.add(cube)

      //球体
      const sphereGeometry=new SphereGeometry(4,20,20);
      const sphereMaterial=new MeshLambertMaterial({color:0x7777ff});
      const sphere=new Mesh(sphereGeometry,sphereMaterial);
      sphere.castShadow=true//添加阴影
      sphere.position.x=-10;//移动
      sphere.position.y=4;
      sphere.position.z=2;
      scene.add(sphere)

      //光源
      const spotLight=new SpotLight(0xffffff)
      spotLight.position.set(-40,60,-10)
      spotLight.castShadow=true
      scene.add(spotLight)

      // 渲染
      const renderer=new WebGLRenderer();
      renderer.setClearColor(new Color(0xEEEEEE));
      renderer.setSize(window.innerWidth,window.innerHeight);
      renderer.shadowMap.enabled=true;

      // 摄像机
      const camera=new PerspectiveCamera(45,window.innerWidth/window.innerHeight,0.1,1000)
      camera.position.x=-30;
      camera.position.y=40;
      camera.position.z=30;
      camera.lookAt(scene.position);

      threeRef.value.appendChild(renderer.domElement)
      var step=0
      renderScene()
      function renderScene(){
        statsRef.value?.begin()
        //移动球体和方块
        cube.rotation.x+=0.02
        cube.rotation.y+=0.02
        cube.rotation.z+=0.02
        step += 0.04
        sphere.position.x=20+10*Math.cos(step);
        sphere.position.y=2+10*Math.abs(Math.sin(step));
        statsRef.value?.end()
        requestAnimationFrame(renderScene)//循环调用
        renderer.render(scene,camera)
      }
    }
    onMounted(()=>{
      init()
    })
    return{
      threeRef
    }
  },
})


</script>

<style scoped>

</style>