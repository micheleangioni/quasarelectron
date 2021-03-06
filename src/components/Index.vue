<template>
  <q-layout>
    <div slot="header" class="toolbar">
      <q-toolbar-title :padding="0">
        Quasar Framework v{{$q.version}}
      </q-toolbar-title>
    </div>

    <!--
      Replace following "div" with
      "<router-view class="layout-view">" component
      if using subRoutes
    -->
    <div class="layout-view">

      <div class="logo-container non-selectable no-pointer-events">
        <div class="logo" :style="position">
          <img src="~assets/quasar-logo.png">
          <p class="caption text-center">
            <span class="desktop-only">Move your mouse.</span>
            <span class="touch-only">Touch screen and move.</span>
          </p>
        </div>
      </div>

      <div class="button-container flex justify-center">
        <button class="primary" @click="sendSynchronousMessage()">Send Synchronous Message</button>
      </div>
    </div>
  </q-layout>
</template>

<script>
const {ipcRenderer} = require('electron')
const {dialog} = require('electron').remote

var moveForce = 30
var rotateForce = 40

import { Utils } from 'quasar'

export default {
  data () {
    return {
      moveX: 0,
      moveY: 0,
      rotateY: 0,
      rotateX: 0
    }
  },

  computed: {
    position () {
      let transform = `rotateX(${this.rotateX}deg) rotateY(${this.rotateY}deg)`
      return {
        top: this.moveY + 'px',
        left: this.moveX + 'px',
        '-webkit-transform': transform,
        '-ms-transform': transform,
        transform
      }
    }
  },

  methods: {
    move (event) {
      const {width, height} = Utils.dom.viewport()
      const {top, left} = Utils.event.position(event)
      const halfH = height / 2
      const halfW = width / 2

      this.moveX = (left - halfW) / halfW * -moveForce
      this.moveY = (top - halfH) / halfH * -moveForce
      this.rotateY = (left / width * rotateForce * 2) - rotateForce
      this.rotateX = -((top / height * rotateForce * 2) - rotateForce)
    },

    sendSynchronousMessage () {
      console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"
    }
  },

  mounted () {
    this.$nextTick(() => {
      document.addEventListener('mousemove', this.move)
      document.addEventListener('touchmove', this.move)
    })

    dialog.showMessageBox({
      title: 'Hello Quasar Electron',
      message: 'It works!',
      detail: 'This message is shown by Quasar running inside Electron.\n\nClick OK to proceed...',
      buttons: ['Ok']
    })
  },
  beforeDestroy () {
    document.removeEventListener('mousemove', this.move)
    document.removeEventListener('touchmove', this.move)
  }
}
</script>

<style lang="styl">
.logo-container
  width 192px
  height 268px
  perspective 800px
  position absolute
  top 50%
  left 50%
  transform translateX(-50%) translateY(-50%)
.logo
  position absolute
  transform-style preserve-3d
.button-container
  margin-top: 400px
</style>
