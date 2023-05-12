<script setup>
// 引入组合式API
import { ref, onMounted } from 'vue'

// 引入address info
import address from './assets/text/address'
import info from './assets/text/info'

// 引入THREE.js
import * as THREE from 'three'

// 引入动画库
import gsap from 'gsap'

// 引入 用于载入glTF 2.0资源的加载器。
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'

// 一个用Draco库压缩的几何体加载器。
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js'

// 引入画布节点、官网介绍节点
const sceneDom = ref(null)
const pages = ref(null)

// 实例化 gltf、draco
const gltfLoader = new GLTFLoader()
const dracoLoader = new DRACOLoader()

// 组件挂载完毕
onMounted(()=>{
  // 创建场景
  const scene = new THREE.Scene()

  // 创建相机
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.z = 10
  scene.add( camera )

  // 创建渲染器
  const renderer = new THREE.WebGLRenderer({
    antialias: true,
    preserveDrawingBuffer: true
  })
  renderer.setSize( window.innerWidth, window.innerHeight )
  sceneDom.value.appendChild( renderer.domElement )

  // 创建星空背景
  const starTexture = new THREE.TextureLoader()
  starTexture.load('/assets/25s.jpg', texture => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    scene.background = texture,
    scene.environment = texture
  })

  // 创建平行光
  const light1 = new THREE.DirectionalLight( 0xffffff, 1 )
  light1.position.set(0, 0, 1)

  const light2 = new THREE.DirectionalLight( 0xffffff, 0.5 )
  light2.position.set(0, 0, -1)

  const light3 = new THREE.DirectionalLight( 0xffffff, 0.5 )
  light3.position.set(-1, 1, 1)
  scene.add(light1, light2, light3)

  // 随鼠标摇摆
  function swayBymouse (event, ob) {
  const x = (event.clientX / window.innerWidth) * 2 - 1
  const y = (event.clientY / window.innerHeight) *2 - 1

  const timeline = gsap.timeline( )
  timeline.to(ob.rotation, {
    duration: 0.5,
    x: y,
    y: x,
  })
}

  // 加载模型
  dracoLoader.setDecoderPath( '/draco/gltf/' )
  gltfLoader.setDRACOLoader( dracoLoader )
  gltfLoader.load('/model/xz.glb', gltf =>{
    const xz = gltf.scene
    xz.scale.set(0.2, 0.2, 0.2)
    xz.position.set(6, 0, 0)
    scene.add( xz )

    window.addEventListener('mousemove', function (event) {
      swayBymouse(event, xz)
    })
  })

  gltfLoader.load('/model/xq6.glb', gltf =>{
    const xq6 = gltf.scene
    xq6.scale.set(0.1, 0.1, 0.1)
    xq6.position.set(6, -20, 0)
    scene.add( xq6 )

    window.addEventListener('mousemove', function (event) {
      swayBymouse(event, xq6)
    })
  })

  gltfLoader.load('/model/gr75.glb', gltf =>{
    const gr75 = gltf.scene
    // gr75.scale.set(0.1, 0.1, 0.1)
    gr75.position.set(6, -40, 0)
    scene.add( gr75 )

    window.addEventListener('mousemove', function (event) {
      swayBymouse(event, gr75)
    })
  })

  // 加载月亮模型
  gltfLoader.load('/model/moon.glb', gltf => {
    const moon = gltf.scene.children[0]
    const moonInstanced = new THREE.InstancedMesh(
      moon.geometry,
      moon.material,
      1000
    )
    
    // 批量设定定位
    for (let i = 0; i < 1000; i++) {
      const x = Math.random() * 1000 - 500
      const y = Math.random() * 1000 - 500
      const z = Math.random() * 1000 - 500

      const size = Math.random() * 10 - 8

      const matrix = new THREE.Matrix4()
      matrix.makeScale(size, size, size)
      matrix.makeTranslation(x, y, z)
      moonInstanced.setMatrixAt(i, matrix)
    }
    scene.add(moonInstanced)

    gsap.to(moonInstanced.position, {
      duration: Math.random() * 10 + 2,
      z: -1000,
      ease: 'linear',
      repeat: -1
    })
  })

  // 渲染函数
  function animate () {
    requestAnimationFrame(animate)
    renderer.render( scene, camera )
  }

  // 侦听屏幕宽高变化，调整相机、渲染器宽高大小
  function handResize () {
    camera.acpect = window.innerWidth / window.innerHeight,
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }

  let count = 0
  function handkeyup (event) {
    const timeline2 = gsap.timeline()
    const timeline3 = gsap.timeline()
    
    if(event.code === 'Space' ) {
      count++
      if(count > 2) count = 0
    }

    // 控制相机位置
    timeline2.to(camera.position, {
      duration: 0.5,
      y: count * -20,
      duration: 1
    })

    // 控制对应官网简介切换
    timeline3.to(pages.value, {
      duration: 0.5,
      y: -count * window.innerHeight,
      duration: 1
    })
    
  }


  animate()

  window.addEventListener('resize', handResize)

  window.addEventListener('keyup', function (event) {
    handkeyup(event)
  })
})


</script>

<template>
  <div class='home'>

    <!-- 画布 -->
    <div class='canvas-container' ref='sceneDom'></div>

    <!-- 首页，官网地址 -->
    <div class="header">
      <div class="logo"></div>
      <div class="menu" >
        <a target="_blank" v-for="(item, index) in address" :key='index' :href="item.targetUrl" class="menuItem">{{ item.name }}</a>
      </div>
    </div>

    <!-- 官网简介 -->
    <div class="pages" ref="pages">
      <div class="page" v-for='(item, index) in info' :key='index'>
        <h2 class="title">{{ item.title }}</h2>
        <p>{{ item.info }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped lang='less'>
  .home {
    width: 100vw;
    height: 100vh;
    transform-origin: center;

    .canvas-container {
      width: 100vw;
      height: 100vh;
    }

    .header {
      position: fixed;
      top: 0;
      left: 0;
      height: 100px;
      width: 100vw;
      display: flex;
      display: flex;
      justify-content: space-between;
      align-items: center;
      z-index:999;

      .logo {
        height: 100px;
        width: 300px;
        background-image: url('./assets/images/logo.png');
        background-size: 60%;
        background-position: center;
        background-repeat: no-repeat;
      }

      .menu {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-right: 50px;

        .menuItem {
          padding: 0 15px;
          text-decoration: none;
          color: #fff;
          font-weight: 900;
          font-size: 15px;
        }
      }
    }

    .pages {
      display: flex;
      flex-direction: column;
      position: fixed;
      top: 0;
      left: 0;

      .page {
        width: 100vw;
        height: 100vh;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: flex-start;
        color: #fff;
        padding: 5%;
        box-sizing: border-box;

        .title {
          font-size: 45px;
          font-weight: 900;
          margin-bottom: 20px;
        }

        p {
          font-size: 18px;
        }
      }
    }
  }

</style>
