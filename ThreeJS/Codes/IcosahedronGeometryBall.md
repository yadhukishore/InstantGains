
`index.html`
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Getting Started with Three.js</title>
    <style>
      body {
        margin: 0;
      }
    </style>
    <script type="importmap">
			{
				"imports": {
					"three": "https://cdn.jsdelivr.net/npm/three@0.161.0/build/three.module.js",
          "jsm/": "https://cdn.jsdelivr.net/npm/three@0.161.0/examples/jsm/"
        }
			}
		</script>
  </head>
  <body>
    <script type="module" src="index.js"></script>
  </body>
</html>

```


`index.js`:-
```javascript
import * as THREE from 'three';
import { OrbitControls } from "jsm/controls/OrbitControls.js";

const w= window.innerWidth;
const h= window.innerHeight;
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(w,h);
document.body.appendChild(renderer.domElement);

const fov = 75;
const aspect = w/h;
const near = 0.1;
const far =10;
const camera = new THREE.PerspectiveCamera(fov,aspect,near,far);
camera.position.z = 2;
const scene = new THREE.Scene();

const controls = new OrbitControls(camera,renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.03;

const geo = new THREE.IcosahedronGeometry(1.0,2);
const mat = new THREE.MeshStandardMaterial({
    color:0xfffffff,
    flatShading:true,
});
const mesh = new THREE.Mesh(geo,mat);
scene.add(mesh);

const hemiLight = new THREE.HemisphereLight(0x0099ff,0xaa5500);
scene.add(hemiLight);

const wireMat = new THREE.MeshBasicMaterial({
    color:0xfffffff,
    wireframe:true,
});

const wireMesh = new THREE.Mesh(geo,wireMat);
wireMesh.scale.setScalar(1.0002);
mesh.add(wireMesh);

function animate(t=0){
    requestAnimationFrame(animate);
    mesh.rotation.y =t *0.0001;
    renderer.render(scene,camera);
    controls.update();
}
animate();

renderer.render(scene,camera);
```




![image](https://github.com/user-attachments/assets/7a9dbad1-3103-4535-9f04-4d567f1d3365)
