<template>
  <div class="background">
    <div id="renderer"></div>
  </div>
</template>

<script>
import * as THREE from 'three';

export default {
  name: "Background",
  components: {
  },
  methods: {
    setDark() {
      this.uniforms.dark.value = true;
    },
    setLight() {
      this.uniforms.dark.value = false;
    },
  },
  mounted() {
    var container;
    var camera, scene, renderer;
    var mesh;
    const initF = () => {
        container = document.getElementById('renderer');

        camera = new THREE.Camera();
        camera.position.z = 1;
        scene = new THREE.Scene();

        this.uniforms = {
            time: { type: "f", value: 1.0 },
            resolution: { type: "v2", value: new THREE.Vector2() },
            mousepos: { type: "v2", value: new THREE.Vector2() },
            dark: { type: "bool", value: false },
        };

        this.material = new THREE.ShaderMaterial({
            uniforms: this.uniforms,
            vertexShader: this.vertexShader,
            fragmentShader: this.fragmentShader,
        });

        mesh = new THREE.Mesh(new THREE.PlaneGeometry(2, 2), this.material);
        scene.add(mesh);

        renderer = new THREE.WebGLRenderer();
        renderer.setPixelRatio(window.devicePixelRatio ? window.devicePixelRatio : 1);
        container.appendChild(renderer.domElement);

        this.uniforms.resolution.value.x = window.innerWidth;
        this.uniforms.resolution.value.y = window.innerHeight;

        document.addEventListener('mousemove', (event) => {
          this.uniforms.mousepos.value.x = event.clientX;
          this.uniforms.mousepos.value.y = event.clientY;
        }, false);

        renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function animate() {
        requestAnimationFrame(animate);
        render();
    }

    const render = () => {
        var elapsedMilliseconds = Date.now() - startTime;
        var elapsedSeconds = elapsedMilliseconds / 1000.;
        this.uniforms.time.value = 60. * elapsedSeconds;
        renderer.setSize(window.innerWidth,  window.innerHeight);
        renderer.render(scene, camera);
    }

    initF();
    var startTime = Date.now();
    animate();
  },
  destroyed() {
  },
  data() {
    return {
      uniforms: {},
      material: undefined,
      vertexShader: `
      uniform float time;
      uniform vec2 mousepos;
      uniform bool dark;
      uniform vec2 resolution;
      void main()	{
          gl_Position = vec4( position, 1.0 );
      }
      `.trim(),
      fragmentShader: `
        #define POINT_CNT 30.
        #define BRIGHTNESS 0.0008
        #define LINE_THICKNESS 0.000001
        #define LINE_COLOR vec3(0.6)
        #define POINT_COLOR vec3(250./255.,124./255.,145./255.)
        #define COLOR_BRIGHTNESS 1.2
        #define PNT_DIST_TO_MOUSE 0.2

        uniform float time;
        uniform vec2 mousepos;
        uniform vec2 resolution;
        uniform bool dark;

        vec2 rand11(float seed) {
            float x = fract(sin(seed * 624.45) * 552.1);
            float y = fract(sin((seed+x) * 340.23) * 120.3);
            return vec2(x,y);
        }

        vec2 rand2(float seed) {
            float x = fract(sin(seed * 432.64) * 126.3);
            float y = fract(sin((seed+x) * 756.32) * 258.2);
            return vec2(x,y);
        }

        vec2 randomDirection(float seed) {
            float x = fract(sin(seed * 356.94) * 349.45);
            float y = fract(sin((seed+x) * 652.23) * 194.34);
            return vec2((x*2.)-1.,(y*2.)-1.);
        }

        vec2 remap11ToRes(vec2 point) {
            return vec2(point.x*((resolution.x/resolution.y)/2.), point.y/2.);
        }

        vec2 scalePos(vec2 pos, float scale) {
            return pos * scale;
        }

        vec2 randPos(float seed) {
            return (remap11ToRes(rand11(seed)*2. -1.));
        }

        vec2 pixelPosToUV(vec2 pos) {
            return (vec2(pos.x, resolution.y-pos.y))/resolution - 0.5;
        }

        vec2 getPosOfPoint(float i) {
            vec2 pointOrigin = scalePos((randPos(i+1.)), 1.3);
            vec2 dir = randomDirection(i+1.);
            vec2 randomDir = randomDirection(i + .1);
            dir = normalize(vec2(sin(time/100.) + randomDir.x, cos(time/100.) + randomDir.y));
            //if(abs(distance(pointOrigin, pixelPosToUV(mousepos))) < PNT_DIST_TO_MOUSE) {
            //    return pointOrigin - (pixelPosToUV(mousepos) - pointOrigin);
            //}
            return pointOrigin + pixelPosToUV(mousepos) / 30. + dir/10.;
        }

        void main()	{
            vec2 uv = (gl_FragCoord.xy-.5*resolution.xy)/resolution.y;
            // uv from -1 to 1 (0 center)
            
            vec3 col = vec3(1);
            if(dark) col = vec3(0);
            
            for(float i = 0.; i < POINT_CNT; i++) {
                vec2 point = getPosOfPoint(i);
                float distanceToPoint = length(uv-point);
                
                if(dark) {
                  col += vec3(BRIGHTNESS/distanceToPoint);
                } else {
                  col -= vec3(BRIGHTNESS/distanceToPoint * (1.-(POINT_COLOR*COLOR_BRIGHTNESS)));
                }
            }
            gl_FragColor = vec4(col,1.0);
        }`.trim(),
    };
  },
};
</script>

<style scoped lang="scss">
.background {
  position: fixed;
  top: 0;
  left: 0;
  z-index: -100;
}
</style>
