<template>
  <div id="container" class="absolute text-white text-center" style="top: 50%; transform: translate(-50%, -50%); left: 50%;" >
        <a href="https://www.keithsinclair.co.za/" target="_blank"><img src="/logo.png" alt="My Color Logo" /></a>
        <a href="https://www.keithsinclair.co.za/" target="_blank"><h1 id="myName" class="font-roboto text-4xl  pt-2 opacity-0">Keith Sinclair</h1></a>
        <div class="absolute max-w-4xl p-4 rounded-lg shadow opacity-0" id="myName" > 
        <div class="h-3 text-5xl text-left text-white-600 mb-2 opacity-0" id="myParagraph">“</div>        
        <p class="font-exo opacity-0" id="myParagraph">We cannot live better than in seeking to become better</p>
        <div class="h-3 text-5xl text-right text-white-600 opacity-0" id="myParagraph">”</div>
        <a href="https://www.keithsinclair.co.za/" target="_blank" class="border px-4 py-2 rounded-full text-sm mt-8  inline-block opacity-0 bg-gradient-to-r from-green-400 to-blue-500 hover:from-pink-500 hover:to-yellow-500 font-exo uppercase" id="myWorkBtn" >View My Work</a>
      </div>             
    </div>
</template>

<script>
import { PlaneGeometry, BufferAttribute, Scene, PerspectiveCamera, WebGLRenderer, MeshPhongMaterial, DoubleSide, FlatShading, Mesh, Raycaster, DirectionalLight, AmbientLight, BufferGeometry, PointsMaterial, Float32BufferAttribute, Points } from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import gsap from 'gsap'
export default {
  mounted(){
const dat = require('dat.gui')

const gui = new dat.GUI()
const world = {
  plane: {
    width: 400,
    height: 400,
    widthSegments: 50,
    heightSegments: 50
  }
}

gui.add(world.plane, 'width', 1, 800).step(1).onChange(generatePlane)
gui.add(world.plane, 'height', 1, 800).step(1).onChange(generatePlane)
gui.add(world.plane, 'widthSegments', 1, 100).step(1).onChange(generatePlane)
gui.add(world.plane, 'heightSegments', 1, 100).step(1).onChange(generatePlane)

function generatePlane(){
  planeMesh.geometry.dispose()
  planeMesh.geometry = new PlaneGeometry(world.plane.width,world.plane.height,world.plane.widthSegments,world.plane.heightSegments)

  //Vertice Position Randomization
  const { array } = planeMesh.geometry.attributes.position;
  const randomValues = []
  for (let i = 0; i < array.length; i++) {
    if (i % 3 === 0) {
      const x = array[i]
    const y = array[i + 1]
    const z = array[i + 2]

    array[i]= x + (Math.random() - 0.5) * 3
    array[i + 1]= y + (Math.random() - 0.5) * 3
    array[i + 2]= z + (Math.random() - 0.5) * 5
    }
    
    randomValues.push(Math.random() * Math.PI * 2)
  }

  planeMesh.geometry.attributes.position.randomValues = randomValues
  planeMesh.geometry.attributes.position.originalPosition = planeMesh.geometry.attributes.position.array
  
  
  const colors = []
  for (let i = 0; i < planeMesh.geometry.attributes.position.count; i++) {
    colors.push(0,0.19,0.4)
  }
  planeMesh.geometry.setAttribute('color', new BufferAttribute(new Float32Array(colors), 3))  
  } 

const scene = new Scene()
const camera = new PerspectiveCamera(75, innerWidth / innerHeight, 0.1, 1000)
const renderer = new WebGLRenderer()

new OrbitControls(camera, renderer.domElement)

renderer.setSize(innerWidth, innerHeight)
renderer.setPixelRatio(devicePixelRatio)
document.body.appendChild(renderer.domElement)


const planeGeometry = new PlaneGeometry(world.plane.width,world.plane.height,world.plane.widthSegments,world.plane.heightSegments)
const planeMaterial = new MeshPhongMaterial({ side: DoubleSide, flatShading: FlatShading, vertexColors: true})
const planeMesh = new Mesh(planeGeometry, planeMaterial)
generatePlane()

const raycaster = new Raycaster()

const light = new DirectionalLight(0xffffff, 1)
light.position.set(0,1,1)


const backLight = new DirectionalLight(0xffffff, 1)
backLight.position.set(0,-1,-1)


const ambientLight = new AmbientLight(0x404040)

scene.add(planeMesh, light, backLight, ambientLight)
camera.position.set(0,0,105)

const starGeometry = new BufferGeometry()
const starMaterial = new PointsMaterial({color: 0xffffff})

const starVertices = []
for (let i = 0; i < 10000; i++) {  
  const x = (Math.random() - 0.5) * 2000
  const y = (Math.random() - 0.5) * 2000
  const z = (Math.random() - 0.5) * 2000
  starVertices.push(x, y, z)
}

starGeometry.setAttribute('position', new Float32BufferAttribute(starVertices, 3))

const stars = new Points(starGeometry, starMaterial)

scene.add(stars)

const mouse = {
  x: undefined,
  y: undefined
}

let frame = 0
function animate(){
  requestAnimationFrame(animate)
  renderer.render(scene, camera)    
  raycaster.setFromCamera(mouse, camera)
  frame += 0.01 
  
  const { array, originalPosition, randomValues } = planeMesh.geometry.attributes.position
  for (let i = 0; i < array.length; i += 3) {
    //X
    array[i] = originalPosition[i] + Math.cos(frame + randomValues[i]) * 0.01 
    //Y
    array[i + 1] = originalPosition[i + 1] + Math.sin(frame + randomValues[i + 1]) * 0.01  
    //z
    array[i + 2] = originalPosition[i + 2] + Math.sin(frame + randomValues[i + 2]) * 0.005  
  }

  planeMesh.geometry.attributes.position.needsUpdate = true

  const intersects = raycaster.intersectObject(planeMesh)
  if (intersects.length > 0) {
    const {color} = intersects[0].object.geometry.attributes    

    const initialColor = {
      r: 0,
      g: .19,
      b: .4
    }

    const hoverColor = {
      r: 0.1,
      g: .5,
      b: 1
    }

    gsap.to(hoverColor, {
      r: initialColor.r ,
      g: initialColor.g,
      b: initialColor.b,
      onUpdate: () => {
        //Vertice 1
      color.setX(intersects[0].face.a,hoverColor.r)
      color.setY(intersects[0].face.a,hoverColor.g)
      color.setZ(intersects[0].face.a,hoverColor.b)

      // Vertice 2
      color.setX(intersects[0].face.b,hoverColor.r)
      color.setY(intersects[0].face.b,hoverColor.g)
      color.setZ(intersects[0].face.b,hoverColor.b)

      //Vertice 3
      color.setX(intersects[0].face.c,hoverColor.r)
      color.setY(intersects[0].face.c,hoverColor.g)
      color.setZ(intersects[0].face.c,hoverColor.b)
      color.needsUpdate = true
      }
    })

    intersects[0].object.geometry.attributes.color.needsUpdate = true

  }
  stars.rotation.x += 0.0002
}

animate()


addEventListener('mousemove', (event) => {
  mouse.x = (event.clientX / innerWidth) * 2 - 1
  mouse.y = -(event.clientY / innerHeight) * 2 + 1  
})

gsap.to('#myName', {opacity: 1, duration: 2.5, y: 0, ease: 'expo'})

gsap.to('#myParagraph', {opacity: 1, duration: 3, delay: 0.5, x:0, ease: 'expo'})

gsap.to('#myWorkBtn', {opacity: 1, duration: 3, delay: 1, x:0, ease: 'expo'})

document.querySelector('#myWorkBtn').addEventListener('click', (e) => {
  e.preventDefault()
  gsap.to('#container', {
    opacity: 0
  })

  gsap.to(camera.position, {
    z: 25,
    ease: 'power3.inOut',
    duration: 2
  })

  gsap.to(camera.rotation, {
    x: 1.57,
    ease: 'power3.inOut',
    duration: 2
  })

  gsap.to(camera.position, {
    y: 1000,
    ease: 'power3.in',
    duration: 1.5,
    delay: 2,
    onComplete: () => {
      
    }
  })
})

addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(innerWidth, innerHeight)
})

  }
}
</script>

<style>
body {
  margin: 0;
  -webkit-font-smoothing: antialiased;
}

.font-exo {
  font-family: "Exo 2", sans-serif;
  font-style: italic;
  text-shadow: 2px 2px 3px rgba(0, 0, 139, 0.6);
}

.font-roboto {
  font-family: "Roboto Mono", monospace;
  text-shadow: 2px 2px 3px rgba(0, 0, 139, 0.6);
  transform: translateY(3rem);
}

.font-roboto:hover {
  color: #2f4f4f;
  text-shadow: 2px 2px 3px #fff;
}

img {
  max-width: 100%;
  height: auto;
  display: block;
  margin-left: auto;
  margin-right: auto;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 139, 0.6),
    0 6px 20px 0 rgba(0, 0, 139, 0.6);
  border-radius: 3rem;
}

img:hover {
  filter: grayscale(70%);
  background: linear-gradient(
    160deg,
    #324ab2,
    rgba(255, 255, 255, 0.6),
    rgba(0, 0, 139, 0.6),
    #324ab2,
    #fff,
    rgba(0, 0, 139, 0.6)
  );
}
</style>
