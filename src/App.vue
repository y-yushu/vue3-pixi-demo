<script setup>
import { onMounted, ref } from 'vue'
import * as PIXI from 'pixi.js'

// 容器宽度高度
let width = 0
let height = 0
const margin_top = 50 // 距顶距离
const margin_left = 50 // 距左距离
const interval = 2 // 模块的间隔

/**
 * 计算单位
 * @param {Number} width 宽度
 * @param {Number} height 高度
 */
const calculate_unit = (width, height) => {
  if (width > height) {
    return Math.floor(height / 5)
  } else {
    return Math.floor(width / 5)
  }
}
const calculate_num = unit => {}
/**
 * 计算图形模型
 * @param {Array} menu 菜单列表
 * @param {Number} width 宽度
 * @param {Number} height 高度
 * @returns 渲染图形及定位
 */
const calculate_model = (menu = [], width = 0, height = 0) => {
  // 单位
  const unit = calculate_unit(width, height)
  const unit_half = unit / 2
  // 长度
  if (!menu.length) return []
  // 判断每行数量
  const line_num = parseInt(width / (unit + interval))
  // 每个开始计算定位
  let step = 0
  let line = 0
  const list = menu.map(e => {
    // 单双行个数不同
    if (line % 2) {
      if (step >= line_num - 1) {
        line++
        step = 0
      }
    } else {
      if (step >= line_num) {
        line++
        step = 0
      }
    }
    let x = margin_left + step * (unit + interval)
    const y = margin_top + line * (unit_half + interval)
    // 单双行交错
    if (line % 2) x += unit_half + interval / 2
    step++
    return {
      id: e,
      x,
      y,
      line,
      loca: [x + unit_half, y, x + unit, y + unit_half, x + unit_half, y + unit, x, y + unit_half]
    }
  })
  return list
}

// 组件加载完成
onMounted(() => {
  console.log('开始渲染')
  // 元素获取
  const page = document.getElementById('menu')
  width = page.offsetWidth
  height = page.offsetHeight - 1
  const menus = calculate_model([1, 2, 3, 4, 5, 6, 7, 8, 9, 18, 11, 12, 13, 14, 15, 16, 17], width, height)

  // 初始化容器
  const app = new PIXI.Application({ width: width, height: height, transparent: true })
  page.appendChild(app.view)

  // 循环创建菜单
  let doms = []
  menus.forEach(e => {
    // 创建精灵并渲染
    const dom = new PIXI.Graphics()
    dom.beginFill(0x66ccff)
    dom.lineStyle(5, 0x011471, 0.5)
    dom.drawPolygon(e.loca)
    dom.endFill()
    // 增加操作事件
    dom.interactive = true
    dom.buttonMode = true
    // 鼠标移入
    dom.on('mouseover', () => mouseover(e.id))
    // 鼠标移出
    dom.on('mouseout', () => mouseout())
    // 添加至元素集合
    doms.push({ id: e.id, dom })
    // 精灵添加至画布
    app.stage.addChild(dom)
  })
  // 鼠标移入事件
  const mouseover = id => {
    doms.forEach(e => {
      if (e.id === id) e.dom.alpha = 1
      else e.dom.alpha = 0.5
    })
  }
  const mouseout = () => {
    doms.forEach(e => (e.dom.alpha = 1))
  }
})
</script>

<template>
  <div id="menu" />
</template>

<style lang="scss">
html,
body,
#app {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}
#menu {
  width: 100%;
  height: 100%;
  margin: 0;
  // background-color: gold;
  background-image: radial-gradient(farthest-corner at 60% 55%, #012ae4, #010a3e);
}
</style>
