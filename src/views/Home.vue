<template>
  <div class="home">
    <demo />
  </div>
</template>

<script>
// @ is an alias to /src
import Demo from '@/components/Demo.vue'

export default {
  name: 'Home',
  components: {
      Demo
  },
  
}
</script>
