

<template>
  <select v-model="selectRenderLibrary">
    <option v-for="item in selectRenderLibraryOptions" :value="item.value">{{ item.label }}</option>
  </select>
  <button @click="render()">渲染</button>
  <input type="number" v-model="blockNum" />
  <div>初始化时间:{{ initTime }}ms</div>
  <div>渲染时间:{{ renderTime }}ms</div>
  <div v-show="selectRenderLibrary === 'konva'" style="width: 400px;height: 400px;border: 1px solid red;" id="canvas">
  </div>
  <canvas v-show="selectRenderLibrary === 'fabric'" width="400" height="400" id="c" style="border: 1px solid #ccc;">
  </canvas>
</template>
<script setup>
import { computed, ref } from 'vue'
import { fabric } from 'fabric' 
import Konva from 'konva'
// 引入 performance 对象
const { performance } = window
const renderTime = ref(0)
const initTime = ref(0)
const blockNum = ref(0)
// const canvas = ref(null)
const selectRenderLibrary = ref('konva')
class FabricRenderer {
  // 实例内容
  fabricCanvas = null;
  constructor() {
    this.fabricCanvas = new fabric.Canvas("c")
  }
  clear() {
    this.fabricCanvas.clear()
  }
  add(...args) {
    this.fabricCanvas.add(...args)
  }
  rect({ top, left, width, height, fill }) {
    return [new fabric.Rect({ top, left, width, height, fill })]
  }
  draw() {
    // this.fabricCanvas.renderAll()
  }
}
class KonvaRenderer {
  konvaStage = null;
  konvaLayer = null;
  constructor() {
    const stage = new Konva.Stage({
      container: "canvas",
      width: 400,
      height: 400
    })
    const layer = new Konva.Layer();
    stage.add(layer);
    console.log(stage)
    this.konvaLayer = layer
   
    // 绑定图形点击事件
    stage.on('click tap', function (e) {
      console.log(e.target)
      // console.log(stage.find('Transformer'))
      if (e.target === stage) {
        stage.find('Transformer').forEach(item=>{
          item.destroy()
        })
        layer.draw();
        return;
      }
      if (!e.target.hasName('rect')) {
        return;
      }
      var tr = new Konva.Transformer();
      layer.add(tr);
      tr.attachTo(e.target);
      layer.draw();
    });
    this.konvaStage = stage

  }
  clear() {
    console.log(this.konvaStage)
    this.konvaLayer.destroyChildren()
  }
  add(...args) {
    console.log("add")
    this.konvaLayer.add(...args)
  }
  rect({ top, left, width, height, fill }) {
    return [new Konva.Rect({ x:top, y:left, width, height, fill,name:'rect',draggable:true})]
  }
  draw() {
    console.log("draw",this.konvaLayer, this.konvaStage)
    this.konvaLayer.draw()
  }
}

const selectRenderLibraryOptions = [
  {
    label: 'fabric',
    value: 'fabric',
    fn: {
      init: () => new fabric.Canvas("c"),// 这里传入的是canvas的id
      clear: () => canvas.value.clear(),
      add: (...args) => canvas.value.add(...args),
      rect: ({ top, left, width, height, fill }) => [new fabric.Rect({ top, left, width, height, fill })],
      draw: () => { }
    },
  },
  {
    label: 'konva',
    value: 'konva',
    fn: {
      init: () => {
        //  渲染stage
        const stage = new Konva.Stage({
          container: "canvas",
          width: 400,
          height: 400
        })
        //  渲染layer
        var layer = new Konva.Layer();
        stage.add(layer);
        // 绑定图形点击事件
        stage.on('click tap', function (e) {
          console.log(e.target)
          console.log(stage.find('Transformer'))
          if (e.target === stage) {
            console.log("")
            stage.find('Transformer')[0].destroy();
            layer.draw();
            return;
          }
          if (!e.target.hasName('rect')) {
            return;
          }
          var tr = new Konva.Transformer();
          layer.add(tr);
          tr.attachTo(e.target);
          layer.draw();
        });
        return layer;
      },
      clear: () => canvas.value.destroyChildren(),
      add: (...args) => {
        canvas.value.add(...args);
      },
      draw: () => {
        canvas.value.draw();
      },
      rect: ({ top, left, width, height, fill }) => {
        const shape = new Konva.Rect({
          x: left,
          y: top,
          width,
          height,
          fill,
          name: "rect",
          draggable: true
        })
        return [shape]
      }
    }
  }
]
const library = computed(() => {
  console.log(selectRenderLibrary.value)
  if (selectRenderLibrary.value === 'fabric') {
    return init(FabricRenderer);
  } else {
    return init(KonvaRenderer);
  }
})
function init(renderClass) {
  console.log(renderClass)
  const startTime = performance.now()
  const renderer=new renderClass()
  const endTime = performance.now()
  initTime.value = endTime - startTime
  return renderer
}

function render() {
  console.log(library.value)
  library.value.clear()
  const startTime = performance.now()

  for (let i = 0; i < blockNum.value; i++) {
    library.value.rect({
      top: Math.random() * 300,
      left: Math.random() * 300,
      width: 50,
      height: 50,
      fill: ` hsl(${Math.random() * 360}, 100%, 50%)` // 填充随机颜色
    }).forEach(item => {
      library.value.add(item)
    })

  }
  library.value.draw();
  // 渲染完成后计算渲染时间
  const endTime = performance.now()
  renderTime.value = endTime - startTime
}



</script>


<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
