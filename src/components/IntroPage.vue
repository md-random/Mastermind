<template>
  <div class="intro-container">
    <div class="video-section">
      <div class="video-controls">
        <button class="icon-btn" @click="toggleMute">
          {{ isMuted ? '🔇' : '🔊' }}
        </button>
        <input 
          v-if="!isMuted"
          type="range" 
          min="0" 
          max="1" 
          step="0.05" 
          v-model.number="volume" 
          @input="onVolumeChange"
          class="volume-slider"
        />
      </div>
      <video 
        ref="bgVideo"
        :muted="isMuted"
        playsinline
        autoplay
        @ended="onVideoEnded"
        @timeupdate="onVideoTimeUpdate"
        class="bg-video"
      >
        <source src="/Mastermind2.mp4" type="video/mp4" />
        Your browser does not support HTML video.
      </video>
      <audio ref="bgAudio" :muted="isMuted" loop class="bg-audio">
        <source src="/Crimson_Tides_of_Kyoto.mp3" type="audio/mpeg" />
      </audio>
    </div>
    <div class="button-section">
      <div class="buttons-wrapper">
        <button class="menu-btn secondary" @click="$emit('showRules')">Gameplay</button>
        <button class="menu-btn primary" @click="$emit('playGame')">Play Game</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'

export default defineComponent({
  name: 'IntroPage',
  emits: ['showRules', 'playGame'],
  setup() {
    const bgVideo = ref<HTMLVideoElement | null>(null)
    const bgAudio = ref<HTMLAudioElement | null>(null)
    const isMuted = ref(true)
    const volume = ref(0.5)

    let isCrossfading = false
    let isVideoEnded = false

    onMounted(() => {
      if (bgVideo.value) bgVideo.value.volume = volume.value
      if (bgAudio.value) bgAudio.value.volume = 0
    })

    const startCrossfade = () => {
      if (!bgAudio.value || !bgVideo.value) return
      isCrossfading = true
      
      const targetAudioVol = volume.value * 0.37
      const startVideoVol = volume.value
      
      bgAudio.value.volume = 0
      bgAudio.value.play().catch(e => console.warn('Autoplay blocked until user interaction', e))
      
      let progress = 0
      const steps = 30
      const crossfadeDuration = 3
      const stepTime = (crossfadeDuration * 1000) / steps
      
      const fadeInterval = setInterval(() => {
        progress++
        const ratio = progress / steps
        
        if (!isMuted.value) {
          if (bgVideo.value && !isVideoEnded) {
            bgVideo.value.volume = Math.max(0, startVideoVol * (1 - ratio))
          }
          if (bgAudio.value) {
            bgAudio.value.volume = Math.min(targetAudioVol, targetAudioVol * ratio)
          }
        }
        
        if (progress >= steps) {
          clearInterval(fadeInterval)
        }
      }, stepTime)
    }

    const onVideoTimeUpdate = () => {
      if (!bgVideo.value || isCrossfading || isNaN(bgVideo.value.duration)) return
      const timeLeft = bgVideo.value.duration - bgVideo.value.currentTime
      if (timeLeft <= 3.0) {
        startCrossfade()
      }
    }

    const toggleMute = () => {
      isMuted.value = !isMuted.value
      if (!isMuted.value) {
        if (bgAudio.value && !bgAudio.value.paused) {
          bgAudio.value.volume = volume.value * 0.37
        }
        if (bgVideo.value && !isCrossfading && !isVideoEnded) {
          bgVideo.value.volume = volume.value
        }
      }
    }

    const onVolumeChange = () => {
      if (bgVideo.value && !isCrossfading && !isVideoEnded) {
        bgVideo.value.volume = volume.value
      }
      if (bgAudio.value && !bgAudio.value.paused) {
        bgAudio.value.volume = volume.value * 0.37
      }
      
      if (volume.value === 0) {
        isMuted.value = true
      } else if (isMuted.value) {
        isMuted.value = false
      }
    }

    const onVideoEnded = () => {
      isVideoEnded = true
      if (bgVideo.value) bgVideo.value.volume = 0
      if (!isCrossfading) {
        startCrossfade()
      }
    }

    return { bgVideo, bgAudio, onVideoEnded, onVideoTimeUpdate, isMuted, volume, toggleMute, onVolumeChange }
  }
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400..900;1,400..900&display=swap');

.intro-container {
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
  background-image: linear-gradient(rgba(13, 22, 32, 0.65), rgba(13, 22, 32, 0.85)), url('../assets/CherryBlossomMastermind.png');
  background-size: cover;
  background-position: center;
  background-color: #0d1620;
  font-family: 'Playfair Display', serif;
  overflow: hidden;
}

.video-section {
  position: absolute;
  top: 0; left: 0; width: 100%; height: 100%;
  z-index: 1;
}

.video-section::after {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: linear-gradient(to bottom, transparent 60%, rgba(13, 22, 32, 0.85) 100%);
  pointer-events: none;
  z-index: 10;
}

.video-controls {
  position: absolute;
  top: 20px;
  right: 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(0, 0, 0, 0.5);
  padding: 6px 14px;
  border-radius: 30px;
  z-index: 30;
  backdrop-filter: blur(4px);
}

.icon-btn {
  background: none;
  border: none;
  font-size: 1.8rem;
  cursor: pointer;
  color: white;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s;
}

.icon-btn:hover {
  transform: scale(1.1);
}

.volume-slider {
  cursor: pointer;
  width: 100px;
  accent-color: #3498db;
}

.bg-video {
  width: 100%;
  height: 100%;
  object-fit: contain;
  display: block;
}

.button-section {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  display: flex;
  justify-content: center;
  padding-bottom: 8vh;
  z-index: 20; 
}

.buttons-wrapper {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 2rem;
  padding: 2rem;
  z-index: 10;
}

.menu-btn {
  padding: 1.2rem 3rem;
  font-size: 1.3rem;
  font-weight: 900;
  border-radius: 20px;
  cursor: pointer;
  background: rgba(0, 0, 0, 0.6);
  border: 2px solid transparent;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 4px;
  position: relative;
  overflow: hidden;
  color: #fff;
  font-family: 'Courier New', Courier, monospace;
  box-shadow: 0 0 15px rgba(0,0,0,0.5);
  backdrop-filter: blur(4px);
}

.menu-btn::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: linear-gradient(45deg, transparent 5%, rgba(255,255,255,0.1) 50%, transparent 95%);
  z-index: -1;
}

.menu-btn.primary {
  border-color: #00ffcc;
  color: #00ffcc;
  text-shadow: 0 0 8px rgba(0, 255, 204, 0.8);
  box-shadow: 0 0 15px rgba(0, 255, 204, 0.3), inset 0 0 10px rgba(0, 255, 204, 0.1);
}

.menu-btn.primary:hover {
  background: rgba(0, 255, 204, 0.15);
  box-shadow: 0 0 25px rgba(0, 255, 204, 0.6), inset 0 0 15px rgba(0, 255, 204, 0.3);
  transform: translateY(-3px) scale(1.05);
}

.menu-btn.secondary {
  border-color: #ff00ff;
  color: #ff00ff;
  text-shadow: 0 0 8px rgba(255, 0, 255, 0.8);
  box-shadow: 0 0 15px rgba(255, 0, 255, 0.3), inset 0 0 10px rgba(255, 0, 255, 0.1);
}

.menu-btn.secondary:hover {
  background: rgba(255, 0, 255, 0.15);
  box-shadow: 0 0 25px rgba(255, 0, 255, 0.6), inset 0 0 15px rgba(255, 0, 255, 0.3);
  transform: translateY(-3px) scale(1.05);
}

@media (max-width: 768px) {
  .button-section {
    padding-bottom: 5vh;
  }
  .buttons-wrapper {
    gap: 1rem;
    padding: 0.5rem;
    width: 100%;
    max-width: 320px;
  }
  .menu-btn {
    width: 100%;
    padding: 0.8rem 1rem;
    font-size: 0.69rem;
  }
}

@media (max-height: 500px) and (orientation: landscape) {
  .intro-container {
    background-image: none;
  }
  .bg-video {
    object-fit: cover;
  }
  .button-section {
    padding-bottom: 2vh;
  }
  .buttons-wrapper {
    flex-direction: row;
    width: auto;
    max-width: none;
    gap: 1rem;
    padding: 0.5rem;
  }
  .menu-btn {
    width: auto;
    padding: 0.8rem 1.5rem;
    font-size: 1rem;
  }
}
</style>
