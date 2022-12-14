<template>
  <div class="demo" id="styles-demo">
    <div class="viewport">
      <screen ref="screen" v-if="visible">
        <markers :markers="markers">
        </markers>
        <edge v-for="edge in graph.edges" :data="edge" :nodes="graph.nodes" :key="edge.id">
        </edge>
        <node :data="node" ref="node" v-for="node in graph.nodes" :key="node.id">
        </node>
      </screen>
    </div>
    <div class="sidebar">
      <div>Theme:</div>
      <select v-model="themeSelect" style="width: 100%">
        <option value="blueprint">blueprint</option>
        <option value="reds">reds</option>
      </select>
      <!-- <textarea v-model="theme" style="height: calc(100% - 70px)"></textarea> -->
      <codemirror v-model="theme" :options="{
          mode: 'text/css',
          theme: 'default',
          lineWrapping: true,
          scrollbarStyle: null,
          styleActiveLine: true,
          line: true,
        }"
        style="font-size: 13.3333px; font-family: monospace; -webkit-text-size-adjust: 100%; height: 120%"
      ></codemirror>
    </div>
  </div>
</template>

<script>
import parse from 'css-parse'
import Screen from '../components/Screen'
import Node from '../components/Node'
import Edge from '../components/Edge'
import graph from '../graph'
import Markers from '../components/Markers.vue'
import { codemirror } from 'vue-codemirror'
import 'codemirror/mode/css/css.js'
import 'codemirror/lib/codemirror.css'

const themes = {
  reds: `
.screen {
  background-color: white
}

.node .content {
  background-color: pink;
  color: red;
  box-shadow: inset 0px 0px 0px 4px red;
}

.edge {
  stroke: red;
  stroke-linejoin: round;
  marker-end: url(#arrow-end-red);
}
`,
  blueprint: `
.screen {
  background-color: #1A53A9
}

.node .content {
  background-color: #559EF5;
  color: #fff;
  box-shadow: inset 0px 0px 0px 4px #fff;
}

.edge {
  stroke: #fff;
  stroke-linejoin: round;
  marker-start: url(#circle-white);
  marker-end: url(#circle-white);
}
  `
}

export default {
  components: {
    Screen,
    Node,
    Edge,
    Markers,
    codemirror
  },
  data() {
    return {
      themeSelect: 'blueprint',
      graph: new graph(),
      visible: true,
      markers: [
        {id:'arrow-end-red', type:'arrow-end', scale:0.5, style:'fill: red'} ,
        {id:'circle-white', type:'circle', scale:1, style:'fill: white'},
      ],
      theme: themes['blueprint'].trim()
    }
  },
  mounted () {
    this.graph.createNode('a')
    this.graph.createNode('b')
    this.graph.createNode('c')
    this.graph.createNode('d')
    this.graph.createNode('e')
    this.graph.createNode('f')

    const fromAnchor = 'rect'
    const toAnchor = 'rect'
    this.graph.createEdge('a', 'b', {fromAnchor, toAnchor})
    this.graph.createEdge('a', 'c', {fromAnchor, toAnchor})
    this.graph.createEdge('a', 'd', {fromAnchor, toAnchor})
    this.graph.createEdge('c', 'e', {fromAnchor, toAnchor})
    this.graph.createEdge('e', 'f', {fromAnchor, toAnchor})

    this.graph.graphNodes({ type: 'tree', spacing: 50 })
    this.applyTheme();
    window.t = this
  },
  methods: {
    selectTheme(theme) {
      this.theme = themes[theme].trim()
    },
    forceRender () {
      this.visible = false
      return this
        .$nextTick()
        .then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs.screen.zoomNodes(this.graph.nodes, { scale: 1 })
          })
        })
    },
    async applyTheme () {
      // parse theme rules
      let rules
      try {
        rules = parse(this.theme)
          .stylesheet.rules
          .filter(r => r.type === 'rule')
      } catch (e) {
        return;
      }
      // apply each rule to its selector elements
      await this.forceRender();
      rules.forEach(rule => {
        const sel = rule.selectors.length ? '#styles-demo ' + rule.selectors.join(', ') : ''
        const els = [...document.querySelectorAll(sel)]
        rule.declarations
          .filter(dec => dec.type === 'declaration')
          .forEach(dec => {
            els.filter(el => el).forEach(el => {
              const prop = dec.property.replace(/-([a-z])/g, function (g) { return g[1].toUpperCase(); });
              el.style[prop] = dec.value
            })
          })
      })
    }
  },
  watch: {
    theme: 'applyTheme',
    themeSelect: 'selectTheme'
  }
}
</script>

<style>
#styles-demo .CodeMirror {
  width: 100%;
  height: 425px;
  margin: 0;
  overflow: hidden;
  position: relative;
  background-color: #f1f1f1;
  border: 1px solid #f1f1f1;
}
</style>