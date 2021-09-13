<template>
  <div class="container">
    <CabinetsList :cabinets="cabinets" />
  </div>
</template>

<script>
export default {
  name: 'Home',
  data: () => ({
    cabinets: []
  }),
  created () {
    for (let i = 1; i <= 573; i++) {
      this.cabinets.push({ id: i })
    }
  }
}
</script>
<style scoped>
.container {
  width: 100%;
  max-width: 900px;
  margin: 0 auto;
}
</style>
