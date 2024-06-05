<template>
  <div ref="sceneEl" class="w-100 h-100"></div>
</template>

<script>
  import { onMounted } from 'vue';
  import { ref } from 'vue';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
  import {CSS2DRenderer, CSS2DObject} from 'three/addons/renderers/CSS2DRenderer';
    export default {
      setup() {
        const sceneEl = ref(null)

        onMounted(()=>{
          const scene = new THREE.Scene();
          const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 100 );

          const renderer = new THREE.WebGLRenderer();
          renderer.setSize( window.innerWidth, window.innerHeight );
          console.log(sceneEl.value)
          sceneEl.value.appendChild(renderer.domElement)
          
          const controls= new OrbitControls(camera, renderer.domElement);
          controls.enableDamping=true;
          controls.dampingFactor=0.03;
          
          const labelRenderer= new CSS2DRenderer();
          labelRenderer.setSize(window.innerWidth, window.innerHeight);
          labelRenderer.domElement.style.position='absolute';
          labelRenderer.domElement.style.top='0px';
          labelRenderer.domElement.style.pointerEvents='none';
          document.body.appendChild(labelRenderer.domElement);

          const group= new THREE.Group();
          
          const geometry = new THREE.IcosahedronGeometry( 3, 10);
          const texture = new THREE.TextureLoader().load("src/Img/terra.jpg")
  const material = new THREE.MeshStandardMaterial( { 
    map: texture,
    side: THREE.DoubleSide,
    //flatShading: true
  } );
  const mesh = new THREE.Mesh( geometry, material );
  scene.add( mesh );
  const wireMat= new THREE.MeshBasicMaterial({
    color: 0xffffff,
    wireframe: true
  })
  const wireMesh= new THREE.Mesh(geometry, wireMat);
  wireMesh.scale.setScalar(1.001);
  //mesh.add(wireMesh);
  camera.position.z = 8;

  const hemiLight= new THREE.HemisphereLight(0xffffff, 0x000000);
  scene.add(hemiLight);

  function createCpointMesh(name, x, y , z){
    const geo= new THREE.SphereGeometry(0.1);
    const mat= new THREE.MeshBasicMaterial({color:0xFF0000});
    const mesh= new THREE.Mesh(geo, mat);
    mesh.position.set(x,y,z);
    mesh.name=name;
    return mesh;
  }

  const sphereMesh1 =createCpointMesh('sphereMesh1', 2.45, 0.6, 1.646);
  const sphereMesh2 =createCpointMesh('sphereMesh2', -2.65, 0.6, -1.3);
  group.add(sphereMesh1);
  group.add(sphereMesh2);

  mesh.add(group);
  const p=document.createElement('p');
  p.className='tooltip';
  const pContainer=document.createElement('div')
  pContainer.appendChild(p);
  const cPointLabel= new CSS2DObject(pContainer);
  mesh.add(cPointLabel);

  const mousePos= new THREE.Vector2();
  const raycaster= new THREE.Raycaster();
  
  window.addEventListener('mousemove', function(e){
    mousePos.x=(e.clientX/this.window.innerWidth)*2-1;
    mousePos.y=-(e.clientY/this.window.innerWidth)*2+1;

    raycaster.setFromCamera(mousePos, camera);
    const intersects=raycaster.intersectObject(group);

    if (intersects.length>0) {
      switch (intersects[0].object.name) {
        case 'sphereMesh1':
          p.className= 'tooltip show';
          cPointLabel.position.set(2.45, 0.6, 1.646)
          p.textContent= 'Hotspot 1 (2.45, 0.6, 1.646)'
          break;
        case 'sphereMesh2':
          p.className= 'tooltip show';
          cPointLabel.position.set(-2.65, 0.6, -1.3)
          p.textContent= 'Hotspot 2 (-2.65, 0.6, -1.3)'
          break;
        default:
          break;
      }
    }
    else{
      p.className = 'tooltip hide'
    }
  })

  function animate() {
    requestAnimationFrame(animate);
    mesh.rotation.y += 0.001;
    labelRenderer.render(scene, camera);
    renderer.render( scene, camera );
    controls.update();
  }
  window.addEventListener('resize', function(){
    camera.aspect=this.window.innerWidth/this.window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    labelRenderer.setSize(this.window.innerWidth, this.window.innerHeight);
  })

  animate();
        })

        return {
          sceneEl
        }
      }
    }
  
</script>
