<template>
  <div class="catalog__wrapper">
    <h1>Cabinets List</h1>
    <div class="catalog">
      <div v-for="item in visibleCabinets" :key="item.id" class="catalog__item">
        {{ item.id }}
      </div>
    </div>
    <PaginationBlock
      :pagination-items="cabinets"
      :pages-limit="limit"
      @get:actual-items="setVisibleCabinets($event)"
    />
  </div>
</template>

<script>
export default {
  name: 'CabinetsList',
  props: {
    cabinets: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
      limit: 10,
      visibleCabinets: []
    }
  },
  head () {
    return {
      title: this.getPageTitle(),
      meta: [
        {
          description: this.getPageTitle()
        }
      ]
    }
  },
  methods: {
    setVisibleCabinets (data) {
      this.visibleCabinets = data
    },
    getPageTitle () {
      if (this.$route.query.page) {
        return `Каталог товаров - страница ${this.$route.query.page}`
      }
      return 'Каталог товаров'
    }
  }
}
</script>
<style lang="scss" scoped>
.catalog__wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  .catalog {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 10px;
    width: 100%;
    place-items: center;
    @media (max-width: 768px) {
      grid-template-columns: 1fr;
    }
    .catalog__item {
      display: flex;
      justify-content: center;
      align-items: center;
      border: 2px solid #f2f2f2;
      height: 300px;
      width: 100%;
      max-width: 450px;
    }
  }
}
</style>
