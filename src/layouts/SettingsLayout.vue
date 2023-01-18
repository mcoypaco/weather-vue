<template>
  <div class="flex flex-col min-h-screen dark:bg-slate-800 dark:text-white px-4 py-4">
    <div
      class="flex flex-row-reverse px-8"
      @click="close"
    >
      <XMarkIcon class="w-8 h-8 cursor-pointer" />
    </div>
    <div class="container mx-auto max-w-xl">
      <AppSettings />
    </div>
  </div>
</template>
<script>
import AppSettings from '../components/AppSettings.vue'
import { XMarkIcon } from '@heroicons/vue/24/solid'
import { mapStores } from 'pinia'
import main from '../stores/main'

export default {
  components: { AppSettings, XMarkIcon },
  data () {
    return {
      //
    }
  },
  computed: {
    ...mapStores(main)
  },
  methods: {
    /**
     *
     *
     */
    close () {
      this.mainStore.$patch({
        showSettings: false
      })
    }
  }
}
</script>
