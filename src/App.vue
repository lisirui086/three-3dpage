<script setup>
// 引入组合式API
import { ref, onMounted } from 'vue'

// 引入THREE.js
import * as THREE from 'three'

// 引入动画库
import gsap from 'gsap'

// 引入画布节点、官网介绍节点
const sceneDom = ref(null)
const pages = ref(null)

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
    scene.background = texture,
    scene.environment = texture
  })

  // 渲染函数
  function animate () {
    requestAnimationFrame(animate)
    renderer.render( scene, camera )
  }

  animate()
})


</script>

<template>
  <div class='home'>

    <!-- 画布 -->
    <div class='canvas-container' ref='sceneDom'></div>

    <!-- 首页，官网地址 -->
    <div class="header">
      <div class="logo"></div>
      <div class="menu">
        <a href="" class="menuItem"></a>
      </div>
    </div>

    <!-- 官网简介 -->
    <div class="pages" ref="pages">
      <div class="page">
        <h2 class="title"></h2>
        <p></p>
      </div>
    </div>
  </div>
</template>

<style scoped lang='less'>
  .home {
    width: 100vw;
    height: 100vh;
    transform-origin: 0,0;

    .canvas-container {
      width: 100vw;
      height: 100vh;
    }

    .header {
      position: fixed;
      top: 0;
      left: 0;
      height: 100px;
      display: flex;
      justify-content: space-between;
      align-items: center;

      .logo {
        width: 300px;
        background-image: url('./assets/images/loading.jpg');
        background-size: 60%;
        background-position: center;
        background-repeat: no-repeat;
      }

      .menu {
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
        justify-content: center;
        align-items: flex-start;
        color: #fff;
        padding: 15%;
        box-sizing: border-box;

        .title {
          font-size: 50px;
          font-weight: 900;
          margin-bottom: 20px;
        }

        p {
          font-size: 30px;
        }
      }
    }
  }

</style>
