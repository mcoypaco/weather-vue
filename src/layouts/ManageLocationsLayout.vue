<template>
  <div class="flex flex-col min-h-screen dark:bg-slate-800 dark:text-white px-4 py-4">
    <div
      class="flex flex-row-reverse"
      @click="close"
    >
      <XMarkIcon class="w-8 h-8 cursor-pointer" />
    </div>
    <ManageLocationsForm />
  </div>
</template>
<script>
import ManageLocationsForm from '../components/ManageLocationsForm.vue'
import { XMarkIcon } from '@heroicons/vue/24/solid'
import { mapStores } from 'pinia'
import main from '../stores/main'

export default {
  components: { ManageLocationsForm, XMarkIcon },
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
      try {
        this.mainStore.$patch({ showLocationsForm: false })
      } catch (error) {
        console.error(error)
      }
    }
  }
}
</script>
