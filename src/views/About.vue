<template>
  <div class="home">
    <div id="factory"></div>
    <div class="infoBox" :style="{ left: left, top: top }" v-if="name !== ''">
      {{ name }}
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { MapControls  } from "three/examples/jsm/controls/OrbitControls";
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader";
import { MTLLoader } from "three/examples/jsm/loaders/MTLLoader";
import { DDSLoader } from "three/examples/jsm/loaders/DDSLoader.js";
export default {
  name: "factory",
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      light: null,
      BACKGROUND_COLOR: "rgb(120, 120, 120)",
      raycaster: null,
      pointer: {},
      mouse: {},
      INTERSECTED: null,
      left: 0,
      top: 0,
      name: "",
    };
  },
  mounted() {
    this.init();
    document.addEventListener("click", this.onPointerMove);
  },
  beforeDestroy() {
    document.removeEventListener("click", this.onPointerMove);
  },
  methods: {
    init() {
      let factory = document.getElementById("factory");

      this.camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        1,
        3000
      );
      this.camera.position.set(1000, 800, 0);
      this.camera.lookAt(new THREE.Vector3(0, 0, 0));

      this.scene = new THREE.Scene();

      const ambientLight = new THREE.AmbientLight(0xcccccc, 0.4);
      this.scene.add(ambientLight);

      const pointLight = new THREE.PointLight(0xffffff, 0.8);
      this.camera.add(pointLight);
      this.scene.add(this.camera);

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setClearColor(this.BACKGROUND_COLOR);
      this.renderer.setSize(window.innerWidth, window.innerHeight);

      this.controls = new MapControls(this.camera, this.renderer.domElement);

      // this.scene.add(new THREE.AxesHelper(100));

      this.model();

      this.raycaster = new THREE.Raycaster();

      factory.appendChild(this.renderer.domElement);

      this.animate();

      window.addEventListener("resize", this.onWindowResize);
    },
    render() {
      this.renderer.render(this.scene, this.camera);
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.render();
    },
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();

      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    model() {
      const manager = new THREE.LoadingManager();
      manager.addHandler(/\.dds$/i, new DDSLoader());

      new MTLLoader(manager).load("/city.mtl", (materials) => {
        materials.preload();
        new OBJLoader(manager).setMaterials(materials).load(
          "/city.obj",
          (object) => {
            const group = new THREE.Group();
            const group1 = new THREE.Group();
            group1.name = "厂房一";
            const group2 = new THREE.Group();
            group2.name = "厂房二";
            const group3 = new THREE.Group();
            group3.name = "厂房三";
            const group4 = new THREE.Group();
            group4.name = "厂房四";
            const group5 = new THREE.Group();
            group5.name = "非厂房区";

            group.add(group1);
            group.add(group2);
            group.add(group3);
            group.add(group4);
            group.add(group5);

            let arr1 = [],
              arr2 = [],
              arr3 = [],
              arr4 = [],
              arr5 = [];

            object.children.forEach((item) => {
              let name = item.name.substr(0, 3);
              if (name == "厂房一") {
                arr1.push(item);
              } else if (name == "厂房二") {
                arr2.push(item);
              } else if (name == "厂房三") {
                arr3.push(item);
              } else if (name == "厂房四") {
                arr4.push(item);
              } else {
                arr5.push(item);
              }
            });

            group1.add(...arr1);
            group2.add(...arr2);
            group3.add(...arr3);
            group4.add(...arr4);
            group5.add(...arr5);
            group.position.x = 500;

            this.scene.add(group);
          },
          function (xhr) {
            if (xhr.lengthComputable) {
              const percentComplete = (xhr.loaded / xhr.total) * 100;
              console.log(Math.round(percentComplete, 2) + "% downloaded");
            }
          }
        );
      });
    },
    onPointerMove(event) {
      event.preventDefault();
      this.mouse.x = event.clientX;
      this.mouse.y = event.clientY;
      this.pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
      this.pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;
      this.Fire();
    },
    Fire() {
      this.raycaster.setFromCamera(this.pointer, this.camera);
      const intersects = this.raycaster.intersectObjects(
        this.scene.children,
        true
      );
      console.log("this.scene.children", this.scene.children);
      console.log("intersects", intersects);
      if (intersects.length > 0) {
        let mesh = intersects[0].object;
        let name = mesh.name.substr(0, 3);
        if (["厂房一", "厂房二", "厂房三", "厂房四"].includes(name)) {
          this.restore();
          this.getInfo(name);
          this.scene.children[2].children.forEach((item) => {
            if (item.name == name) {
              this.INTERSECTED = item;
            }
          });
          console.log("this.INTERSECTED", this.INTERSECTED);
          this.INTERSECTED.children.forEach((item) => {
            if (Array.isArray(item.material)) {
              item.material[0] = item.material[0].clone();
              item.material[1] = item.material[1].clone();
              this.INTERSECTED.currentHexArr = [];
              this.INTERSECTED.currentHexArr[0] = item.material[0].emissive.getHex();
              this.INTERSECTED.currentHexArr[1] = item.material[1].emissive.getHex();
              item.material[0].emissive.setHex(0x147646);
              item.material[1].emissive.setHex(0x37c499);
            } else {
              item.material = item.material.clone();
              this.INTERSECTED.currentHex = item.material.emissive.getHex();
              item.material.emissive.setHex(0x147646);
            }
          });
        } else {
          this.restore();
        }
      } else {
        this.restore();
      }
    },
    restore() {
      this.INTERSECTED && this.INTERSECTED.children.forEach((item) => {
        if (Array.isArray(item.material)) {
          item.material[0].emissive.setHex(this.INTERSECTED.currentHexArr[0]);
          item.material[1].emissive.setHex(this.INTERSECTED.currentHexArr[1]);
        } else {
          item.material.emissive.setHex(this.INTERSECTED.currentHex);
        }
      });
      this.INTERSECTED = null;
      this.name = "";
    },
    getInfo(val) {
      this.top = this.mouse.y + "px";
      this.left = this.mouse.x + "px";
      this.name = val;
    },
  },
};
</script>

<style scoped>
.infoBox {
  position: absolute;
  width: 239px;
  height: 115px;
  background-color: #000a2173;
  z-index: 99;
  top: 0;
  color: #fff;
}
</style>