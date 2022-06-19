<template>
  <q-page class="row items-center justify-evenly">
    <image-form 
      v-if="!isConverting"
      @images-submit="onImagesSubmit"
    />
    <image-display
      v-else
      v-bind:progress="progress"
      v-bind:max-progress="maxProgress"
      v-bind:image-source="imgBase64"
    />
  </q-page>
</template>

<script lang="ts">
import { jsPDF } from 'jspdf';
import { PdfImage } from 'components/models';
import ImageForm from 'components/ImageForm.vue';
import ImageDisplay from 'components/ImageDisplay.vue';
import { defineComponent, ref } from 'vue';


async function getBase64(file: File): Promise<string | undefined> {
  return await new Promise<string | undefined>((resolve) => {
      let fileReader = new FileReader();
      fileReader.onload = () => {
        resolve(fileReader.result?.toString());
      }
      fileReader.readAsDataURL(file);
  });
}

async function getPdfImageData(base64: string): Promise<PdfImage> {
  return await new Promise<PdfImage>((resolve) => {
    var image = new Image();
    image.onload = () => resolve({ 
      base64: base64,
      width: image.width,
      height: image.height,
    });
    image.src = base64;
  });
}

export default defineComponent({
  name: 'IndexPage',
  components: { ImageForm, ImageDisplay },
  setup() {
    return {
      imgBase64: ref<string | undefined>(),
      progress: ref(0),
      maxProgress: ref(0),
      isConverting: ref(false),
    }
  },
  methods: {
    async onImagesSubmit(files: File[]) {
      this.isConverting = true;
      this.maxProgress = files.length;
      this.progress = 0;
      var pdfImages = this.getImagesFromFiles(files);
      var pdf = await this.convertImageToPdf(pdfImages);
      window.open(pdf.output('bloburl'), '_blank');
      setTimeout(() => {
        this.isConverting = false;
      }, 1000);
    },

    async convertImageToPdf(pdfImages: AsyncGenerator<PdfImage>): Promise<jsPDF> {
      var doc = new jsPDF(undefined, 'pt', undefined);
      doc.deletePage(1);
      var numberOfFilesConverted = 0
      for await (let imgData of pdfImages) {
        doc.addPage();
        doc.internal.pageSize.width = imgData.width;
        doc.internal.pageSize.height = imgData.height;
        doc.addImage(imgData.base64, 'JPEG', 0, 0, imgData.width, imgData.height);
        numberOfFilesConverted++;
        if (numberOfFilesConverted % 8 == 0) {
          this.progress = numberOfFilesConverted;
        }
      }
      this.progress = this.maxProgress;

      return doc;
    },

    async * getImagesFromFiles(files: File[]): AsyncGenerator<PdfImage> {
      for (let file of files) {
        let base64 = await getBase64(file);
        if (base64 !== undefined) {
          this.imgBase64 = base64;
          yield await getPdfImageData(base64);
        }
      }
    }
  }
});
</script>
