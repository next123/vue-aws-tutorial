<template>
  <div>
      <nuxt-link :v-if="data" :v-for="a in data" to:`${a.link}`></nuxt-link>
  </div>
</template>

<script>
export default {
    prop:['data']
}
</script>

<style>

</style>