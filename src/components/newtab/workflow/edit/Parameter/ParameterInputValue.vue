<template>
  <div v-if="paramData.type === 'file'" class="w-full">
    <label 
      class="cursor-pointer relative flex flex-col items-center justify-center w-full h-32 bg-gray-50 dark:bg-gray-800 border-2 border-dashed border-gray-300 dark:border-gray-600 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors"
    >
      <div class="flex flex-col items-center justify-center pt-5 pb-6">
        <v-remixicon name="riUploadCloud2Line" class="mb-3 text-gray-400" size="32" />
        <p class="mb-1 text-sm text-gray-500 dark:text-gray-400">
          <span class="font-semibold">{{ fileName || 'Klik untuk pilih file CSV' }}</span>
        </p>
        <p class="text-xs text-gray-400">.CSV atau .TXT (Otomatis Baca)</p>
      </div>
      <input 
        type="file" 
        class="hidden" 
        accept=".csv,.txt"
        @change="handleFileUpload" 
      />
    </label>
    <input type="hidden" :value="modelValue" />
  </div>

  <ui-input
    v-else
    :model-value="modelValue"
    :mask="mask"
    type="text"
    class="w-full"
    :placeholder="paramData.placeholder"
    @change="$emit('update:modelValue', $event)"
    @keyup.enter="$emit('execute')"
  />
</template>

<script setup>
import { computed, ref } from 'vue'; // Tambahkan 'ref'

const props = defineProps({
  modelValue: {
    type: String,
    default: '',
  },
  paramData: {
    type: Object,
    default: () => ({}),
  },
});
const emit = defineEmits(['update:modelValue', 'execute']);

// --- [MODIFIKASI] LOGIC FILE UPLOAD ---
const fileName = ref('');

function handleFileUpload(event) {
  const file = event.target.files[0];
  if (!file) return;

  // 1. Tampilkan nama file di UI agar user tahu
  fileName.value = file.name;

  // 2. Baca isi file
  const reader = new FileReader();
  reader.onload = (e) => {
    // 3. Masukkan ISI FILE (Text) ke dalam variabel parameter Automa
    // Jadi 'modelValue' nanti isinya bukan path, tapi teks mentah CSV-nya.
    emit('update:modelValue', e.target.result);
  };
  reader.readAsText(file);
}
// ---------------------------------------

// --- [ASLI] LOGIC MASKING BAWAAN AUTOMA ---
const mask = computed(() => {
  const options = props.paramData.data;
  if (!options || !options.useMask) return null;

  const masks = options.masks.map((item) => {
    const cloneMask = { ...item };
    if (cloneMask.isRegex) cloneMask.mask = new RegExp(cloneMask.mask);
    else cloneMask.mask = cloneMask.mask.replaceAll('!', '\\');

    delete cloneMask.isRegex;

    return cloneMask;
  });

  if (masks.length === 1) return masks[0];

  return {
    mask: masks,
  };
});
</script>