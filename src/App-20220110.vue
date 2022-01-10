<script setup>
import { onMounted, ref } from 'vue'
import * as PIXI from 'pixi.js'

// 容器宽度高度
let width = 0
let height = 0
const margin_top = 50 // 距顶距离
const margin_left = 50 // 距左距离
const interval = 0 // 模块的间隔
const unit = ref(0)

/**
 * 计算单位
 * @param {Number} width 宽度
 * @param {Number} height 高度
 */
const calculate_unit = (width, height) => {
  if (width > height) {
    unit.value = Math.floor(height / 4)
  } else {
    unit.value = Math.floor(width / 4)
  }
}
/**
 * 计算图形模型
 * @param {Array} menu 菜单列表
 * @param {Number} width 宽度
 * @param {Number} height 高度
 * @returns 渲染图形及定位
 */
const calculate_model = (menu = [], width = 0, height = 0) => {
  // 单位
  const _unit = unit.value
  const _unit_half = unit.value / 2
  // 长度
  if (!menu.length) return []
  // 判断每行数量
  const _line_num = parseInt(width / (_unit + interval))
  // 每个开始计算定位
  let step = 0
  let line = 0
  const list = menu.map(e => {
    // 单双行个数不同
    if (line % 2) {
      if (step >= _line_num - 1) {
        line++
        step = 0
      }
    } else {
      if (step >= _line_num) {
        line++
        step = 0
      }
    }
    let x = margin_left + step * (_unit + interval)
    const y = margin_top + line * (_unit_half + interval)
    // 单双行交错
    if (line % 2) x += _unit_half + interval / 2
    step++
    return {
      id: e, // id
      x, // 定位
      y,
      line, // 第几行
      loca: [x + _unit_half, y, x + _unit, y + _unit_half, x + _unit_half, y + _unit, x, y + _unit_half]
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
  // 计算单位
  calculate_unit(width, height)
  const menus = calculate_model([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20], width, height)

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
    dom.on('mouseover', () => overOne(e.id))
    // 鼠标移出
    dom.on('mouseout', () => outOne(e.id))
    // 鼠标点击
    dom.on('click', () => click(e))
    // 添加至元素集合
    doms.push({
      ...e,
      dom,
      isFocus: false
    })
    // 精灵添加至画布
    app.stage.addChild(dom)
  })
  // 鼠标点击事件
  const click = row => {
    console.log('点击了', row)
  }
  // 一级元素鼠标移入&移出事件
  const overOne = id => {
    const _unit = unit.value
    const _unit_half = unit.value / 2
    // 选中元素透明度调整为1，其他为0.5
    doms.forEach(e => {
      if (e.id === id) {
        e.dom.alpha = 1
        const { x, y } = e
        const chil = new PIXI.Graphics()
        chil.beginFill(0xdab9c6)
        chil.lineStyle(5, 0x011471, 0.5)
        const dian1 = [x + _unit, y + _unit_half]
        const dian2 = [x + _unit, y + _unit]
        const dian3 = [x + _unit_half, y + _unit]
        chil.drawPolygon([...dian1, ...dian2, ...dian3])
        e.chil = chil
        // 增加操作事件
        chil.interactive = true
        chil.buttonMode = true
        // 鼠标移入
        chil.on('mouseover', () => overTwo(e.id))
        // 鼠标移出
        chil.on('mouseout', () => outTwo(e.id))
        app.stage.addChild(chil)
      } else {
        e.dom.alpha = 0.5
      }
    })
  }
  const outOne = id => {
    // 如果子级获取焦点，则不进行改变
    const _self = doms.find(e => e.id === id)
    if (_self.isFocus) return
    // 所有元素透明度调整为1
    doms.forEach(e => {
      e.dom.alpha = 1
      if (e.id === id) {
        app.stage.removeChild(e.chil)
      }
    })
  }
  // 二级元素移入&移出事件
  const overTwo = id => {
    const _self = doms.find(e => e.id === id)
    _self.isFocus = true
    console.log('触发Aaa')
  }
  const outTwo = id => {
    const _self = doms.find(e => e.id === id)
    _self.isFocus = false
    outOne(id)
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
