<template>
  <div>
    <!-- See https://www.tailwindtoolbox.com/components/modal -->
    <button class="mt-3 text-white font-bold p-3 bg-blue-500" @click="modalActive = !modalActive">
      Load Samples
    </button>

    <div>
      <div :class="{ 'opacity-0 pointer-events-none': !modalActive }" class="modal fixed w-full h-full top-0 left-0 flex items-center justify-center">
        <div class="modal-overlay absolute w-full h-full bg-gray-900 opacity-50" @click="closeModal"></div>

        <div class="modal-container bg-white w-11/12 md:max-w-md mx-auto rounded shadow-lg z-50 overflow-y-auto">

          <div @click="closeModal" class="modal-close absolute top-0 right-0 cursor-pointer flex flex-col items-center mt-4 mr-4 text-white text-sm z-50">
            <svg class="fill-current text-white" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 18 18">
              <path d="M14.53 4.53l-1.06-1.06L9 7.94 4.53 3.47 3.47 4.53 7.94 9l-4.47 4.47 1.06 1.06L9 10.06l4.47 4.47 1.06-1.06L10.06 9z"></path>
            </svg>
            <span class="text-sm">(Esc)</span>
          </div>

          <!-- Add margin if you want to see some of the overlay behind the modal-->
          <div class="modal-content py-4 text-left px-6">
            <!--Title-->
            <div class="flex justify-between items-center pb-3">
              <p class="text-2xl font-bold">Load Samples</p>
              <div @click="closeModal" class="modal-close cursor-pointer z-50">
                <svg class="fill-current text-black" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 18 18">
                  <path d="M14.53 4.53l-1.06-1.06L9 7.94 4.53 3.47 3.47 4.53 7.94 9l-4.47 4.47 1.06 1.06L9 10.06l4.47 4.47 1.06-1.06L10.06 9z"></path>
                </svg>
              </div>
            </div>

            <!--Body-->
            <div>
              Here are a few packs of samples for you to play around with.

              <div class="mt-4">
                <div v-for="[key, sample] in defaultSamples" class="inline-block pr-3">
                  <button class="bg-red-500 text-white p-3" v-text="sample.label" @click="loadSample(key)"></button>
                </div>
              </div>
            </div>

            <!--Footer-->
            <div class="flex justify-end pt-2">
              <button @click="closeModal" class="modal-close bg-indigo-500 p-3 text-white hover:bg-indigo-400">Close</button>
            </div>

          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    props: {
      defaultSamples: {
        type: Array,
      },
    },
    data () {
      return {
        modalActive: false,
      }
    },
    watch: {
      modalActive () {
        document.body.classList.toggle('modal-active')
      },
    },
    methods: {
      loadSample (key) {
        this.$emit('load', key)
        this.closeModal()
      },
      closeModal () {
        this.modalActive = false
      },
    },
  }
</script>
<style>
  .modal {
    transition: opacity 0.25s ease;
  }

  body.modal-active {
    overflow-x: hidden;
    overflow-y: visible !important;
  }
</style>
