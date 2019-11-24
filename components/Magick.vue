<template>
  <no-ssr>
    <div class="container">
      <div class="my-12">
        <input type="file" name="" id="" @change="onImageChanged" />
      </div>
      <vue-compare-image
        v-if="image_modified !== ''"
        :leftImage="image_original.thumbnail"
        :rightImage="image_modified" />
    </div>
  </no-ssr>
</template>

<script>
import * as Magick from 'wasm-imagemagick'
export default {
  components: {
    VueCompareImage: () => import('vue-compare-image')
  },
  data: () => ({
    convert_options: '[ "convert", "srcFile.jpg", "-charcoal", "2", "out.png" ]',
    image_original: {
      name: "",
      image: null,
      thumbnail: "",
    },
    image_modified: "",
  }),
  watch: {
    'image_original': {
      handler: function(val, old_val) {
        if (val.name == '') return
        if (val.image == null) return

         this.doMagick()
      },
      deep: true
    }
  },
  methods: {
    onImageChanged(event) {
      const files = event.target.files || event.dataTransfer.files
      if (files && files[0]) {
        let obj = {}
        const fileReader = new FileReader()
        fileReader.addEventListener('load', () => {
          this.image_original.name = files[0].name
          if (this.image_original.name && this.image_original.name.lastIndexOf('.') <= 0) return

          this.image_original.thumbnail = fileReader.result
          this.image_original.image = files[0]
        })
        fileReader.readAsDataURL(files[0])
      }
    },
    async doMagick() {
      const sourceBytes = new Uint8Array(await this.image_original.image.arrayBuffer());

      const inputFiles = [{ name: this.image_original.name, content: sourceBytes }]
      let command = []
      command = JSON.parse('[ "convert", "' + this.image_original.name + '", "-type", "GrayScale", "out.png" ]')

      const processedFiles = await Magick.Call(inputFiles, command);

      const firstOutputImage = processedFiles[0];
      console.log(firstOutputImage)
      this.image_modified = URL.createObjectURL(firstOutputImage["blob"]);
      console.log("created image " + firstOutputImage["name"]);

    }
  },
}
</script>
