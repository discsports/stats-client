<template>
  <div class="home">
    <svg viewBox="0 0 330 120">
      <rect class="field" @mousemove="report" @click="vector" x="0" y="0" width="330" height="120"/>
      <rect class="endzone" x="0" y="0" width="60" height="120"/>
      <rect class="endzone" x="270" y="0" width="60" height="120"/>
      <circle class="brick" cx="120" cy="60" r="2"/>
      <circle class="brick" cx="210" cy="60" r="2"/>
      <line class="flightpath" v-for="th, i in point" :key="th.id" :x1="th.x0" :y1="th.y0" :x2="th.x1" :y2="th.y1" :stroke="colors[Math.min(point.length-i,colors.length-1)]"/>
    </svg>
    <radial-menu id="radialMenu" :open="radialBoolean" :angle-restriction="360" :radius="80" :size="40" :itemSize="70" @cancel="closeMenu">
      <radial-menu-item
        v-for="(player, index) in activePlayers"
        :key="index"
        @click="() => playerClick(player)"
      >
        <span>{{ player.name }}</span>
      </radial-menu-item>
    </radial-menu>
    <div>
      <span>X: {{ x }}, Y: {{ y }}</span>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue';
import RadialMenu from '@/components/RadialMenu.vue';
import RadialMenuItem from '@/components/RadialMenuItem.vue';
import chroma from 'chroma-js';

export default {
  name: 'home',
  components: {
    HelloWorld,
    RadialMenu,
    RadialMenuItem,
  },
  data() {
    return {
      x: null,
      y: null,
      point: [],
      roster: [
        { id: 0, name: "Steve", number: 52, active: 1 },
        { id: 1, name: "Kelsey", number: 17, active: 1 },
        { id: 2, name: "Gordy", number: 117, active: 1 },
        { id: 3, name: "Tico", number: 7, active: 1 },
        { id: 4, name: "Groot", number: 52, active: 1 },
        { id: 5, name: "Nate", number: 17, active: 1 },
        { id: 6, name: "Dakota", number: 117, active: 1 },
        { id: 7, name: "Cotton", number: 7, active: 0 },
      ],
      radialBoolean: false,
      state: {
        p0: {},
        x0: null,
        y0: null,
        p1: {},
        x1: null,
        y1: null,
      },
      // http://davidjohnstone.net/pages/cubehelix-gradient-picker#20,0.2,2.5,0.3,1,0.6,.7
      // colors: [
      //   "#f97d25",
      //   "#e58b29",
      //   "#d29832",
      //   "#c2a33e",
      //   "#b5ab4d",
      //   "#acb15e",
      //   "#a7b670",
      //   "#a5b883",
      //   "#a6b994",
      //   "#abb8a5",
      // ]
      colors: chroma.cubehelix()
        .start(200)
        .rotations(-0.3)
        .gamma(0.7)
        .hue([1,0.7])
        .lightness([0.3, 0.6])
        .scale()
        .correctLightness()
        .colors(8)
    };
  },
  computed: {
    activePlayers() {
      return this.roster.filter(p => p.active && p.id != this.state.p0.id);
    }
  },
  methods: {
    report(event) {
      const svg = event.target.parentElement;
      // console.log(svg);
      const pt = svg.createSVGPoint();
      // console.log(svg.getBoundingClientRect());
      pt.x = event.clientX;
      pt.y = event.clientY;
      const tPt = pt.matrixTransform(svg.getScreenCTM().inverse());
      this.x = Math.round(tPt.x);
      this.y = Math.round(tPt.y);
    },
    vector(event) {
      const svg = event.target.parentElement;
      const pt = svg.createSVGPoint();
      pt.x = event.clientX;
      pt.y = event.clientY;
      const radial = document.querySelector('#radialMenu');
      radial.style.display = 'initial';
      radial.style.left = `${event.pageX - radial.clientWidth/2}px`;
      radial.style.top = `${event.pageY - radial.clientHeight/2}px`;
      this.radialBoolean = true;
      const tPt = pt.matrixTransform(svg.getScreenCTM().inverse());
      const x = Math.round(tPt.x);
      const y = Math.round(tPt.y);
      if (this.state.x0) {
        this.state.x1 = x;
        this.state.y1 = y;
      } else {
        this.state.x0 = x;
        this.state.y0 = y;
      }
      this.$forceUpdate();
    },
    playerClick(player) {
      if (Object.entries(this.state.p0).length === 0) {
        this.state.p0 = player;
      } else {
        this.state.p1 = player;
        this.point.push(JSON.parse(JSON.stringify(this.state)));
        this.state.p0 = this.state.p1;
        this.state.x0 = this.state.x1;
        this.state.y0 = this.state.y1;
      }
      this.closeMenu();
    },
    closeMenu() {
      this.radialBoolean = false;
      const radial = document.querySelector('#radialMenu');
      setTimeout(() => radial.style.display = 'none', 150);
    },
  },
};
</script>

<style type="text/css">
  body {
    background: #ccc;
  }

  svg {
    /*position: absolute;*/
    /*top: 113px;*/
    /*left: 247px;*/
    width: 80%;
    /*height: 600px;*/
    /*background: #999;*/
  }
  
  .field {
    stroke-width: 2;
    fill: #a4c99c;
  }
  
  .endzone {
    pointer-events: none;
  }
  
  .brick {
    pointer-events: none;
    /*stroke: none;*/
    /*fill: #ccc;*/
  }
  
  .flightpath {
    pointer-events: none;
    stroke-linecap: round;
    stroke-linejoin: round;
    stroke-width: 1.5;
  }
  
  svg rect,circle {
    pointer-events: fill;
    stroke: #f2f2f2;
    fill: none;
    stroke-width: 1;
  }
  
  #radialMenu {
    display: none;
    position: absolute;
    left: 0px;
    top: 0px;
  }
  
</style>
