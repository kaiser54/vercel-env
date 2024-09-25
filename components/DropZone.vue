<template>
    <div
      id="dropZone"
      class="drop-zone"
      @dragover.prevent="dragover"
      @dragleave="dragleave"
      @drop.prevent="drop"
      @click="triggerFileInput"
      :class="{ dragover: isDragover }"
    >
      <div class="icon-title">
        <div v-html="arrowTray"></div>
        <p class="import-title">Import Files</p>
      </div>
      <p class="import-snippet">
        Drag and drop files here or click to upload
      </p>
    </div>
  </template>
  
  <script>
  import { ref } from 'vue';
  import { arrowTray } from './svg';
  
  export default {
    emits: ['file-dropped'],
    setup(props, { emit }) {
      const isDragover = ref(false);
  
      const dragover = (e) => {
        isDragover.value = true;
      };
  
      const dragleave = (e) => {
        isDragover.value = false;
      };
  
      const drop = (e) => {
        isDragover.value = false;
        const files = e.dataTransfer.files;
        if (files.length > 0) {
          emit('file-dropped', files[0]);
        }
      };
  
      const triggerFileInput = () => {
        document.getElementById('fileInput').click();
      };
  
      return {
        isDragover,
        dragover,
        dragleave,
        drop,
        triggerFileInput,
        arrowTray,
      };
    },
  };
  </script>
  
  <style scoped>
  /* Include the necessary styles here */
  </style>