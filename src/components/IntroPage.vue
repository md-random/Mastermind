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
      <div class="blossom-panel left"></div>
      <div class="buttons-wrapper">
        <button class="menu-btn secondary" @click="$emit('showRules')">Gameplay</button>
        <button class="menu-btn primary" @click="$emit('playGame')">Play Game</button>
      </div>
      <div class="blossom-panel right"></div>
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
      const crossfadeDuration = 3 // Crossfade over 3 seconds structurally
      const stepTime = (crossfadeDuration * 1000) / steps
      
      const fadeInterval = setInterval(() => {
        progress++
        const ratio = progress / steps
        
        if (!isMuted.value) {
          if (bgVideo.value && !isVideoEnded) { // fade out video
            bgVideo.value.volume = Math.max(0, startVideoVol * (1 - ratio))
          }
          if (bgAudio.value) { // fade in mp3
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
      if (timeLeft <= 3.0) { // start crossfade precisely 3 seconds before end
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
        startCrossfade() // Edgecase fallback if timeupdate misfires
      }
    }

    return { bgVideo, bgAudio, onVideoEnded, onVideoTimeUpdate, isMuted, volume, toggleMute, onVolumeChange }
  }
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400..900;1,400..900&display=swap');

.intro-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: 100vw;
  background: radial-gradient(circle at 50% 30%, #3a5c7c 0%, #111a24 100%);
  background-color: #111a24;
  font-family: 'Playfair Display', serif;
  overflow-x: hidden;
}

.video-section {
  width: 100%;
  aspect-ratio: 16 / 9; 
  position: relative;
  overflow: hidden;
  
  padding: 8px;
  box-sizing: border-box; 
  background: linear-gradient(135deg, #2b4159, #1a2a3a, #0d1620, #2b4159);
  box-shadow: 
    inset 0 1px 3px rgba(255, 255, 255, 0.3), 
    inset 0 -2px 5px rgba(0, 0, 0, 0.8),
    0 1px 2px rgba(0, 255, 204, 0.8), 
    0 5px 15px rgba(0, 255, 204, 0.2); 
  z-index: 10; 
}

/* Carbon Steel Screen Glass Reflection Overlay */
.video-section::after {
  content: '';
  position: absolute;
  top: 8px; left: 8px; right: 8px; bottom: 8px;
  background: linear-gradient(105deg, rgba(255, 255, 255, 0.05) 0%, rgba(255, 255, 255, 0.01) 30%, transparent 35%);
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.5); /* Inner shadow visually minimized by ~76% */
  pointer-events: none;
  border-radius: 10px; /* match the inner video curve */
  z-index: 10;
}



.video-controls {
  position: absolute;
  bottom: 8px; /* Tighter to the corner */
  right: 10px; /* Tighter to the corner */
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(0, 0, 0, 0.8); /* Darker so it fully hides the watermark text */
  padding: 6px 14px;
  border-radius: 30px;
  z-index: 30;
  transform: scale(0.85); /* Slightly smaller footprint */
  transform-origin: bottom right;
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
  object-fit: cover; 
  display: block;
  border-radius: 10px; 
  transform: scale(1.04); /* Zooms past any baked-in video watermarks or encoded letterbox margins */
}

.button-section {
  flex: 1; 
  display: flex;
  justify-content: space-between;
  align-items: stretch;
  background-color: transparent;
  padding: 0;
  position: relative;
}

.blossom-panel {
  flex: 1;
  background-image: url('../assets/CherryBlossomMastermind.png');
  background-size: cover;
  background-repeat: no-repeat;
  opacity: 0.85;
}

.blossom-panel.left {
  background-position: left center;
  -webkit-mask-image: linear-gradient(to right, rgba(0,0,0,1) 5%, rgba(0,0,0,0) 95%);
  mask-image: linear-gradient(to right, rgba(0,0,0,1) 5%, rgba(0,0,0,0) 95%);
}

.blossom-panel.right {
  background-position: right center;
  -webkit-mask-image: linear-gradient(to left, rgba(0,0,0,1) 5%, rgba(0,0,0,0) 95%);
  mask-image: linear-gradient(to left, rgba(0,0,0,1) 5%, rgba(0,0,0,0) 95%);
}

.buttons-wrapper {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 2rem;
  padding: 2rem;
  flex: 0 1 auto;
  min-width: 320px;
  z-index: 10;
}

.menu-btn {
  padding: 1.2rem 3rem;
  font-size: 1.3rem;
  font-weight: 900;
  border-radius: 6px;
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
  border-color: #00ffcc; /* Neon Cyan to match the system blue */
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
  border-color: #ff00ff; /* Neon Magenta to match the matrix demons */
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
    flex-direction: row;
    padding: 0;
  }
  .buttons-wrapper {
    gap: 1.5rem;
    padding: 1rem;
    min-width: 220px;
  }
  .menu-btn {
    width: 100%;
    padding: 1rem 1.5rem;
    font-size: 1.15rem;
  }
}
</style>
