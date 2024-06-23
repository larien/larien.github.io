<template>
  <div class="typewriter-container">
    <vue-typer
    :text="sentences"
  :repeat='Infinity'
  :shuffle='true'
  initial-action='typing'
  :pre-type-delay='70'
  :type-delay='70'
  :pre-erase-delay='2000'
  :erase-delay='100'
  erase-style='backspace'
  :erase-on-complete='true'
  caret-animation='phase'
  class="typewriter"></vue-typer>
  </div>
</template>

<script>
import { VueTyper } from 'vue-typer';

export default {
  name: 'TypewriterComponent',
  components: {
    VueTyper
  },
  props: {
    sentences: {
      type: Array,
      required: true
    }
  }
};
</script>

<style scoped>
.typewriter-container {
  display: inline-block;
}

.typewriter {
  background-color: white;
  font-size: 48px; /* Adjust size as needed */
  font-weight: bold;
  font-family: 'RocknRoll One', sans-serif;
}

/* Media queries for mobile devices */
@media (max-width: 768px) {
  .typewriter {
    font-size: 24px;
  }
}
</style>
