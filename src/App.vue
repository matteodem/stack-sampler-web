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

        <div v-if="sampleKeys.length === 0 && hasSamples" class="text-gray-400 text-ms">Loading...</div>

        <div style="max-height: 220px" class="overflow-auto">
          <div v-for="key in sampleKeys" class="text-xs cursor-pointer inline-block mr-2 lg:mr-0 lg:block">
            <div v-text="key" class="my-1 pl-2 leading-relaxed bg-gray-300"></div>
          </div>
        </div>
      </div>

      <div class="lg:w-9/12 float-left">
        <div v-for="n in sampleAmount" class="inline-block">
          <div :key="n"
               style="width: 200px; height: 80px"
               :class="{ 'text-gray-500 italic': !currentlyPlayingSamples[n - 1] }"
               @click="playSample(currentlyPlayingSamples[n -1])"
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
          <div v-if="sampleKeys.length > 0">
            <button class="mt-3 text-white font-bold p-3"
                    style="min-width: 120px"
                    :class="{'bg-red-400': isPlaying, 'bg-red-600': !isPlaying}"
                    @click="playRandomSamples">
              <span v-text="isPlaying ? 'Stop' : 'Randomize'"></span>
            </button>

            <button class="mt-3 text-white font-bold p-3 bg-red-700"
                    style="min-width: 120px"
                    v-if="currentlyPlayingSamples.length > 0 && !isPlaying"
                    @click="playSamples">
              Replay
            </button>

            <button class="mt-3 text-white font-bold p-3 bg-blue-500"
                    @click="exportSamples">
              <span v-text="isExporting ? 'Exporting...' : 'Export'"></span>
            </button>
          </div>
          <div v-else>
            <load-samples-modal :defaultSamples="Object.entries(defaultSampleMap)" @load="loadDefaultSamples"></load-samples-modal>
          </div>
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

  import LoadSamplesModal from './components/LoadSamplesModal'
  import defaultSamples from './samples/*/*.mp3'

  const defaultSampleMap = {
    epiano: {
      sampleAmount: 3,
      folder: 'epiano',
      label: 'E-Piano Chords',
    },
    synth_arpeggio: {
      sampleAmount: 4,
      folder: 'synth_arpeggio',
      label: 'Synth Arpeggio',
    },
  }

  // TODO: Add custom samples upload
  // TODO: Button component with rounded borders
  // TODO: Play Pause Components.. Storybook?

  export default {
    components: { LoadSamplesModal },
    data () {
      return {
        bundler: "Parcela",
        sampleKeys: [],
        currentlyPlayingSamples: [],
        currentSampleKey: '',
        defaultSampleMap,
        sampleAmount: 6,
        isPlaying: false,
        isExporting: false,
        hasSamples: false,
      };
    },
    methods: {
      loadDefaultSamples (sampleKey) {
        const currentSampleMap = defaultSampleMap[sampleKey]
        const { folder, sampleAmount } = currentSampleMap

        const sampleData = Object.entries(defaultSamples[folder])

        this.sampleAmount = sampleAmount

        const promises = sampleData.map(([key, samplePath]) => {
          return new Promise((resolve) => {
            samples.load(key, samplePath, key => {
              resolve(key)
            })
          })
        })

        Promise.all(promises)
          .then(resolved => {
            this.sampleKeys = resolved
            this.currentSampleKey = sampleKey
            this.hasSamples = true
          })
      },
      playRandomSamples () {
        if (!this.isPlaying) {
          this.currentlyPlayingSamples = sampleSize(this.sampleAmount)(this.sampleKeys)
        }

        this.playSamples()
      },
      playSamples () {
        this.currentlyPlayingSamples.forEach(
          (this.isPlaying ? samples.stop : samples.start)
        )

        this.isPlaying = !this.isPlaying
      },
      playSample (key) {
        if (key) samples.start(key)
      },
      exportSamples () {
        this.isExporting = true
        const folder = this.currentSampleKey

        const promises = this.currentlyPlayingSamples.map(key => (new Promise((resolve, reject) => {
          JSZipUtils.getBinaryContent(defaultSamples[folder][key], (err, binaryContent) => {
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
              defaultSamples[folder][key],
              binaryContent,
              { binary: true })
            )

            zip.generateAsync({ type: 'blob' }).then((blob) => {
              saveAs(blob, 'export.zip')
              this.isExporting = false
            })
          })
      },
    },
  }
</script>
