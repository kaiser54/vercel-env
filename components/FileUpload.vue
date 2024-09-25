<template>
  <div class=" w-full flex justify-start flex-col gap-2">
    <div
      v-if="!imageUploaded"
      class="w-full flex h-30 py-4 px-14 flex-col justify-center items-center gap-4 rounded-[12px] border border-dashed border-border-on-bg bg-background-canvas cursor-pointer"
      @dragover.prevent
      @dragenter.prevent
      @drop="handleDrop"
      @click="triggerFileInput"
    >
      <ICloud />
      <slot />
      <input
        type="file"
        ref="fileInput"
        accept="image/png, image/jpeg, text/csv"
        class="hidden"
        @change="handleFileSelect"
      />
    </div>

    <div
      v-if="imageUploaded"
      :class="{
        'border border-background-active': !imageUploading,
        'border-transparent': imageUploading,
      }"
      class="relative flex flex-col items-center rounded-[12px]"
    >
      <input
        type="file"
        ref="fileInput"
        accept="image/png, image/jpeg, text/csv"
        class="hidden"
        @change="handleFileSelect"
      />
      <div
        class="relative w-full h-30 rounded-[12px]"
        :class="{
          'bg-background-canvas': fileType === 'csv',
          'bg-transparent': fileType === 'image',
        }"
      >
        <div v-if="fileType === 'csv'" class="flex gap-4 items-center p-4">
          <div
            class="flex p-3 items-center gap-2 rounded-lg bg-white shadow z-[5]"
          >
            <img class="h-[30px]" src="./svg/icons/excel.svg" alt="CSV icon" />
          </div>

          <div class="flex flex-col gap-1 items-start">
            <p class="text-sm text-default">{{ truncatedFileName }}</p>
            <p v-if="fileSize !== null" class="text-neutral text-sm">{{ formatFileSize(fileSize) }}</p>
          </div>
          <Button
            v-if="!imageUploading && fileType === 'csv'"
            class="h-8 w-8 bg-transparent ml-auto hover:bg-transparent active:bg-transparent rounded-tr-[12px]"
            @click="deleteImage"
            ><ITrash
          /></Button>
        </div>

        <img
          v-else
          class="h-[120px] mx-auto max-w-full"
          :src="imagePreviewUrl ?? undefined"
          alt="Uploaded image"
        />
        <p
          class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-on-accent-bg font-medium text-[24px] leading-[32px] tracking[-0.2px] whitespace-nowrap m-0 p-0 z-[3]"
          v-if="imageUploading"
        >
          {{ uploadProgress }}
          <span
            class="text-[14px] leading-[21px] tracking[0.1px] font-normal text-on-accent-bg"
            >%</span
          >
        </p>
        <div
          v-if="imageUploading"
          class="absolute top-0 left-0 w-full h-auto bg-[rgba(255,255,255,0.7)] backdrop-blur-xs flex flex-col justify-center items-center"
        >
          <div
            class="w-full bg-transparent rounded-sm overflow-hidden mb-[10px] relative"
            :class="{ 'h-[120px]': fileType === 'image', 'h-[88px]': fileType === 'csv' }"
          >
            <div
              class="h-full bg-background-accent-bg transition-width duration-300 ease-in-out relative z-[1] rounded-[12px]"
              :style="{ width: `${uploadProgress}%` }"
            ></div>
          </div>
        </div>
        <div
          class="flex absolute top-[0px] right-[0px]"
          v-if="!imageUploading && fileType === 'image'"
        >
          <Button
            class="h-8 w-8 bg-transparent hover:bg-transparent active:bg-transparent rounded-none"
            @click="triggerFileInput"
          >
            <IEdit />
          </Button>
          <Button
            class="h-8 w-8 bg-transparent hover:bg-transparent active:bg-transparent rounded-tr-[12px]"
            @click="deleteImage"
            ><ITrash
          /></Button>
        </div>
      </div>
    </div>
    <p
      v-if="errorMessage"
      class="text-negative text-[12px] font-normal leading-[18px] tracking-[0.1px]"
    >
      {{ errorMessage }}
    </p>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import { ICloud, IEdit, ITrash } from "~/components/svg";

const fileInput = ref<HTMLInputElement | null>(null);
const uploadProgress = ref<number>(0);
const imagePreviewUrl = ref<string | null | undefined>(null);
const imageUploaded = ref<boolean>(false);
const imageUploading = ref<boolean>(false);
const fileName = ref<string | null>(null);
const fileSize = ref<number | null>(null);
const errorMessage = ref<string | null>(null);

const props = defineProps<{
  type: "image" | "csv";
}>();

const emit = defineEmits(["file-uploaded"]);

const fileType = props.type;

const truncatedFileName = computed(() => {
  if (fileName.value && fileName.value.length > 40) {
    return fileName.value.substring(0, 40) + "...";
  }
  return fileName.value;
});

function triggerFileInput() {
  if (fileInput.value) {
    fileInput.value.click();
  }
}

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (target.files) {
    validateAndUploadFile(target.files[0]);
  }
};

const handleDrop = (event: DragEvent) => {
  event.preventDefault();
  const files = event.dataTransfer?.files;
  if (files && files.length > 0) {
    validateAndUploadFile(files[0]);
  }
};

const formatFileSize = (size: number): string => {
  if (size < 1024) {
    return `${size} bytes`;
  } else if (size < 1024 * 1024) {
    return `${(size / 1024).toFixed(2)} KB`;
  } else {
    return `${(size / (1024 * 1024)).toFixed(2)} MB`;
  }
};

const validateAndUploadFile = (file: File) => {
  errorMessage.value = null;
  if (props.type === "csv") {
    if (file.type !== "text/csv") {
      errorMessage.value = "Invalid file type. Only CSV files are allowed.";
      return;
    }
    if (file.size > 25 * 1024 * 1024) {
      errorMessage.value = "CSV file size exceeds the limit of 25MB.";
      return;
    }
  } else if (props.type === "image") {
    if (!["image/jpeg", "image/png"].includes(file.type)) {
      errorMessage.value = "Invalid file type. Only JPEG and PNG images are allowed.";
      return;
    }
    if (file.size > 1 * 1024 * 1024) {
      errorMessage.value = "Image file size exceeds the limit of 1MB.";
      return;
    }
  }
  uploadFile(file);
};

function uploadFile(file: File) {
  fileName.value = file.name;
  fileSize.value = file.size;

  const reader = new FileReader();

  reader.onload = (e) => {
    if (fileType === "image") {
      imagePreviewUrl.value = e.target?.result as string;
      localStorage.setItem("uploadedImage", imagePreviewUrl.value);
    } else if (fileType === "csv") {
      imagePreviewUrl.value = "./svg/icons/excel.svg";
      localStorage.setItem("uploadedFileName", fileName.value || "");
      localStorage.setItem("uploadedFileSize", fileSize.value?.toString() || "");
    }
    imageUploaded.value = true;
    imageUploading.value = true;
    uploadProgress.value = 0;
    const interval = setInterval(() => {
      if (uploadProgress.value < 100) {
        uploadProgress.value += 10;
      } else {
        clearInterval(interval);
        imageUploading.value = false;
        uploadProgress.value = 0;
        emit("file-uploaded", file);
      }
    }, 300);
  };
  reader.readAsDataURL(file);
}

function deleteImage() {
  imageUploaded.value = false;
  imagePreviewUrl.value = null;
  errorMessage.value = null;
  localStorage.removeItem("uploadedImage");
  localStorage.removeItem("uploadedFileName");
  localStorage.removeItem("uploadedFileSize");
}

onMounted(() => {
  const savedImage = localStorage.getItem("uploadedImage");
  const savedFileName = localStorage.getItem("uploadedFileName");
  const savedFileSize = localStorage.getItem("uploadedFileSize");

  if (savedImage) {
    imagePreviewUrl.value = savedImage;
    imageUploaded.value = true;
  }
  if (savedFileName && savedFileSize) {
    fileName.value = savedFileName;
    fileSize.value = parseInt(savedFileSize);
  }
});
</script>

<style scoped>
</style>
