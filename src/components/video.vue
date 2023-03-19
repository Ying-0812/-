<template>
  <div ref="containerRef" class="videoContainer" :style="{width: width + 'px', height: height + 'px'}" @mouseenter="videoShow" @mouseleave="videoHide">
    <video ref="videoRef" :poster="poster" :autoplay="autoplay" :muted="muted" :loop="loop" style="width: 100%;height: 100%;">
      <source :src="src" type="video/mp4">
    </video>
    <div class="controls" v-show="controlRef">
      <div class="left">
        <span class="iconfont icon-shuaxin" class="iconSize" @click="changeReload"></span>
		<div class="timeContainer">
			<div>{{min + ':'}}</div>
			<div>{{seconds}}<span style="margin: 0px 5px">/</span></div>
        	<div>{{allTime}}</div>
		</div>
      </div>
      <div class="center">
        <span class="iconfont icon-shangyiji" class="iconSize" @click="changeRate('back')"></span>
        <span class="iconfont icon-kaishi" class="iconSize begin" v-if="begin" @click="changeStatus"></span>
        <span class="iconfont icon-zanting" class="iconSize begin" v-else @click="changeStatus"></span>
        <span class="iconfont icon-xiayiji" class="iconSize" @click="changeRate('move')"></span>
      </div>
      <div class="right">
		<span style="font-size: 20px" class="iconfont icon-yinliang" class="iconSize sound" v-if="sound" @click="changeSound"></span>
        <span style="font-size: 20px" class="iconfont icon-jingyin" class="iconSize sound" v-else @click="changeSound"></span>
		<span class="add" style="margin-left: 5px" @click="soundControl">-</span>
		<span class="add" style="font-size: 14px;margin: 0px 5px;width: 17px;text-align: center;">{{parseInt(soundControls * 10)}}</span>
		<span class="add" @click="soundControl('add')">+</span>
        <select class="select" v-model="speed" @change="changeSpeed">
          <option :value="itm" :name="itm" v-for="itm in speedList" :key="itm" class="option">{{itm}}</option>
        </select>
        <span class="iconfont icon-quanping" class="iconSize screen" @click="changeScreen"></span>
      </div>
      <div class="progress" ref="progressRef" @click="changeProgress">
        <div class="over" ref="overRef"></div>
        <div class="radius" ref="radiusRef"></div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue'
export default {
  name: 'HelloWorld',
  props: {
    src: {
      type: String,
      required: true
    },
    poster: {
      type: String,
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '100%'
    },
    autoplay: {
      type: String,
      default: null
    },
    muted: {
      type: String,
      default: null
    },
    loop: {
      type: Boolean,
      default: false
    },
    skip: {
      type: Number,
      default: 5
    },
    speedList: {
      type: Array,
      default: [0.5, 1, 1.5, 2.0]
    }
  },
  setup(props) {
    const speed = ref(1)
	const min = ref('00')
	const seconds = ref('00')
	const allTime = ref('')
    const begin = ref(true)
    const sound = ref(true)
    const videoRef = ref(null)
    const fullScreen = ref(true)
    const containerRef = ref(null)
    const progressRef = ref(null)
    const overRef = ref(null)
    const radiusRef = ref(null)
	const timerRef = ref(null)
	const controlRef = ref(false)
	const soundControls = ref()

    watchEffect(() => {
		if(videoRef.value) {
			soundControls.value = videoRef.value.volume
			videoRef.value.onloadedmetadata = function () {
				allTime.value = getAllTime()
			}
		}
       if(props.autoplay === 'autoplay' && videoRef.value) {
         startPlay(speed.value)
       }
    })

	function getAllTime() {
		let time = `${parseInt(videoRef.value.duration / 60 % 60).toString().padStart(2, '0')}:${parseInt(videoRef.value.duration % 60).toString().padStart(2, '0')}`
		return time
	}

	function soundControl(type) {
		if(type === 'add') {
			if(videoRef.value.volume >= 1) {
				return
			}
			videoRef.value.volume += 0.1
			soundControls.value = videoRef.value.volume
		} else {
			console.log(videoRef.value.volume)
			if(videoRef.value.volume <= 0.11) {
				return
			}
			videoRef.value.volume -= 0.1
			soundControls.value = videoRef.value.volume
		}
	}

    // 开始播放
    function startPlay(time) {
       let duration = videoRef.value.duration
		clearInterval(timerRef.value)
       timerRef.value = setInterval(() => {
         let currentTime = videoRef.value.currentTime
         if(currentTime >= duration) {
           clearInterval(timerRef.value)
		   if(props.loop) {
			startPlay(speed.value)
			overRef.value.style.width = '0px'
      		radiusRef.value.style.left = '0px'
		   }
         } else {
			min.value = parseInt(currentTime / 60 % 60).toString().padStart(2, '0')
			seconds.value = parseInt(currentTime % 60).toString().padStart(2, '0')
			overRef.value.style.width = (containerRef.value.clientWidth * (currentTime / duration)) + 'px'
      		radiusRef.value.style.left = ((containerRef.value.clientWidth * (currentTime / duration)) - (radiusRef.value.offsetWidth / 2)) + 'px'
         }
       }, (1000 / time))
    }

    // 播放暂停
    function changeStatus() {
      begin.value = !begin.value
      let video = videoRef.value
      if(!begin.value) {
        video.play()
        startPlay(speed.value)
      } else {
		clearInterval(timerRef.value)
        video.pause()
      }
    }

    // 重新播放
    function changeReload() {
      let video = videoRef.value
      video.load()
      begin.value = true
      overRef.value.style.width = '0px'
      radiusRef.value.style.left = '0px'
    }

    // 设置播放速度
    function changeSpeed(e) {
      let video = videoRef.value
      video.playbackRate = e.target.value
	  speed.value = e.target.value
	  startPlay(e.target.value)
    }

    // 静音
    function changeSound() {
      sound.value = !sound.value
      let video = videoRef.value
      video.muted = !sound.value
    }

    // 前进或者回退N秒
    function changeRate(mode) {
      let video = videoRef.value
      let currentTime = video.currentTime
      if(mode === 'move') {
        video.currentTime = currentTime += props.skip
        // if(currentTime >= duration) {
        //   if(props.loop) {
        //     return
        //   } else {
        //     video.pause()
        //     begin.value = true
        //   }
        // }
      } else {
        video.currentTime = currentTime -= props.skip
        if(currentTime <= 0) {
          video.pause()
          begin.value = true
        }
      }
    }

    //进入/退出全屏
    function changeScreen() {
        let element = containerRef.value
        if (!fullScreen.value) {
			overRef.value.style.width = (videoRef.value.currentTime / videoRef.value.duration) * progressRef.value.clientWidth + 'px'
          if (document.exitFullscreen) {
            document.exitFullscreen()
          } else if (document.webkitCancelFullScreen) {
            document.webkitCancelFullScreen()
          } else if (document.mozCancelFullScreen) {
            document.mozCancelFullScreen()
          } else if (document.msExitFullscreen) {
            document.msExitFullscreen()
          }
        } else {
			overRef.value.style.width = (videoRef.value.currentTime / videoRef.value.duration) * progressRef.value.clientWidth + 'px'
          if (element.requestFullscreen) {
            element.requestFullscreen()
          } else if (element.webkitRequestFullScreen) {
            element.webkitRequestFullScreen()
          } else if (element.mozRequestFullScreen) {
            element.mozRequestFullScreen()
          } else if (element.msRequestFullscreen) {
            // IE11
            element.msRequestFullscreen()
          }
        }
        fullScreen.value = !fullScreen.value
    }

    // 点击改变进度条
    function changeProgress(e) {
      let process = progressRef.value
      let radius = radiusRef.value
      let leftSpace = e.clientX - containerRef.value.getBoundingClientRect().left
      let scale = leftSpace / process.clientWidth
      let duration = videoRef.value.duration
      videoRef.value.currentTime = scale * duration
      overRef.value.style.width = leftSpace + 'px'
      radius.style.left = (leftSpace - (radius.offsetWidth / 2)) + 'px'
    }

	// 鼠标移入显示控制条
	function videoShow() {
		controlRef.value = true
	}

	// 鼠标移出隐藏控制条
	function videoHide() {
		controlRef.value = false
	}

    return {
      speed,
	  min,
	  seconds,
	  allTime,
      begin,
      sound,
	  soundControls,
      videoRef,
      containerRef,
      progressRef,
      radiusRef,
      overRef,
	  controlRef,
      fullScreen,
      changeStatus,
      changeReload,
      changeSpeed,
      changeSound,
      changeRate,
      changeScreen,
      changeProgress,
      startPlay,
	  videoShow,
	  videoHide,
	  getAllTime,
	  soundControl
    }
    
  }
}
</script>

<style>
@import url('../css/iconfont.css');
.videoContainer{
  position: relative;
  z-index: 1;
  margin: 0 auto;
  background: #000;
  user-select: none;
}
.controls{
  position: absolute;
  left: 0;
  bottom: 0;
  z-index: 999;
  width: 100%;
  height: 46px;
  background-color: rgba(0, 0, 0, 0);
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.left{
  margin-left: 10px;
  display: flex;
  align-items: center;
}
.timeContainer {
	margin-left: 10px;
	display: flex;
	color: #fff;
	font-size: 14px;
}
.right{
  margin-right: 10px;
  display: flex;
  align-items: center;
}
.progress{
  width: 100%;
  height: 8px;
  background-color: rgb(194, 194, 194);
  position: absolute;
  top: -8px;
  left: 0;
  cursor: pointer;
}
.radius{
  position: absolute;
  top: 50%;
  left: 0;
  transform: translate(0, -50%);
  width: 15px;
  height: 15px;
  background: #fff;
  border-radius: 50%;
}
.over{
  width: 0;
  height: 100%;
  background: linear-gradient(45deg, rgb(255, 0, 0), rgb(255, 165, 0), rgb(255, 255, 0), rgb(0, 255, 0), rgb(0, 127, 255), rgb(0, 0, 255), rgb(139, 0, 255));
}
.iconSize{
  color: #fff;
  display: inline-block;
  cursor: pointer;
  font-size: 22px;
}
.sound{
  margin-left: 10px;
}
.begin{
  margin: 0px 10px;
}
.select{
  width: 40px;
  height: 22px;
  margin: 0px 10px;
  border: none;
  outline: none;
  text-align: center;
  background-color: rgba(0, 0, 0, .6);
  color: #fff;
}
.add{
	color: #fff;
	font-size: 26px;
	cursor: pointer;
}
</style>
