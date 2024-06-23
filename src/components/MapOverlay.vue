<template>
  <div class="overlay">
    <img src="@/assets/logo.svg" alt="Logo" class="logo">
    <div class="overlay-content">
      <h1 class="welcome-text">Hello!</h1>
      <h1 class="name-text">My name is <span class="highlight-text">Lauren</span> and I'm</h1>
      <TypewriterComponent :sentences="sentences" />
      <h1 class="location-text">based in <span class="highlight-text">Florianópolis</span>, Brazil</h1>
    </div>
  </div>
</template>

<script>
import TypewriterComponent from './TypewriterComponent.vue';

export default {
  name: 'MapOverlay',
  components: {
    TypewriterComponent
  },
  data() {
    return {
      sentences: [
      'a Korra fangirl',
            'an amazon in training',
            'an overexcited dev',
            'a Golang developer',
            'a backpacker',
            'a beach lover',
            'a dog mom',
            'a tattoo addict'
      ]
    };
  }
};
</script>

<style scoped>
/* Existing styles */

.logo {
  position: absolute;
  top: 20px;
  left: 20px;
  width: 80px; /* Adjust size as needed */
  height: auto;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.overlay-content {
  text-align: center;
  color: white;
  font-family: 'RocknRoll One', sans-serif;
}

.welcome-text {
  font-size: 64px;
  margin-bottom: 20px;
}

.name-text, .location-text {
  font-size: 48px;
  margin-top: 10px;
}

.highlight-text {
  color: #1466c3; /* One of the blue tones */
}

/* Media queries for mobile devices */
@media (max-width: 768px) {
  .welcome-text {
    font-size: 32px;
  }

  .name-text, .location-text {
    font-size: 24px;
  }

  .typewriter-container .typewriter {
    font-size: 24px;
  }
}
</style>
