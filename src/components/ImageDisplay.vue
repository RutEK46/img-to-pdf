<template>
  <div class="q-pa-md">
    <q-linear-progress 
      rounded
      size="50px"
      :value="_progress.value"
      class="q-mt-sm"
      style="--q-linear-progress-speed: 10ms"
    >
      <div class="absolute-full flex flex-center">
        <q-badge color="white" text-color="accent" :label="_progress.label" />
      </div>
    </q-linear-progress>
    <img v-bind:src="imageSource" />
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue';


interface Progress {
  value: number,
  label: string,
}


export default defineComponent({
  name: 'ImageDisplay',
  props: {
    progress: {
      type: Number,
      required: false,
      default: 0,
    },
    maxProgress: {
      type: Number,
      required: false,
      default: 0,
    },
    imageSource: {
      type: String as PropType<string | undefined>,
      required: false
    },
  },
  computed: {
    _progress(): Progress {
      let value = this.maxProgress !== 0
          ? this.progress / this.maxProgress : 0;

      let rounded = Math.round(value * 100);
      let label = `${rounded}%`;
      value = rounded / 100;
      return {
        value,
        label
      };
    },
  }
});
</script>