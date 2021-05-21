<template>
  <div id="home"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import Stats from "three/examples/jsm/libs/stats.module.js";
import building1 from "../assets/Textures/building1.jpg";
import building2 from "../assets/Textures/building2.jpg";
import building3 from "../assets/Textures/building3.jpg";
import building4 from "../assets/Textures/building4.jpg";
import background from "../assets/Textures/background.jpg";
export default {
  name: "Home",
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      stats: null,
      MATERIAL_COLOR: "rgb(120, 120, 120)",
      /** 其他随机建筑物的坐标位置数组 */
      randomBuildingPositions: [
        { x: -8, y: 0, z: -5 },
        { x: 19, y: 0, z: -8 },
        { x: -2, y: 0, z: -10 },
        { x: 9, y: 0, z: -10 },
        { x: 25, y: 0, z: -15 },
        { x: -3, y: 0, z: -18 },
        { x: 30, y: 0, z: -18 },
        { x: 35, y: 0, z: -20 },
        { x: 8, y: 0, z: -20 },
        { x: 20, y: 0, z: -20 },
        { x: -18, y: 0, z: -25 },
        { x: -6, y: 0, z: -25 },
        { x: 12, y: 0, z: -25 },
        { x: -3, y: 0, z: -30 },
        { x: -10, y: 0, z: -30 },
        { x: 1, y: 0, z: -30 },
        { x: 42, y: 0, z: -32 },
        { x: 36, y: 0, z: -32 },
        { x: -3, y: 0, z: -35 },
        { x: -12, y: 0, z: -35 },
        { x: -20, y: 0, z: -35 },
        { x: -16, y: 0, z: -40 },
        { x: 16, y: 0, z: -40 },
        { x: 16, y: 0, z: -40 },
        { x: 30, y: 0, z: -40 },
      ],
      /** 其他随机建筑物的贴图路径数组 */
      randomBuildingTextures: [building1, building2, building3, building4],
      buildings: null,
      loader: null,
      light: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      let home = document.getElementById("home");

      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        1,
        1000
      );
      this.camera.position.set(0, 30, 90);
      this.camera.lookAt(new THREE.Vector3(0, 0, 0));

      this.scene = new THREE.Scene();

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setClearColor(this.MATERIAL_COLOR);
      this.renderer.shadowMap.enabled = true; // 开启渲染器的阴影功能
      this.renderer.shadowMap.type = THREE.PCFShadowMap; // PCF阴影类型
      this.renderer.setSize(window.innerWidth, window.innerHeight);

      this.loader = new THREE.TextureLoader();

      this.light = new THREE.Group();
      this.scene.add(this.light);
      this.setLight();

      // this.scene.add(new THREE.AxesHelper(100));
      // this.scene.add(new THREE.GridHelper(120, 120));

      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.maxDistance = 800;

      this.scene.background = this.setBackground();

      this.setGround();
      this.scene.add(this.getOrientalPearl());

      this.buildings = new THREE.Group();
      this.buildings.name = "建筑";
      this.scene.add(this.buildings);
      this.setBuilding();

      home.appendChild(this.renderer.domElement);

      this.stats = new Stats();
      home.appendChild(this.stats.domElement);

      this.render();
    },
    render() {
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.render);
      this.stats.update();
    },
    setBackground() {
      //360全景视图
      const r = "../assets/Park3Med/";
      const urls = [
        r + "px.jpg",
        r + "nx.jpg",
        r + "py.jpg",
        r + "ny.jpg",
        r + "pz.jpg",
        r + "nz.jpg",
      ];
      const textureCube = new THREE.CubeTextureLoader().load(urls);
      textureCube.mapping = THREE.CubeRefractionMapping;
      return textureCube;
    },
    setGround() {
      let shape = new THREE.Shape();
      shape.moveTo(50, 50);
      shape.lineTo(-50, 50);
      shape.lineTo(-50, -50);
      shape.lineTo(50, -50);
      shape.lineTo(50, 50);

      let extrudeGeometry = new THREE.ExtrudeGeometry(shape, {
        depth: 3,
        steps: 2,
        bevelThickness: 0,
        bevelSize: 1,
      });

      const texture = this.loader.load(background);
      texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
      // // 设置行列的重复次数
      // texture.repeat.set(1, 1);

      let material = new THREE.MeshLambertMaterial({
        color: "#666",
        map: texture,
      });

      let mesh = new THREE.Mesh(extrudeGeometry, material);

      mesh.rotation.x = Math.PI / 2; // 地面旋转90度
      this.scene.add(mesh);
    },
    setLight() {
      const light1 = new THREE.PointLight(0xffffff, 1.2);
      light1.position.set(100, 100, 80);
      this.light.add(light1);
      const light2 = new THREE.PointLight(0xffffff, 1.2);
      light2.position.set(-100, -100, 80);
      this.light.add(light2);
      const light3 = new THREE.AmbientLight(0x404040);
      this.light.add(light3);
    },
    setBuilding() {
      let defauleLength = 16;
      // w, h, d
      for (let i = 0; i < this.randomBuildingPositions.length; i++) {
        let w = Math.random() * 3 + 2; // 随机数(2, 5);
        let d = Math.random() * 3 + 2; // 随机数(2, 5);
        let _h = Math.random() * defauleLength + 5;
        let h = _h < 8 ? _h + 8 : _h; // 随机数(0, 15.5);

        let textureInd = Math.floor(Math.random() * 4);
        let texture = this.loader.load(this.randomBuildingTextures[textureInd]);
        // // 设置贴图的矩阵重复方式
        texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
        // // 设置行列的重复次数
        texture.repeat.set(1, h / 2);

        let geometry = new THREE.BoxGeometry(w, h, d);
        let material = new THREE.MeshPhongMaterial({
          map: texture,
        });
        let mesh = new THREE.Mesh(geometry, material);
        mesh.position.set(
          this.randomBuildingPositions[i].x,
          this.randomBuildingPositions[i].y + h / 2,
          this.randomBuildingPositions[i].z
        );
        this.buildings.add(mesh);
      }
    },
    getOrientalPearl() {
      let orientalPearl = new THREE.Object3D();
      // 1. 底部圆台 2个圆柱
      let bottom = new THREE.Object3D();
      // 圆台1
      let cylinder1_geometry = new THREE.CylinderGeometry(2, 2, 0.38, 30);
      let cylinder1_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let cylinder1 = new THREE.Mesh(cylinder1_geometry, cylinder1_material);
      cylinder1.castShadow = true;
      cylinder1.receiveShadow = true;

      // 圆台2
      let cylinder2_geometry = new THREE.CylinderGeometry(1.8, 1.8, 0.2, 30);
      let cylinder2_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let cylinder2 = new THREE.Mesh(cylinder2_geometry, cylinder2_material);
      cylinder2.position.y = 0.28;
      cylinder2.castShadow = true;
      cylinder2.receiveShadow = true;

      // 底部小斜体圆柱1
      let little_bottom_cylinder1_geometry = new THREE.CylinderGeometry(
        0.15,
        0.15,
        3,
        30
      );
      let little_bottom_cylinder1_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let little_bottom_cylinder1 = new THREE.Mesh(
        little_bottom_cylinder1_geometry,
        little_bottom_cylinder1_material
      );
      little_bottom_cylinder1.position.set(1.1, 1, 0.5);
      little_bottom_cylinder1.rotation.set(
        (Math.PI / 2) * 1.15,
        (Math.PI / 5.5) * 1.5,
        (-Math.PI / 2.5) * 1.1
      );

      // 底部小斜体圆柱2
      let little_bottom_cylinder2_geometry = new THREE.CylinderGeometry(
        0.15,
        0.15,
        3,
        30
      );
      let little_bottom_cylinder2_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let little_bottom_cylinder2 = new THREE.Mesh(
        little_bottom_cylinder2_geometry,
        little_bottom_cylinder2_material
      );
      little_bottom_cylinder2.position.set(-0.05, 1, -1.35);
      little_bottom_cylinder2.rotation.set(
        (Math.PI / 2) * 1.47,
        -(Math.PI / 5.5) * 1.35,
        (Math.PI / 2.5) * 0.05
      );

      // 底部小斜体圆柱3
      let little_bottom_cylinder3_geometry = new THREE.CylinderGeometry(
        0.15,
        0.15,
        3,
        30
      );
      let little_bottom_cylinder3_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let little_bottom_cylinder3 = new THREE.Mesh(
        little_bottom_cylinder3_geometry,
        little_bottom_cylinder3_material
      );
      little_bottom_cylinder3.position.set(-1, 1, 0.8);
      little_bottom_cylinder3.rotation.set(
        -(Math.PI / 2) * 1.25,
        (Math.PI / 5.5) * 1,
        -(Math.PI / 3.5) * 0.9
      );

      // 底部支柱
      let bottom_cylinder = this.getBottomCylinder();
      bottom.add(
        cylinder1,
        cylinder2,
        bottom_cylinder,
        little_bottom_cylinder1,
        little_bottom_cylinder2,
        little_bottom_cylinder3
      );

      // 2. 中间部分 三个圆柱 + 圆球
      let body = new THREE.Object3D();

      // 圆柱1
      let body_cylinder1_geometry = new THREE.CylinderGeometry(
        0.35,
        0.35,
        15,
        30
      );
      let body_cylinder1_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let body_cylinder1 = new THREE.Mesh(
        body_cylinder1_geometry,
        body_cylinder1_material
      );
      body_cylinder1.position.set(0, 7, 0.8);
      // 圆柱2
      let body_cylinder2_geometry = new THREE.CylinderGeometry(
        0.35,
        0.35,
        15,
        30
      );
      let body_cylinder2_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let body_cylinder2 = new THREE.Mesh(
        body_cylinder2_geometry,
        body_cylinder2_material
      );
      body_cylinder2.position.set(0.7, 7, -0.5);
      // 圆柱3
      let body_cylinder3_geometry = new THREE.CylinderGeometry(
        0.35,
        0.35,
        15,
        30
      );
      let body_cylinder3_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let body_cylinder3 = new THREE.Mesh(
        body_cylinder3_geometry,
        body_cylinder3_material
      );
      body_cylinder3.position.set(-0.7, 7, -0.45);

      // 圆环
      let torus1 = this.getTorus(0.8, 0.2, 16, 10, 2);
      let torus2 = this.getTorus(0.8, 0.2, 16, 10, 7.5);
      let torus3 = this.getTorus(0.8, 0.2, 16, 10, 8.6);
      let torus4 = this.getTorus(0.8, 0.2, 16, 10, 9.7);
      let torus5 = this.getTorus(0.8, 0.2, 16, 10, 10.8);
      let torus6 = this.getTorus(0.8, 0.2, 16, 10, 11.9);
      let torus7 = this.getTorus(0.8, 0.2, 16, 10, 13);

      // 大球
      var sphere_big_geometry = new THREE.SphereGeometry(2, 32, 32);
      var sphere_big_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      var sphere_big = new THREE.Mesh(sphere_big_geometry, sphere_big_material);
      sphere_big.position.y = 5;

      // 中球
      var sphere_middle_geometry = new THREE.SphereGeometry(1.5, 32, 32);
      var sphere_middle_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
        // color: "rgb(175, 91, 123)"
      });
      var sphere_middle = new THREE.Mesh(
        sphere_middle_geometry,
        sphere_middle_material
      );
      sphere_middle.position.y = 15;

      body.add(body_cylinder1);
      body.add(body_cylinder2);
      body.add(body_cylinder3);
      body.add(torus1);
      body.add(torus2);
      body.add(torus3);
      body.add(torus4);
      body.add(torus5);
      body.add(torus6);
      body.add(torus7);
      body.add(sphere_big);
      body.add(sphere_middle);

      // 顶部
      let head = new THREE.Object3D();
      // 顶部圆柱1
      let head_cylinder1_geometry = new THREE.CylinderGeometry(
        0.3,
        0.3,
        2.5,
        5
      );
      let head_cylinder1_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let head_cylinder1 = new THREE.Mesh(
        head_cylinder1_geometry,
        head_cylinder1_material
      );
      head_cylinder1.position.y = 17.5;

      // 顶部球1
      var head_sphere_geometry = new THREE.SphereGeometry(0.6, 32, 32);
      var head_sphere_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      var head_sphere = new THREE.Mesh(
        head_sphere_geometry,
        head_sphere_material
      );
      head_sphere.position.y = 19;

      // 顶部圆柱2
      let head_cylinder2_geometry = new THREE.CylinderGeometry(0.25, 0.2, 3, 5);
      let head_cylinder2_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let head_cylinder2 = new THREE.Mesh(
        head_cylinder2_geometry,
        head_cylinder2_material
      );
      head_cylinder2.position.y = 20;

      let head_torus1 = this.getTorus(0.15, 0.15, 16, 10, 21.5);

      // 顶部圆柱3
      let head_cylinder3_geometry = new THREE.CylinderGeometry(
        0.15,
        0.15,
        2,
        10
      );
      let head_cylinder3_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let head_cylinder3 = new THREE.Mesh(
        head_cylinder3_geometry,
        head_cylinder3_material
      );
      head_cylinder3.position.y = 22.5;
      let head_torus2 = this.getTorus(0.13, 0.13, 16, 10, 23.5);

      // 顶部圆柱4
      let head_cylinder4_geometry = new THREE.CylinderGeometry(0.1, 0.1, 3, 10);
      let head_cylinder4_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let head_cylinder4 = new THREE.Mesh(
        head_cylinder4_geometry,
        head_cylinder4_material
      );
      head_cylinder4.position.y = 25;

      head.add(head_sphere, head_torus1, head_torus2, head_cylinder4);
      head.add(head_cylinder1, head_cylinder2, head_cylinder3);

      orientalPearl.add(bottom);
      orientalPearl.add(body);
      orientalPearl.add(head);
      orientalPearl.castShadow = true;
      orientalPearl.receiveShadow = true;

      return orientalPearl;
    },
    getBottomCylinder() {
      // 获取底部斜体圆柱
      let object = new THREE.Object3D();

      // 圆柱1
      let bottom_cylinder1_geometry = new THREE.CylinderGeometry(
        0.2,
        0.2,
        7,
        30
      );
      let bottom_cylinder1_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let bottom_cylinder1 = new THREE.Mesh(
        bottom_cylinder1_geometry,
        bottom_cylinder1_material
      );

      // 圆柱2
      let bottom_cylinder2_geometry = new THREE.CylinderGeometry(
        0.2,
        0.2,
        7,
        30
      );
      let bottom_cylinder2_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let bottom_cylinder2 = new THREE.Mesh(
        bottom_cylinder2_geometry,
        bottom_cylinder2_material
      );

      // 圆柱3
      let bottom_cylinder3_geometry = new THREE.CylinderGeometry(
        0.2,
        0.2,
        7,
        30
      );
      let bottom_cylinder3_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let bottom_cylinder3 = new THREE.Mesh(
        bottom_cylinder3_geometry,
        bottom_cylinder3_material
      );

      // 圆柱中间球
      var sphere1 = this.getBottomSphere();
      var sphere2 = this.getBottomSphere();
      var sphere3 = this.getBottomSphere();

      bottom_cylinder1.add(sphere1);
      bottom_cylinder2.add(sphere2);
      bottom_cylinder3.add(sphere3);

      bottom_cylinder1.position.set(2, 2, 1);
      bottom_cylinder1.rotation.set(
        -(Math.PI / 5.5),
        Math.PI / 10,
        Math.PI / 5
      );

      bottom_cylinder2.position.set(0, 2, -2.5);
      bottom_cylinder2.rotation.set(Math.PI / 4.5, Math.PI / 6, -Math.PI / 1);

      bottom_cylinder3.position.set(-2, 2, 1.5);
      bottom_cylinder3.rotation.set(
        -Math.PI / 15,
        Math.PI / 8,
        (-Math.PI / 10) * 2
      );

      object.add(bottom_cylinder1, bottom_cylinder2, bottom_cylinder3);
      return object;
    },
    getBottomSphere() {
      var sphere_geometry = new THREE.SphereGeometry(0.32, 32, 32);
      var sphere_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      var sphere = new THREE.Mesh(sphere_geometry, sphere_material);
      return sphere;
    },
    getTorus(a, b, c, d, y) {
      let torus_geometry = new THREE.TorusGeometry(a, b, c, d);
      let torus_material = new THREE.MeshPhongMaterial({
        color: this.MATERIAL_COLOR,
      });
      let torus = new THREE.Mesh(torus_geometry, torus_material);
      torus.radius = 8;
      torus.tube = 2;
      torus.radialSegments = 7;
      torus.tubularSegments = 30;
      torus.rotation.x = Math.PI / 2;
      torus.position.y = y;
      return torus;
    },
  },
};
</script>
