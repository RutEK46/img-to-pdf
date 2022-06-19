<template>
  <q-file
    v-model="files"
    label="Convert Images to PDF File"
    filled
    multiple
    use-chips
    style="max-width: 300px"
    accept=".jpg, image/*"
    @update:model-value="onModelChange"
  />
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';

export default defineComponent({
	name: 'ImageForm',
	setup() {
		return {
			files: ref<File[]>()
		}
	},
	methods: {
		onModelChange() { 
      if (this.files) {
			  this.$emit('images-submit', this.files);
      }
		}
	},
});
</script>