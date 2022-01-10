<script setup>
import { onMounted, ref } from 'vue'
import * as PIXI from 'pixi.js'
import Candle from '@/assets/icon/candle.png'
import ChristmasGift from '@/assets/icon/christmas-gift.png'
import Fruit from '@/assets/icon/fruit.png'
import Snowflake from '@/assets/icon/snowflake.png'
import Stockings from '@/assets/icon/stockings.png'
import SmallBell from '@/assets/icon/small-bell.png'
import Glove from '@/assets/icon/glove.png'
import Snowman from '@/assets/icon/snowman.png'
import Tree from '@/assets/icon/tree.png'
import Cap from '@/assets/icon/cap.png'
import Candies from '@/assets/icon/candies.png'
import Cup from '@/assets/icon/cup.png'
import DeerHead from '@/assets/icon/deer-head.png'
import Coffee from '@/assets/icon/coffee.png'
import PieDoll from '@/assets/icon/pie-doll.png'
import BallShaped from '@/assets/icon/ball-shaped.png'

// 容器宽度高度
let width = 0
let height = 0
const margin_top = 20 // 距顶距离
const margin_left = 20 // 距左距离
const interval = 5 // 模块的间隔
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
      ...e,
      x, // 定位
      y,
      line, // 第几行
      loca: [x + _unit_half, y, x + _unit, y + _unit_half, x + _unit_half, y + _unit, x, y + _unit_half],
      dom: null // 生成的精灵（暂时未生成）
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
  const metaList = [
    { name: 'Candle', url: Candle },
    { name: 'ChristmasGift', url: ChristmasGift },
    { name: 'Fruit', url: Fruit },
    { name: 'Snowflake', url: Snowflake },
    { name: 'Stockings', url: Stockings },
    { name: 'SmallBell', url: SmallBell },
    { name: 'Glove', url: Glove },
    { name: 'Snowman', url: Snowman },
    { name: 'Tree', url: Tree },
    { name: 'Cap', url: Cap },
    { name: 'Candies', url: Candies },
    { name: 'Cup', url: Cup },
    { name: 'DeerHead', url: DeerHead },
    { name: 'Coffee', url: Coffee },
    { name: 'BallShaped', url: BallShaped },
    { name: 'PieDoll', url: PieDoll }
  ]
  const menus = calculate_model(metaList, width, height)

  // 初始化容器
  const app = new PIXI.Application({ width: width, height: height, transparent: true })

  let doms = []
  app.loader.add(metaList).load((_, resources) => {
    menus.forEach(e => {
      // 背景
      const dom = new PIXI.Graphics()
      dom.beginFill(0x66ccff)
      dom.lineStyle(5, 0x011471, 0.5)
      dom.drawPolygon(e.loca)
      dom.endFill()
      // icon
      const icon = new PIXI.Sprite(resources[e.name].texture)
      icon.x = e.x + unit.value / 2
      icon.y = e.y + unit.value / 2
      icon.anchor.set(0.5, 0.5)
      icon.width = unit.value * 0.6
      icon.height = unit.value * 0.6
      dom.addChild(icon)
      // 增加操作事件
      dom.interactive = true
      dom.buttonMode = true
      // 鼠标移入
      dom.on('mouseover', () => overOne(e.name))
      // 鼠标移出
      dom.on('mouseout', () => outOne())
      // 添加至元素集合
      doms.push({
        ...e,
        dom
      })
      app.stage.addChild(dom)
    })
  })

  page.appendChild(app.view)

  // // 循环创建菜单
  // let doms = []
  // menus.forEach(e => {
  //   // 创建精灵并渲染
  //   const dom = new PIXI.Graphics()
  //   dom.beginFill(0x66ccff)
  //   dom.lineStyle(5, 0x011471, 0.5)
  //   dom.drawPolygon(e.loca)
  //   dom.endFill()
  //   // 增加操作事件
  //   dom.interactive = true
  //   dom.buttonMode = true
  //   // 鼠标移入
  //   dom.on('mouseover', () => overOne(e.id))
  //   // 鼠标移出
  //   dom.on('mouseout', () => outOne(e.id))
  //   // 鼠标点击
  //   dom.on('click', () => click(e))
  //   // 添加至元素集合
  //   doms.push({
  //     ...e,
  //     dom
  //   })
  //   // 精灵添加至画布
  //   app.stage.addChild(dom)
  // })
  // // 鼠标点击事件
  // const click = row => {
  //   console.log('点击了', row)
  // }
  // 一级元素鼠标移入&移出事件
  const overOne = name => {
    // 选中元素透明度调整为1，其他为0.5
    doms.forEach(e => {
      if (e.name === name) {
        e.dom.alpha = 1
      } else {
        e.dom.alpha = 0.5
      }
    })
  }
  const outOne = () => {
    // 所有元素透明度调整为1
    doms.forEach(e => {
      e.dom.alpha = 1
    })
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
  background-image: radial-gradient(farthest-corner at 60% 55%, #012ae4, #010a3e);
}
</style>
