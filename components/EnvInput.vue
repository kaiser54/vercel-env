<template>
  <div class="input-container fade-in">
    <div class="input-wrap">
      <input
        type="text"
        placeholder="e.g. CLIENT_KEY"
        class="key-input"
        :value="input.key"
        @input="updateInput('key', $event.target.value)"
        @paste="handlePaste($event, 'key')"
      />
      <input
        type="text"
        placeholder="aBb123XyZ"
        class="value-input"
        :value="input.value"
        @input="updateInput('value', $event.target.value)"
        @paste="handlePaste($event, 'value')"
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
      @paste="handlePaste($event, 'note')"
    ></textarea>
  </div>
</template>

<script>
import { ref } from 'vue';
import { pencilSvg, trashSvg } from './svg';

export default {
  props: ['input'],
  emits: ['delete', 'update', 'paste'],
  setup(props, { emit }) {
    const showNote = ref(false);

    const toggleNote = () => {
      showNote.value = !showNote.value;
    };

    const updateInput = (field, value) => {
      emit('update', { [field]: value });
    };

    const handlePaste = (event, field) => {
      const pastedText = event.clipboardData.getData('text');
      const lines = pastedText.split('\n');

      if (lines.length > 1 || props.input.key || props.input.value) {
        // If there are multiple lines or the current input is filled,
        // let the parent component handle the paste
        emit('paste', event);
      } else {
        // If there's only one line and the current input is empty,
        // update this field
        event.preventDefault();
        if (field === 'key') {
          const [key, value] = lines[0].split('=');
          updateInput('key', key.trim());
          if (value) {
            updateInput('value', value.trim());
          }
        } else {
          updateInput(field, lines[0].trim());
        }
      }
    };

    return {
      showNote,
      toggleNote,
      updateInput,
      handlePaste,
      pencilSvg,
      trashSvg,
    };
  },
};
</script>

<style scoped>
/* Include the necessary styles here */
</style>