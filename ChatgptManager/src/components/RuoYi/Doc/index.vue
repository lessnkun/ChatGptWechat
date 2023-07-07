<template>
  <div>
    <svg-icon icon-class="question" @click="goto" />
  </div>
</template>

<script>
export default {
  name: 'RuoYiDoc',
  data() {
    return {
      url: 'https://yaiwiki.likesrt.com/archives/no1.html'
    }
  },
  methods: {
    goto() {
      window.open(this.url)
    }
  }
}
</script>
