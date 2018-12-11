<template>
  <div id="app">
    <button @click="dialogFormVisible=true">弹出</button>
    <gradient-picker-conatiner></gradient-picker-conatiner>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'
import GradientPickerConatiner from './gradient-picker/PickerContainer'
export default {
  name: 'App',
  components: {
    HelloWorld,
    GradientPickerConatiner
  },
  data () {
    return {
      dialogFormVisible: false,
      dialogWidth: '400px'
    }
  }
}
</script>

<style>
</style>
