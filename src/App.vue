<script setup lang="ts">
import { ref } from 'vue';

import Tesseract from 'tesseract.js';

const result = ref('');
const file = ref<File | null>(null);
const imageUrl = ref('');
const inputRef = ref<HTMLInputElement | null>(null);

const handleChangeFile = (e: Event) => {
  const target = e.target as HTMLInputElement;
  if (target && target.files) {
    file.value = target.files[0];
    imageUrl.value = URL.createObjectURL(file.value);
  }
}

const handleExtract = () => {
  if (!file.value) {
    result.value = 'No file selected for OCR.';
    return;
  }

  result.value = 'Processing...';

  const reader = new FileReader();
  reader.onload = () => {
    if (reader.result) {
      Tesseract.recognize(reader.result as string, 'eng', {
        logger: (info) => console.log(info),
      })
        .then(({ data: { text } }) => {
          result.value = text;
        })
        .catch((error) => {
          console.error('Error OCR: ', error);
          result.value = 'Error processing the image.';
        });
    }
  };

  reader.readAsDataURL(file.value);
};

const handleRemove = () => {
  file.value = null;
  imageUrl.value = '';
  result.value = '';

  if (inputRef.value) {
    inputRef.value.value = '';
  }
}
</script>

<template>
  <main>
    <div class="center-flex">
      <h1 class="font-bold text-4xl text-blue-300">OCR + Vue</h1>
    </div>
    <div class="center-flex my-5">
      <input type="file" ref="inputRef" @change="handleChangeFile" />
      <div class="flex gap-2" v-if="file">
        <button class="bg-blue-600 text-white p-1 rounded-md" @click="handleExtract">Extract</button>
        <button class="bg-red-600 text-white p-1 rounded-md" @click="handleRemove">Remove</button>
      </div>
    </div>
    <div class="center-flex my-5" v-if="imageUrl">
      <img alt="the image" :src="imageUrl" width="500px" />
    </div>
    <div class="center-flex my-5" v-show="result">
      <h4 class="font-bold text-gray-300 text-2xl block">Result:</h4>
    </div>
    <p class="text-white text-justify">{{ result }}</p>
  </main>
</template>

<style scoped>
.center-flex {
  display: flex;
  justify-content: center;
}
</style>
