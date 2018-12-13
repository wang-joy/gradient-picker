<template>
  <div class="gradient-picker-container">
    <fieldset>
      <legend>方向</legend>
      <el-radio v-model="direction" v-for="direct in arr" :label="direct.label" :key="direct.label">{{direct.desc}}</el-radio>
    </fieldset>
    <el-checkbox v-model="symmetry">对称</el-checkbox>
    <div>
      <div class="gradient-show-span" :style="{background: gradientSpanColor}"></div>
      <div class="gradient-cursor" @click='addCursor'>
        <ul ref="cursors" >
          <li @click="clickCursor(cursor)" v-for="cursor in cursors" :key="cursor.id" :style="{left: cursor.percent}">
            <span class="trangle"></span>
            <span class="rect">
              <span :style="{'background-color': cursor.bg}"></span>
            </span>
          </li>
        </ul>
      </div>
    </div>
    <div class="color-span" v-clickoutside='hideColorPickerContainer'>
      <span class="color-dispaly-span" :style="{backgroundColor: colorValue}"></span>
      <i class="el-icon-caret-bottom" @click="showColor=true"></i>
      <color-picker-container v-model="showColor" @change="change" :color-value="colorValue" class="color-picker-container" ref="cpc"></color-picker-container>
    </div>
    <div class="display-gradient" :style="{'background':gradient}"></div>
  </div>
</template>

<script>
import ColorPickerContainer from '../color-picker/ColorContainer'
import Clickoutside from 'element-ui/src/utils/clickoutside'
import Color from 'element-ui/packages/color-picker/src/color'
export default {
  data () {
    return {
      direction: '',
      arr: [
        {
          label: 'horizontal',
          desc: '水平'
        },
        {
          label: 'vertical',
          desc: '垂直'
        },
        {
          label: 'upward',
          desc: '斜上'
        },
        {
          label: 'downward',
          desc: '斜下'
        },
        {
          label: 'radiate',
          desc: '放射'
        }
      ],
      cursors: [
        {
          id: 0,
          percent: '0%',
          bg: '#000000'
        },
        {
          id: 100,
          percent: '100%',
          bg: '#ffffff'
        }
      ],
      showColor: false,
      activeCursor: null,
      symmetry: false,
      colorValue: '',
      cursor: null,
      count: 1
    }
  },
  components: {ColorPickerContainer},
  methods: {
    hideColorPickerContainer () {
      this.showColor = false
    },
    change (val) {
      var target = this.target
      target.style['background-color'] = val
      this.colorValue = val
      this.cursor.bg = val
    },
    clickCursor (cursor) {
      this.activeCursor = event.currentTarget
      this.cursor = cursor
      event.stopPropagation()
    },
    toPoint (percent) {
      var str = percent.replace('%', '')
      return str / 100
    },
    toPercent (point) {
      var percent = Number(point * 100).toFixed(2)
      percent += '%'
      return percent
    },
    update () {
      this.$nextTick(_ => {
        let {cpc} = this.$refs
        cpc && cpc.update()
      })
    },
    getHexColorValue (color) {
      var current = new Color({format: 'hex'})
      current.fromString(color)
      return current.value
    },
    addCursor () {
      var percent = this.getPercent(event)
      var id = this.count++
      var newCursor = {
        id: id,
        percent: percent,
        bg: '#000000'
      }
      this.cursors.splice(this.cursors.length - 1, 0, newCursor)
    },
    getPercent (evt) {
      var rect = evt.currentTarget.getBoundingClientRect()
      var width = rect.width
      var left = rect.left
      var x = evt.clientX
      return this.toPercent((x - left - 2.5) / width)
    }
  },
  directives: {Clickoutside},
  mounted () {
    let cursors = this.$refs.cursors
    this.activeCursor = cursors.firstChild
    this.cursor = this.cursors[0]
  },
  computed: {
    gradient () {
      // 获取方向
      let direction = this.direction
      let gradientStr = ''
      switch (direction) {
        case 'horizontal':
          gradientStr = 'linear-gradient(90deg' + this.traverseCursors + ')'
          break
        case 'vertical':
          gradientStr = 'linear-gradient(180deg' + this.traverseCursors + ')'
          break
        case 'upward':
          gradientStr = 'linear-gradient(45deg' + this.traverseCursors + ')'
          break
        case 'downward':
          gradientStr = 'linear-gradient(135deg' + this.traverseCursors + ')'
          break
        case 'radiate':
          let cursors = this.cursors.concat()
          gradientStr = 'radial-gradient('
          if (!this.symmetry) {
            cursors.forEach(cursor => {
              gradientStr += cursor.bg + ' ' + cursor.percent + ','
            })
          } else {
            cursors.forEach(cursor => {
              gradientStr += cursor.bg + ' ' + this.toPercent(this.toPoint(cursor.percent) / 2) + ','
            })
            cursors.reverse().forEach(cursor => {
              gradientStr += cursor.bg + ' ' + this.toPercent(1 - this.toPoint(cursor.percent) / 2) + ','
            })
          }
          gradientStr = gradientStr.substr(0, gradientStr.length - 1)
          gradientStr += ')'
          break
        default:
          gradientStr = 'linear-gradient(90deg' + this.traverseCursors + ')'
      }
      return gradientStr
    },
    traverseCursors () {
      let gradientStr = ''
      let cursors = this.cursors.concat()
      if (!this.symmetry) {
        cursors.forEach(cursor => {
          gradientStr += ',' + cursor.bg + ' ' + cursor.percent
        })
      } else {
        cursors.forEach(cursor => {
          gradientStr += ',' + cursor.bg + ' ' + this.toPercent(this.toPoint(cursor.percent) / 2)
        })
        cursors.reverse().forEach(cursor => {
          gradientStr += ',' + cursor.bg + ' ' + this.toPercent(1 - this.toPoint(cursor.percent) / 2)
        })
      }
      return gradientStr
    },
    gradientSpanColor () {
      let gradientStr = 'linear-gradient(90deg'
      this.cursors.forEach(cursor => { gradientStr += ',' + cursor.bg + ' ' + cursor.percent })
      return gradientStr
    },
    target () {
      if (this.activeCursor) {
        return this.activeCursor.getElementsByClassName('rect')[0].getElementsByTagName('span')[0]
      }
      return null
    }
  },
  watch: {
    target () {
      if (this.target) {
        this.colorValue = this.getHexColorValue(this.target.style['background-color'])
      }
    },
    colorValue () {
      this.update()
    },
    showColor () {
      this.update()
    }
  }
}
</script>

<style>
.gradient-picker-container{
  width: 400px;
  height: 300px;
}
.gradient-picker-container>fieldset{
  width: 400px;
  height: 40px;
}
.gradient-picker-container .gradient-show-span{
  width: 428px;
  height: 50px;
  border: 1px solid #ccc;
}
.gradient-picker-container .gradient-cursor{
  width: 428px;
  height: 30px;
  margin-left: -4px;
}
.gradient-picker-container .gradient-cursor ul{
  position: relative;
  list-style: none;
  padding: 0;
  margin: 0;
  font-size:0;
}
.gradient-picker-container .gradient-cursor ul>li{
  position: absolute;
  display: inline-block;
  padding: 0;
  margin: 0;
  vertical-align:bottom;
  cursor: pointer;
}
.gradient-picker-container .gradient-cursor ul>li .trangle{
  display: block;
  width: 0;
  height: 0;
  border-right: 5px solid transparent;
  border-left: 5px solid transparent;
  border-bottom: 5px solid rgb(0, 0, 0);
}
.gradient-picker-container .gradient-cursor ul>li .rect{
  display: block;
  width: 8px;
  height: 8px;
  border: 1px solid #000;
}
.gradient-picker-container .gradient-cursor ul>li .rect span{
  display: block;
  width: 6px;
  height: 6px;
  margin: 1px;
  background-color: #000;
}
.gradient-picker-container .color-span {
  width: 200px;
  height: 25px;
  border: 1px solid #ccc;
}
.gradient-picker-container .color-span .color-dispaly-span {
  display: inline-block;
  height: 21px;
  width: 170px;
  margin: 2px;
}
.gradient-picker-container .color-span i {
  width: 25px;
  height: 25px;
  line-height: 25px;
  text-align: center;
  float: right;
  cursor: pointer;
  background: #fff;

}
.gradient-picker-container .color-span i:hover {
  background-color: #ccc;
}
.gradient-picker-container .color-span .color-picker-container{
  position: absolute;
  width: 260px;
  background: #fff;
  border: 1px solid #ccc;
}
.gradient-picker-container .display-gradient{
  width: 200px;
  height: 200px;
  border: 1px solid #000;
}
</style>
