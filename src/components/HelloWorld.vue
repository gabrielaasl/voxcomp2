<template>
  <div>
    <!-- <barra-herramientas /> -->
    <button @click="exportSceneObject()">descargar</button>
  </div>
</template>
<script>
// import ButtonCounter from "./ButtonCounter.vue";
// import BarraHerramientas from "../components/BarraHerramientas.vue";

//import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import * as THREE from "three";
import { GLTFExporter } from "three/examples/jsm/exporters/GLTFExporter.js";

// import { saveAs } from 'file-saver';
// import * as CSG from 'csg';

export default {
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      plane: null,
      pointer: null,
      raycaster: null,
      isShiftDown: false,
      rollOverMaterial: null,
      rollOverMesh: null,
      cubeGeo: null,
      cubeMaterial: null,
      objects: [],
      controls: null,
      params: {
        trs: false,
        onlyVisible: true,
        truncateDrawRange: true,
        binary: false,
        maxTextureSize: 4096,
        // exportScene1: exportScene1,
        // exportScenes: exportScenes,
        // exportSphere: exportSphere,
        // exportHead: exportHead,
        // exportObjects: exportObjects,
        // exportSceneObject: exportSceneObject
      },
    };
  },
  // components: { BarraHerramientas},
  methods: {
    init: function () {
      // let container = document.getElementById("container");
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.01,
        3000
      );

      this.camera.position.set(500, 800, 1300);
      this.camera.lookAt(new THREE.Vector3(0, 0, 0));

      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0xf0f0f0);

      // roll-over helpers

      const rollOverGeo = new THREE.BoxGeometry(50, 50, 50);
      this.rollOverMaterial = new THREE.MeshBasicMaterial({
        color: 0xff0000,
        opacity: 0.5,
        transparent: true,
      });
      this.rollOverMesh = new THREE.Mesh(rollOverGeo, this.rollOverMaterial);
      this.scene.add(this.rollOverMesh);

      // cubes

      this.cubeGeo = new THREE.BoxGeometry(50, 50, 50);

      this.cubeMaterial = new THREE.MeshPhongMaterial({
        color: 0xf3f3f3,
        // wireframe: true,
      });

      // Declaracion de la grilla
      const size = 1000;
      const divisions = 20;
      this.gridHelper = new THREE.GridHelper(
        size,
        divisions,
        0x39ff14,
        0x39ff14
      );
      this.scene.add(this.gridHelper);

      this.raycaster = new THREE.Raycaster();
      this.pointer = new THREE.Vector2();

      // Plane
      const geometry = new THREE.PlaneGeometry(1000, 1000);
      geometry.rotateX(-Math.PI / 2);

      this.plane = new THREE.Mesh(
        geometry,
        new THREE.MeshBasicMaterial({ visible: false })
      );
      this.scene.add(this.plane);
      this.objects.push(this.plane);

      // lights

      // const ambientLight = new THREE.AmbientLight(0xffffff, 2);
      // this.scene.add(ambientLight);

      const directionalLight = new THREE.DirectionalLight(0xffffff, 2);
      directionalLight.position.set(1, 0.75, 0.5);
      this.scene.add(directionalLight);

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(this.renderer.domElement);
      //this.controls = new OrbitControls(this.camera, this.renderer.domElement);

      document.addEventListener("pointermove", this.onPointerMove);
      document.addEventListener("pointerdown", this.onPointerDown);
      document.addEventListener("keydown", this.onDocumentKeyDown);
      document.addEventListener("keyup", this.onDocumentKeyUp);
    },
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();

      this.renderer.setSize(window.innerWidth, window.innerHeight);

      // this.animate();
    },
    onDocumentKeyDown(event) {
      switch (event.keyCode) {
        case 16:
          this.isShiftDown = true;
          break;
      }
    },

    onDocumentKeyUp(event) {
      switch (event.keyCode) {
        case 16:
          this.isShiftDown = false;
          break;
      }
    },
    onPointerMove(event) {
      this.pointer.set(
        (event.clientX / window.innerWidth) * 2 - 1,
        -(event.clientY / window.innerHeight) * 2 + 1
      );

      this.raycaster.setFromCamera(this.pointer, this.camera);

      const intersects = this.raycaster.intersectObjects(this.objects, false);

      if (intersects.length > 0) {
        const intersect = intersects[0];

        this.rollOverMesh.position
          .copy(intersect.point)
          .add(intersect.face.normal);
        this.rollOverMesh.position
          .divideScalar(50)
          .floor()
          .multiplyScalar(50)
          .addScalar(25);

        // this.animate();
      }
    },
    onPointerDown(event) {
      this.pointer.set(
        (event.clientX / window.innerWidth) * 2 - 1,
        -(event.clientY / window.innerHeight) * 2 + 1
      );

      this.raycaster.setFromCamera(this.pointer, this.camera);

      const intersects = this.raycaster.intersectObjects(this.objects, false);

      if (intersects.length > 0) {
        const intersect = intersects[0];

        // delete cube

        if (this.isShiftDown) {
          if (intersect.object !== this.plane) {
            this.scene.remove(intersect.object);

            this.objects.splice(this.objects.indexOf(intersect.object), 1);
          }

          // create cube
        } else {
          const voxel = new THREE.Mesh(this.cubeGeo, this.cubeMaterial);

          voxel.position.copy(intersect.point).add(intersect.face.normal);
          voxel.position
            .divideScalar(50)
            .floor()
            .multiplyScalar(50)
            .addScalar(25);
          console.log(voxel.position);
          this.scene.add(voxel);

          // const cubeMaterial2 = new THREE.MeshPhongMaterial({
          //   color: 0x000000,
          //   wireframe: true,
          // });
          // const contorno = new THREE.Mesh(this.cubeGeo, cubeMaterial2);
          // contorno.position.copy(voxel.position);
          // this.scene.add(contorno);

          const box = new THREE.BoxHelper(voxel, 0x000000);
          this.scene.add(box);

          this.objects.push(voxel);
        }

        // this.animate();
      }
    },
    export() {
      // Instantiate a exporter
      const exporter = new GLTFExporter();
      const options = {
        trs: this.params.trs,
        onlyVisible: this.params.onlyVisible,
        truncateDrawRange: this.params.truncateDrawRange,
        binary: this.params.binary,
        maxTextureSize: this.params.maxTextureSize,
      };

      // Parse the input and generate the glTF output
      exporter.parse(
        this.scene,
        // called when the gltf has been generated
        function (gltf) {
          console.log(gltf);
          if (gltf instanceof ArrayBuffer) {
            this.saveArrayBuffer(gltf, "scene.glb");
          } else {
            const output = JSON.stringify(gltf, null, 2);
            // console.log(output);

            // this.saveString(output, "scene.gltf");
            // new Blob([output], { type: "text/plain" }), "scene.gltf";

            const link = document.createElement("a");
            link.style.display = "none";
            document.body.appendChild(link); // Firefox workaround, see #6594
            link.href = URL.createObjectURL(
              new Blob([output], { type: "text/plain" })
            );
            link.download = "scene.gltf";
            link.click();
          }
        },
        // called when there is an error in the generation
        function (error) {
          console.log("An error happened", error);
        },
        options
      );
    },
    save(blob, filename) {
      const link = document.createElement("a");
      link.style.display = "none";
      document.body.appendChild(link); // Firefox workaround, see #6594
      link.href = URL.createObjectURL(blob);
      link.download = filename;
      link.click();

      // URL.revokeObjectURL( url ); breaks Firefox...
    },

    saveString(text, filename) {
      console.log(text, filename);
      this.save(new Blob([text], { type: "text/plain" }), filename);
    },

    saveArrayBuffer(buffer, filename) {
      this.save(
        new Blob([buffer], { type: "application/octet-stream" }),
        filename
      );
    },
    exportSceneObject() {
      this.export([this.scene]);
    },
    animate: function () {
      requestAnimationFrame(this.animate);
      this.onWindowResize();
      this.renderer.render(this.scene, this.camera);
    },
  },
  mounted() {
    this.init();
    this.animate();
    // console.log(this.objects);
    // console.log(this.scene);
  },
};
</script>

<style scoped>
.menu {
  position: absolute;
}
#container {
  height: 100vh;
  width: 100vw;
}
#insideText {
  position: absolute;
  left: 20px;
  top: 20px;
}
</style>
