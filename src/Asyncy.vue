<template>
  <div
    id="asyncy"
    ref="self"
    :class="{'not-found': $route.name === 'not-found'}">
    <div class="stars-container">
      <a-stars
        fixed
        animated />
    </div>
    <a-navbar :dark="$route.meta.darkNav" />
    <transition name="fade">
      <router-view />
    </transition>
  </div>
</template>

<script>
// import debounce from 'lodash.debounce'

export default {
  name: 'Asyncy',
  data: () => ({
    isHandlingScroll: false,
    isHandlingResize: false,
    isHandlingMousemove: false,
    client: {
      screen: {
        width: 0,
        height: 0,
        innerHeight: 0,
        scrollTop: 0,
        topHeight: 140
      },
      mouse: {
        x: 0,
        y: 0
      }
    },
    events: []
  }),
  provide () {
    return {
      client: this.client,
      addEvent: this.addEvent,
      removeEvent: this.removeEvent
    }
  },
  mounted: function () {
    window.addEventListener('scroll', this.handleScroll) // debounce(this.handleScroll, 10, { leading: true, maxWait: 10 }))
    window.addEventListener('resize', this.handleResize) // debounce(this.handleResize, 150, { leading: true, maxWait: 10 }))
    window.addEventListener('mousemove', this.handleMousemove) // debounce(this.handleMousemove, 10, { leading: true, maxWait: 10 }))
    this.handleResize()
    this.handleScroll()
  },
  beforeDestroy: function () {
    window.removeEventListener('scroll', this.handleScroll)
    window.removeEventListener('resize', this.handleResize)
    window.removeEventListener('mousemove', this.handleMousemove)
  },
  methods: {
    addEvent: function (type, call, preCall = false) {
      this.events.push({ type, call })
      if (preCall) {
        try {
          call()
        } catch (e) { console.error(e) }
      }
    },
    removeEvent: function (type, call) {
      let idx = this.events.findIndex(v => v.type === type && v.call === call)
      if (idx !== -1) {
        this.events.splice(idx, 1)
      }
    },
    process: function (type, e) {
      for (let event of this.events) {
        if (event.type.includes(type)) {
          try {
            event.call(e)
          } catch (ignored) {}
        }
      }
    },
    handleScroll: function (event) {
      if (!this.isHandlingScroll) {
        this.isHandlingScroll = true
        if (this.client.screen.scrollTop !== (document.body.scrollTop || document.documentElement.scrollTop)) {
          this.client.screen.scrollTop = document.body.scrollTop || document.documentElement.scrollTop
        }
        this.process('scroll', event)
        this.isHandlingScroll = false
      }
    },
    handleResize: function (event) {
      if (!this.isHandlingResize) {
        this.isHandlingResize = true
        if (this.client.screen.width !== this.$refs.self.clientWidth ||
          this.client.screen.height !== this.$refs.self.clientHeight) {
          this.client.screen.width = this.$refs.self.clientWidth
          this.client.screen.height = this.$refs.self.clientHeight
        }
        if (this.client.screen.innerHeight !== window.innerHeight) {
          this.client.screen.innerHeight = window.innerHeight
        }
        this.process('resize', event)
        this.isHandlingResize = false
      }
    },
    handleMousemove: function (event) {
      if (!this.isHandlingMousemove) {
        this.isHandlingMousemove = true
        if (this.client.mouse.x !== event.clientX ||
          this.client.mouse.y !== event.clientY) {
          this.client.mouse.x = event.clientX
          this.client.mouse.y = event.clientY
        }
        this.process('mousemove', event)
        this.isHandlingMousemove = false
      }
    }
  }
}
</script>

<style lang="scss">
.fade-enter-active,
.fade-leave-active {
  transition-duration: 0.3s;
  transition-property: opacity;
  transition-timing-function: ease-in;
}

.fade-enter,
.fade-leave-active {
  opacity: 0
}

.stars-container {
  overflow: hidden;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: 0;
  padding: 0;
  position: absolute;
}
</style>
