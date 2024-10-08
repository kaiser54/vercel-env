<template>
  <div class="main-con">
    <div class="input-container-wrap">
      <div id="inputContainer" class="border first">
        <EnvInput
          v-for="(input, index) in inputs"
          :key="index"
          :input="input"
          @delete="deleteInput(index)"
          @update="updateInput(index, $event)"
          @paste="handlePaste($event, index)"
        />
      </div>
    </div>
    <div class="border btn-group">
      <button id="addButton" @click="addInput">
        <span class="svg" v-html="plusSvg"></span>
        <span>Add another</span>
      </button>
      <p class="snippet">or paste the .env contents above</p>
    </div>
    <input type="file" id="fileInput" accept=".env" @change="handleFileUpload" />
    <DropZone @file-dropped="handleFileDrop" />
  </div>
</template>

<script>
import { ref } from 'vue';
import EnvInput from './components/EnvInput.vue';
import DropZone from './components/DropZone.vue';
import { plusSvg } from '@/components/svg';

export default {
  components: {
    EnvInput,
    DropZone,
  },
  setup() {
    const inputs = ref([{ key: '', value: '', note: '' }]);

    const addInput = () => {
      inputs.value.push({ key: '', value: '', note: '' });
    };

    const deleteInput = (index) => {
      if (inputs.value.length > 1) {
        inputs.value.splice(index, 1);
      }
    };

    const updateInput = (index, newData) => {
      inputs.value[index] = { ...inputs.value[index], ...newData };
    };

    const handleFileUpload = (event) => {
      const file = event.target.files[0];
      if (file) {
        handleFile(file);
      }
    };

    const handleFileDrop = (file) => {
      handleFile(file);
    };

    const handleFile = (file) => {
      const reader = new FileReader();
      reader.onload = (e) => {
        const text = e.target.result;
        parseAndAppendEnvFile(text);
      };
      reader.onerror = (e) => {
        console.error("File reading error:", e);
      };
      reader.readAsText(file);
    };

    const parseAndAppendEnvFile = (text, startIndex = inputs.value.length) => {
      const lines = text.split("\n");
      let currentIndex = startIndex;

      lines.forEach((line) => {
        if (line.trim() && !line.startsWith("#")) {
          const [key, value] = line.split("=");
          if (key && value !== undefined) {
            if (currentIndex < inputs.value.length) {
              // Update existing input
              inputs.value[currentIndex] = { key: key.trim(), value: value.trim(), note: '' };
            } else {
              // Add new input
              inputs.value.push({ key: key.trim(), value: value.trim(), note: '' });
            }
            currentIndex++;
          }
        }
      });
    };

    const handlePaste = (event, index) => {
      event.preventDefault();
      const pastedText = event.clipboardData.getData('text');
      parseAndAppendEnvFile(pastedText, index);
    };

    return {
      inputs,
      addInput,
      deleteInput,
      updateInput,
      handleFileUpload,
      handleFileDrop,
      handlePaste,
      plusSvg,
    };
  },
};
</script>


<style>
/* @import 'geist/font.css'; */

:root {
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  transition-duration: 0.26s;
  font-family: 'Geist', sans-serif;
}

body {
  min-height: 100vh;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.main-con {
  max-width: 500px;
  width: 100vw;
}

h1 {
  font-size: 24px;
  font-weight: 600;
  line-height: 36px;
}

header {
  margin-bottom: 24px;
}

.input-container-wrap {
  width: 100%;
}

.input-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.input-wrap {
  width: 100%;
  display: flex;
  flex-grow: 1;
  align-items: center;
  justify-content: center;
  gap: 16px;
}

input {
  height: 32px;
}

input,
textarea {
  display: flex;
  width: max-content;
  flex-grow: 1;
  padding: 0px 8px;

  outline: 0;
  border: 0;

  border-radius: 6px;
  background: #fff;

  box-shadow:
    0px 1px 2px 0px rgba(0, 0, 0, 0.12),
    0px 0px 0px 1px rgba(0, 0, 0, 0.08);

  color: #18181b;
  font-size: 13px;
  font-style: normal;
  font-weight: 400;
  line-height: 20px; /* 153.846% */
}

textarea {
  width: 100%;
  flex-grow: 1;
  padding: 8px !important;
  display: flex;
  min-height: 32px;
  max-height: 120px;
}

input::placeholder,
textarea::placeholder {
  color: #a1a1aa;
  font-size: 12px;
  font-style: normal;
  font-weight: 400;
  line-height: 20px; /* 153.846% */
}

input:focus,
textarea:focus {
  border-radius: var(--radius-6, 6px);
  background: var(--backgrounds-bg-field-component, #fff);

  /* Light/Borders/interactive-with-active */
  box-shadow:
    0px 0px 0px 1px #3b82f6,
    0px 0px 0px 4px rgba(59, 130, 246, 0.2);
}

input:hover,
textarea:hover {
  background: var(--backgrounds-bg-field-component-hover, #fafafa);
}

.action-btn-group {
  display: flex;
  gap: 8px;
  align-items: center;
}

button.icon-button {
  display: inline-flex;
  padding: 8.5px;
  justify-content: center;
  align-items: center;

  border-radius: var(--radius-6, 6px);
  background: var(--buttons-button-neutral, #fff);

  /* Light/Buttons/neutral */
  box-shadow:
    0px 1px 2px 0px rgba(0, 0, 0, 0.12),
    0px 0px 0px 1px rgba(0, 0, 0, 0.08);
}

button.icon-button:hover {
  background: var(--buttons-button-neutral-hover, #f4f4f5);
}

button.icon-button:active {
  border-radius: var(--radius-6, 6px);
  background: var(--buttons-button-neutral-pressed, #e4e4e7);

  /* Light/Buttons/neutral */
  box-shadow:
    0px 1px 2px 0px rgba(0, 0, 0, 0.12),
    0px 0px 0px 1px rgba(0, 0, 0, 0.08);
}

button {
  display: inline-flex;
  padding: 6px 10px;
  justify-content: center;
  align-items: center;
  gap: 6px;

  border: none;
  cursor: pointer;

  border-radius: 6px;
  background: var(--buttons-button-inverted, #27272a);

  /* Light/Buttons/inverted */
  box-shadow:
    0px 0.75px 0px 0px rgba(255, 255, 255, 0.2) inset,
    0px 1px 2px 0px rgba(0, 0, 0, 0.4),
    0px 0px 0px 1px #18181b;

  color: rgba(255, 255, 255, 0.88);
  font-size: var(--font-size-small, 13px);
  font-style: normal;
  font-weight: var(--font-weight-500, 500);
  line-height: 20px; /* 153.846% */
}

textarea::-webkit-resizer {
  background-image: url("data:image/svg+xml,%3Csvg width='15' height='15' viewBox='0 0 15 15' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M13.182 8.93937L8.93933 13.182' stroke='%23A1A1AA' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3Cpath d='M12.3033 3.81806L3.81805 12.3033' stroke='%23A1A1AA' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 8px bottom 6px;
  background-size: 60%;
  padding-right: 8px;
  padding-bottom: 6px;
}

textarea::-moz-resizer {
  background-image: url("data:image/svg+xml,%3Csvg width='15' height='15' viewBox='0 0 15 15' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M13.182 8.93937L8.93933 13.182' stroke='%23A1A1AA' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3Cpath d='M12.3033 3.81806L3.81805 12.3033' stroke='%23A1A1AA' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 8px bottom 6px;
  background-size: 60%;
  padding-right: 8px;
  padding-bottom: 6px;
}

.drop-zone {
  display: inline-flex;
  padding: var(--spacing-32, 32px);
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 4px;
  margin-top: 16px;

  width: 100%;

  border-radius: var(--radius-8, 8px);
  border: 1px dashed var(--borders-border-strong, #d4d4d8);
  background: var(--backgrounds-bg-component, #fafafa);
}

.icon-title {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  align-self: stretch;
}

.import-title {
  color: var(--foregrounds-fg-subtle, #52525b);
  text-align: center;

  font-size: var(--font-size-small, 13px);
  font-style: normal;
  font-weight: var(--font-weight-500, 500);
  line-height: 20px; /* 153.846% */
}

.subtitle,
.snippet,
.import-snippet {
  color: var(--foregrounds-fg-muted, #a1a1aa);
  text-align: center;

  font-size: var(--font-size-small, 13px);
  font-style: normal;
  font-weight: var(--font-weight-400, 400);
  line-height: 20px; /* 153.846% */
}

.subtitle,
.snippet {
  font-size: 14px;
}

.subtitle {
  text-align: left;
}

.drop-zone:hover {
  border: 1px dashed var(--borders-border-interactive, #3b82f6);
}

.drop-zone:active,
#dropZone.dragover {
  border: 1px solid var(--borders-border-interactive, #3b82f6);

  /* Light/Borders/focus */
  box-shadow:
    0px 0px 0px 1px #fff,
    0px 0px 0px 3px rgba(59, 130, 246, 0.6);
}

.border {
  border: 1px solid #e4e4e7;
  padding: 16px;
  border-radius: 6px;
}

.border.first {
  border-bottom-style: dashed;
}

.btn-group {
  margin-top: -1px;
  border-top-style: dashed;
  display: flex;
  align-items: center;
  gap: 16px;
}

.svg {
  display: flex;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    max-height: 0;
  }

  to {
    opacity: 1;
    max-height: 200px;
  }
}

@keyframes fadeOut {
  from {
    opacity: 1;
    max-height: 200px;
  }

  to {
    opacity: 0;
    max-height: 0;
  }
}

#fileInput {
  display: none;
}

.fade-in {
  animation: fadeIn 0.8s forwards;
}

.fade-out {
  animation: fadeOut 0.2s forwards;
}
</style>
