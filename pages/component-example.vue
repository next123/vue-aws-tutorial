<template>
  <div>

  </div>
</template>

<script>
  import axios from 'axios'
  // import fileUploader from 'components/FileUploader'
  // import linklist from '~/components/linklist'
  export default {
    // components: {
    //   fileUploader
    // },
    async created() {
      const config = {
        headers: {
          'Accept': 'application/json'
        }
      }
      try {
        const res = await axios.get('https://reqres.in/api/unknown/2', config)

        console.log(res)

      } catch (error) {
        console.log(error)
      }
    }
  }
</script>

<style>

</style>
