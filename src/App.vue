<template>
  <div
    id="app"
    :class="appClasses"
  >
    <div class="maincard card p-5 is-flex is-flex-direction-column" ref="maincard">
      <h1 class="title mb-3 is-flex flex-direction-row">dotjulia</h1>
      <b-carousel
        class="custom-carousel"
        :autoplay="false"
        has-drag
        :arrow="false"
        repeat
        :indicator="false"
        v-model="page"
      >
        <b-carousel-item>
          <div class="firstpage">
            <p class="subtitle mb-2">
              find me on
              <b-tooltip position="is-bottom">
                <a href="https://github.com/dotjulia">github</a>
                <template v-slot:content>
                  <div class="tooltip-cls">
                    <span>show </span>
                    <b-icon
                      type="is-danger"
                      icon="heart"
                      size="is-small"
                    ></b-icon>
                    <span> with a </span>
                    <b-icon
                      type="is-warning"
                      size="is-small"
                      icon="star"
                    ></b-icon>
                  </div>
                </template>
              </b-tooltip>
            </p>
            <p class="subtitle">
              check out
              <b-tooltip position="is-top" label="soon">
                <a href="https://kalimba.chat">kalimba</a>
              </b-tooltip>
            </p>
          </div>
        </b-carousel-item>
        <b-carousel-item>
          <div class="secondpage is-flex is-flex-direction-column">
            <div class="is-flex is-flex-direction-row is-justify-content-space-between">
              <p class="subtitle mb-1">dark? </p>
              <b-switch v-model="dark"></b-switch>
            </div>
            <p class="subtitle mb-1">check out the <a href="https://www.shadertoy.com/view/NsfSR2">background</a></p>
            <p class="subtitle"> or a <a href="/dnd-map-viewer">map viewer for dnd</a></p>
          </div>
        </b-carousel-item>
      </b-carousel>
      <div class="is-flex chrome-fix">
        <b-button
          class="next-button"
          @click="page++"
          rounded
          icon-right="arrow-right"
          type="is-white is-small"
          :outlined="false"
        />
      </div>
    </div>
    <background ref="bgobj"/>
  </div>
</template>

<script>
import Background from './components/Background.vue';
export default {
  name: "App",
  components: {Background},
  data() {
    return {
      page: 0,
      dark: false,
      appClasses: ['is-flex', 'is-justify-content-center', 'is-align-items-center'],
      pageWidths: [
        250, 300,
      ],
    };
  },
  methods: {
    setPageWidth() {
      if(this.page >= this.pageWidths.length) return;
      this.$refs.maincard.style.width = this.pageWidths[this.page] + 'px';
    },
  },
  watch: {
    page() {
      this.setPageWidth();
    },
    dark() {
      if(this.dark) {
        if(!this.appClasses.includes('dark')) {
          this.appClasses.push('dark');
          this.$refs.bgobj.setDark();
        }
      } else {
        if(this.appClasses.includes('dark')) {
          this.appClasses = this.appClasses.filter(x => x != 'dark');
          this.$refs.bgobj.setLight();
        }
      }
    }
  },
  mounted() {
    this.setPageWidth();
  },
};
</script>

<style lang="scss">
@import '@/styles/dark.scss';
body {
  overflow: hidden;
}

.dark {
  .maincard {
    background-color: $background-darker;
    .title {
      color: $title-dark;
    }
    .subtitle {
      color: $text-dark;
    }
    a {
      color: $link-dark;
    }
    .next-button {
      background-color: $background-darker;
      color: $text-dark !important;
    }
  }
}

#app {
  width: 100vw;
  height: 100vh;
}
.tooltip-cls {
  z-index: 100;
}
.maincard {
  //width: 250px;
  transition: width 0.5s;
}
.custom-carousel {
  min-height: 80px !important;
}
.firstpage {
  //padding-right: 5vw;
  overflow: visible;
  //width: 200px;
}
.secondpage {
  //padding-right: 5vw;
  //width: 300px;
}
.chrome-fix {
  justify-content: flex-end !important;
}
.next-button {
  color: $primary !important;
}
</style>
