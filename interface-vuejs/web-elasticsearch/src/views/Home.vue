<template>
  <div class="home">
    <Elasticsearch/>
  </div>
</template>

<script>
// @ is an alias to /src
import Elasticsearch from '@/components/Elasticsearch.vue'

export default {
  name: 'Home',
  components: {
    Elasticsearch
  }
}
</script>
