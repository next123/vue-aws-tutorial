<template>
  <div>
      <h1>
      F12 to check the meta tag of the page
      </h1>
  </div>
</template>

<script>
export default {
        head() {
           return {
               title: 'metaTagExample',
                meta: [{ 
                    conntent: 'Random string here'
                }
                ]
            }

        }
}
</script>

<style>

</style>