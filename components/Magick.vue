<template>
  <no-ssr>
    <div class="container">
      <div class="my-12 bg-gray-300 text-gray-800">
        <p class="text-center text-lg">image modifier</p>
      </div>
      <div class="my-12">
        <label class="block border-4 border-dashed cursor-pointer border-green-400 bg-green-200 text-green-800 flex items-center justify-center p-8 hover:opacity-50">
          <p>クリック/ドラックアンドドロップをして画像を選択</p>
          <input type="file" class="hidden" name="" id="" @change="onImageChanged" />
        </label>
      </div>
      <div class="my-12 border border-gray-800">
        <vue-compare-image
          v-if="image_modified !== ''"
          :leftImage="image_original.thumbnail"
          :rightImage="image_modified"
        />
      </div>
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
      Magick.Call([], ['convert', '-list', 'format'])
      let sourceBytes
      if (typeof this.image_original.image.arrayBuffer === 'undefined') {
        // for safari
        // https://stackoverflow.com/questions/14340989/blob-does-not-accept-uint8array-on-ios
        const buffer = new ArrayBuffer(this.image_original.image.size);
        sourceBytes = new Uint8Array(buffer);
        const f = atob(this.image_original.thumbnail.split(',')[1])
        for (let i = 0; i < this.image_original.image.size; i++) {
            sourceBytes[i] = f.charCodeAt(i);
        }
      } else {
        // for chrome
        sourceBytes = new Uint8Array(await this.image_original.image.arrayBuffer())
      }

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
