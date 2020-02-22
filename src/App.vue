<template>
  <!-- TODO: Use css3 grid layout! -->
  <div class="container mx-auto p-3 lg:p-0">
    <!-- Title -->
    <h1 class="text-5xl text-center font-bold my-10">
      Stack Sampler
    </h1>

    <!-- Sampler -->
    <div class="max-w-4xl border-2 border-gray-400 p-5 mx-auto rounded clearfix">

      <div class="lg:w-3/12 pr-3 float-left mb-2 lg:mb-0">
        <h2 class="font-bold">Samples</h2>

        <div v-if="sampleKeys.length === 0" class="text-gray-400 text-ms">Loading...</div>

        <div style="max-height: 220px" class="overflow-auto">
          <div v-for="key in sampleKeys" class="text-xs cursor-pointer inline-block mr-2 lg:mr-0 lg:block">
            <div v-text="key" class="my-1 pl-2 leading-relaxed bg-gray-300"></div>
          </div>
        </div>
      </div>

      <div class="lg:w-9/12 float-left">
        <div v-for="n in 6" class="inline-block">
          <div :key="n"
               style="width: 200px; height: 80px"
               :class="{ 'text-gray-500 italic': !currentlyPlayingSamples[n -1] }"
               class="border-gray-600 border-2 p-3 mr-3 mb-3 rounded cursor-pointer">
            <div v-if="!currentlyPlayingSamples[n -1]">
              Sample <span v-text="n"></span>
            </div>
            <div v-else>
              <span v-text="currentlyPlayingSamples[n -1]"></span>
            </div>
          </div>
        </div>

        <div class="block mt-4">
          <button class="mt-3 text-white font-bold p-3"
                  style="min-width: 120px"
                  :class="{'bg-red-400': isPlaying, 'bg-red-600': !isPlaying}"
                  @click="playRandomSamples">
            <span v-text="isPlaying ? 'Stop' : 'Play Random'"></span>
          </button>

          <button class="mt-3 text-white font-bold p-3 bg-blue-500"
                  @click="exportSamples">
            <span v-text="isExporting ? 'Exporting...' : 'Export'"></span>
          </button>
        </div>
      </div>
    </div>

    <div class="mt-4 text-center text-gray-600 max-w-2xl">
      Heavily inspired by the
      <a class="text-black" href="https://www.youtube.com/watch?v=pcHY67wM9Cs">Tiramisu Sampler</a> by
      <a class="text-black" href="https://www.patreon.com/marshallmcgee">Marshall McGee</a>.
    </div>
  </div>
</template>

<script>
  import JSZip from 'jszip'
  import JSZipUtils from 'jszip-utils'
  import { sampleSize } from 'lodash/fp'
  import saveAs from 'jszip/vendor/FileSaver'
  import defaultSamples from './samples/*.wav'

  // TODO: Load Samples Button
  // TODO: Prepare default samples categories
  // TODO: Add custom samples upload
  export default {
    data () {
      return {
        bundler: "Parcela",
        sampleKeys: [],
        currentlyPlayingSamples: [],
        isPlaying: false,
        isExporting: false,
      };
    },
    mounted () {
      const sampleData = Object.entries(defaultSamples)

      const promises = sampleData.map(([key, samplePath]) => {
        return new Promise((resolve) => {
          console.log(key, samplePath)
          samples.load(key, samplePath, key => {
            resolve(key)
          })
        })
      })

      Promise.all(promises)
        .then(resolved => {
          this.sampleKeys = resolved
        })
    },
    methods: {
      playRandomSamples () {
        const { isPlaying } = this

        if (!isPlaying) {
          this.currentlyPlayingSamples = sampleSize(6)(this.sampleKeys)
        }

        this.currentlyPlayingSamples.forEach(
          (isPlaying ? samples.stop : samples.start)
        )

        this.isPlaying = !this.isPlaying
      },
      exportSamples () {
        this.isExporting = true

        const promises = this.currentlyPlayingSamples.map(key => (new Promise((resolve, reject) => {
          JSZipUtils.getBinaryContent(defaultSamples[key], (err, binaryContent) => {
            if (err) reject(err)

            resolve({
              key,
              binaryContent
            })
          })
        })))

        const zip = new JSZip()

        Promise.all(promises)
          .then(contents => {
            contents.forEach(({ key, binaryContent }) => zip.file(
              defaultSamples[key],
              binaryContent,
              { binary: true })
            )

            zip.generateAsync({ type: 'blob' }).then(function (blob) {
              saveAs(blob, 'export.zip')
              this.isExporting = false
            })
          })
      },
    },
  }
</script>
