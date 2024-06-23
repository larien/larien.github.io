<template>
    <div>
       <vue-typer :text="sentences" :repeat="Infinity" class="typewriter" /> 
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
  
  <style>
  .typewriter {
    background-color: white;
    font-size: 32px;
    font-weight: bold;
  }
  </style>
  