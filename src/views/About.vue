<template>
  <div class="home">
    <div id="factory"></div>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
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
      BACKGROUND_COLOR: "#010723",
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
      this.camera.position.set(800, 600, 350);
      this.camera.lookAt(new THREE.Vector3(0, 0, 0));

      this.scene = new THREE.Scene();

      this.setLight();
      // this.scene.add(new THREE.AxesHelper(1000));

      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      this.renderer.setClearAlpha(0.2);
      this.renderer.setSize(window.innerWidth, window.innerHeight);

      // this.controls = new OrbitControls(this.camera, this.renderer.domElement);

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
    setLight() {
      // const ambientLight = new THREE.AmbientLight(0xcccccc, 0.4);
      // this.scene.add(ambientLight);
      const ambientLight = new THREE.AmbientLight(0xcccccc, 1);
      this.scene.add(ambientLight);
      const pointLight = new THREE.PointLight(0xffffff, 1);
      pointLight.position.set(1000, 1000, 0);
      this.scene.add(pointLight);

      // const pointLight = new THREE.PointLight(0xffffff, 1);
      // pointLight.position.set(-2458, 1003, 786);
      // this.scene.add(pointLight);

      // const pointLight2 = new THREE.PointLight(0xffffff, 1);
      // pointLight2.position.set(-887, 698, -885);
      // this.scene.add(pointLight2);

      // const pointLight3 = new THREE.PointLight(0xffffff, 1);
      // pointLight3.position.set(240, 74, -833);
      // this.scene.add(pointLight3);

      // const pointLight4 = new THREE.PointLight(0xffffff, 1);
      // pointLight4.position.set(45, 627, 799);
      // this.scene.add(pointLight4);
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
            group.position.z = 100;
            group.position.x = 450;
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
      if (intersects.length > 0) {
        let mesh = intersects[0].object;
        let name = mesh.name.substr(0, 3);
        if (["厂房一", "厂房二", "厂房三", "厂房四"].includes(name)) {
          this.restore();
          this.scene.children[2].children.forEach((item) => {
            if (item.name == name) {
              this.INTERSECTED = item;
            }
          });
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
      this.INTERSECTED &&
        this.INTERSECTED.children.forEach((item) => {
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
      val.x = val.x + 450;
      val.z = val.z + 100;
      let world_vector = new THREE.Vector3(val.x,val.y,val.z)
      let vector = world_vector.project(this.camera)
      let halfWidth = window.innerWidth / 2
      let halfHeight = window.innerHeight / 2;
      this.left = Math.round(vector.x * halfWidth + halfWidth)
      this.top = Math.round( -vector.y * halfHeight + halfHeight)
      this.name = val;
    },
  },
};
</script>

<style scoped>
.infoBox {
  position: absolute;
  width: 165px;
  height: 80px;
  background-color: #000a2173;
  z-index: 99;
  top: 0;
  color: #fff;
  font-size:12px;
}

.home {
  margin: 0;
  overflow: hidden;
  background-image: url("../assets/ground/ground.jpg");
  background-size: 100% 100%;
  background-repeat: no-repeat;
}
</style>