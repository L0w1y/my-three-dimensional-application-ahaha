<template>
  <div id="app">
    <div ref="sceneContainer"></div>
  </div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

export default {
  name: 'App',
  mounted() {
    this.initScene();
    this.loadModel();
  },
  methods: {

    // метод, кт получит рандомную точку в пространстве какого-то куба. 
    // Тупо чтобы расставить освещение - написал. Так то роли не хуйча не играт
    getRandomPointInBoundingBox(boundingBox) {
      const randomPoint = new THREE.Vector3();
      boundingBox.getCenter(randomPoint);
      const max = boundingBox.max;
      const min = boundingBox.min;

      randomPoint.x = THREE.MathUtils.randFloat(min.x, max.x);
      randomPoint.y = THREE.MathUtils.randFloat(min.y, max.y);
      randomPoint.z = THREE.MathUtils.randFloat(min.z, max.z);

      return randomPoint;
    },


    // Метод инициализации сцены, йоб
    initScene() {
      // Порносцена - её создание
      this.scene = new THREE.Scene();

      // Рассчёты для угла обзора (анального позора) камеры
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

      const ambientLight = new THREE.AmbientLight(0xffffff, 0.5); // Типо непрямое затянение
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8); // Типа солнце
      directionalLight.position.set(5, 5, 5);

      // Добавляет источники света на порносцену
      this.scene.add(ambientLight);
      this.scene.add(directionalLight);

      // Вебкам орбитальный
      this.controls = new OrbitControls(this.camera, this.$refs.sceneContainer);
      this.controls.enableDamping = true;
      this.controls.dampingFactor = 0.25;
      this.controls.screenSpacePanning = false;

      // Тут инициализация WebGL редера
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(window.innerWidth, window.innerHeight);

      // Тупо рендерер в контейнер добавляем. Можно и канвас йобнуть
      this.$refs.sceneContainer.appendChild(this.renderer.domElement);

      // Биндим эвенты на ресайз окна, шоб перерисовать заново окно рендера, йоббана
      window.addEventListener('resize', this.onWindowResize, false);

      // Вебкам установки - позиция ищначально пусть 0 по всем осям, йоб
      this.camera.position.set(0, 0, 0);
    },

    // Метод загруки модели откуда-то (хуууууй знааает)
    loadModel() {
      // Загрузка модели (твой путь или мой или или ли)
      const loader = new GLTFLoader();

      // Здесь твоя модель, йобб
      // Модели хранятся в папке проекта - public/models
      // Так уж и быть - оставил тебе спизженный глок со скетчфаба) Не благодари
      const model = 'models/glock.gltf';

      // Тут тупо сама загрузка модели на сцену
      loader.load(
        model, // твоя модель тут юзается, шоб загрузиться
        (gltf) => {
          this.model = gltf.scene;

          // Считали радиус той хуйни, сколько занимает обьект в пространстве. 
          // Короче шоб камера была отдалена
          const boundingBox = new THREE.Box3().setFromObject(this.model);
          const boundingSphere = boundingBox.getBoundingSphere(new THREE.Sphere()); // Именно тут радиус, йоб


          // Обозначить надо было, сколько будет источников освещения, 
          // шоб было светлее, йоб
          const ligthsNum = 30; 


          // Тупо в цикле идём от 0 до значения ligthsNum включительно
          for (let i = 0; i < ligthsNum; i++) {

            // Создаём точечный источник света, с радиусом в 100, интерсивностью в 1, и цветом белым
            const light = new THREE.PointLight(0xffffff, 1, 100);

            // Тупо выбираем рандомную точку в пространстве нашей модели
            const randomPoint = this.getRandomPointInBoundingBox(boundingBox);
            
            // Перемешаем источник света на позицию кт только что посчитали
            light.position.copy(randomPoint);

            // Добавляем на сцену, ибо нехуй!
            this.scene.add(light);
          }

          // Считаем расстояние камеры от обьекта
          // Рассчёт выглядит следующим образом - берётся куб, в который вписан обьект. 
          // Куб покрывает весь обьект. Считаем радиус куба этого, и умножаем на 1.5, шоб не пиздело
          this.camera.position.set(0, 0, boundingSphere.radius * 1.5);

          // Камера теперь смотрит в центр обьекта, йоббана
          boundingBox.getCenter(this.controls.target);
          this.controls.update();

          // Модель нас уже заебала валяться. Поэтому пихаем её на порносцену!
          this.scene.add(this.model);

          // Если там анимация какая у неё есть - запускаем, Ибо похуй
          this.animate();
        },
        undefined,
        (error) => console.error(error) // в случае ошибки открываешь F12 И ошибки будут в консоль в браузере выводиться
      );
    },

    // метод перересовывания окна в случае пиздеца
    onWindowResize() {
      // Если окно изменило свой ебаный размер - пересчитываем его и похуй. Забываем на этом
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },

    // метод анимирования моделей, в случае модельного пиздеца 
    // и обработка камеры и перересовка сцены

    animate() {
      // Анимацию зациклим?
      requestAnimationFrame(this.animate);

      // Обновляем йоббану камеру
      this.controls.update();

      // Перерисовываем сцену йоббану
      this.renderer.render(this.scene, this.camera);
    },
  },
};
</script>

<style> /* Стили по желанию. Не желай лучше! */
</style>
