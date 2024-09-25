<template>
    <div class="input-container fade-in">
      <div class="input-wrap">
        <input
          type="text"
          placeholder="e.g. CLIENT_KEY"
          class="key-input"
          :value="input.key"
          @input="updateInput('key', $event.target.value)"
        />
        <input
          type="text"
          placeholder="aBb123XyZ"
          class="value-input"
          :value="input.value"
          @input="updateInput('value', $event.target.value)"
        />
        <div class="action-btn-group">
          <button class="note-btn icon-button" @click="toggleNote" v-html="pencilSvg"></button>
          <button class="delete-btn icon-button" @click="$emit('delete')" v-html="trashSvg"></button>
        </div>
      </div>
      <textarea
        v-if="showNote"
        class="note-input"
        placeholder="Enter your text here..."
        :value="input.note"
        @input="updateInput('note', $event.target.value)"
      ></textarea>
    </div>
  </template>
  
  <script>
  import { ref } from 'vue';
  import { pencilSvg, trashSvg } from './svg';
  
  export default {
    props: ['input'],
    emits: ['delete', 'update'],
    setup(props, { emit }) {
      const showNote = ref(false);
  
      const toggleNote = () => {
        showNote.value = !showNote.value;
      };
  
      const updateInput = (field, value) => {
        emit('update', { [field]: value });
      };
  
      return {
        showNote,
        toggleNote,
        updateInput,
        pencilSvg,
        trashSvg,
      };
    },
  };
  </script>
  
  <style scoped>
  /* Include the necessary styles here */
  </style>