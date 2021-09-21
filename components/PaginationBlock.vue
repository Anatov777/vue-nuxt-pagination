<template>
  <div class="pagination__wrapper">
    <button
      class="show-more"
      :disabled="activePage === totalPages"
      @click="showMore"
    >
      Показать еще
    </button>
    <div class="pagination">
      <div
        v-if="activePage !== 1"
        class="pagination-arrow"
        @click="setActivePage(activePage - 1)"
      >
        <div class="arrow-left" />
      </div>
      <div class="page__wrapper">
        <nuxt-link
          v-for="pageNumber in visiblePages"
          :key="pageNumber"
          :to="getLinkRoute(pageNumber)"
          class="page"
          :class="{
            'active-page': pageNumber === activePage,
            'page-mr-20': hasMarginRight(pageNumber),
            'page-ml-20': hasMarginLeft(pageNumber),
            'mobile-page-ml-20': hasMobileMarginLeft(pageNumber),
            'mobile-hidden-link': isHidden(pageNumber)
          }"
          @click.native="setActivePage(pageNumber)"
        >
          {{ pageNumber }}
        </nuxt-link>
      </div>
      <div
        v-if="activePage !== totalPages"
        class="pagination-arrow"
        @click="setActivePage(activePage + 1)"
      >
        <div class="arrow-right" />
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'PaginationBlock',
  props: {
    paginationItems: {
      type: Array,
      default: () => []
    },
    pagesLimit: {
      type: Number,
      default: 10
    }
  },
  data () {
    return {
      activePage: +this.$route.query.page || 1,
      moreCabinets: []
    }
  },
  computed: {
    totalPages () {
      return Math.ceil(this.paginationItems.length / this.pagesLimit)
    },
    visibleItems () {
      const lastElement = this.activePage * this.pagesLimit
      return [...this.moreCabinets, ...this.paginationItems.slice(lastElement - this.pagesLimit, lastElement)]
    },
    middlePageNumber () {
      if (this.totalPages < 40) {
        return Math.ceil((this.totalPages - 5) / 2) + 5
      } else {
        return Math.ceil((this.totalPages - 5) / 2)
      }
    },
    visiblePages () {
      const pages = []

      if (this.totalPages < 8) {
        for (let i = 1; i <= this.totalPages; i++) {
          pages.push(i)
        }
      } else if (this.activePage < 5) {
        for (let i = 1; i <= 5; i++) {
          pages.push(i)
        }
        pages.push(this.middlePageNumber)
        pages.push(this.totalPages)
      } else if (this.activePage >= this.totalPages - 4) {
        pages.push(1)
        if (this.totalPages >= 40) {
          pages.push(this.middlePageNumber)
        }

        if (this.activePage === this.totalPages - 4) {
          pages.push(this.activePage - 2)
          pages.push(this.activePage - 1)
        }
        for (let i = this.totalPages - 4; i <= this.totalPages; i++) {
          pages.push(i)
        }
      } else {
        pages.push(1)
        if (this.activePage === 5) {
          pages.push(2)
        }
        if (this.activePage < this.middlePageNumber + 2) {
          pages.push(...this.getActualPages())
        }
        if (!(this.middlePageNumber >= (this.activePage - 2) && this.middlePageNumber <= (this.activePage + 2))) {
          pages.push(this.middlePageNumber)
        }
        if (this.activePage >= this.middlePageNumber + 2) {
          pages.push(...this.getActualPages())
        }

        pages.push(this.totalPages)
      }
      return pages
    }
  },
  watch: {
    visibleItems () {
      this.$emit('get:actual-items', this.visibleItems)
    }
  },
  created () {
    this.$emit('get:actual-items', this.visibleItems)
  },
  methods: {
    setActivePage (page) {
      this.moreCabinets = []
      this.activePage = page
      this.$router.push(this.getLinkRoute(page))
    },
    getActualPages () {
      const pages = []
      for (let i = this.activePage - 2; i <= this.activePage + 2; i++) {
        pages.push(i)
      }
      return pages
    },
    getLinkRoute (page) {
      if (page !== 1) {
        return `/catalog?page=${page}`
      } else {
        return '/catalog'
      }
    },
    getPageTitle () {
      if (this.$route.query.page) {
        return `Каталог товаров - страница ${this.$route.query.page}`
      }
      return 'Каталог товаров'
    },
    showMore () {
      this.moreCabinets = [...this.visibleItems]
      this.activePage++
      this.$router.push(`/catalog?page=${this.activePage}`)
    },
    hasMarginRight (pageNumber) {
      return this.totalPages >= 8 && ((pageNumber === 1 && this.activePage > 5) ||
              (pageNumber === this.middlePageNumber && this.activePage > this.middlePageNumber + 2 &&
              this.middlePageNumber < this.totalPages - 4))
    },
    hasMarginLeft (pageNumber) {
      return this.totalPages >= 8 && ((pageNumber === this.totalPages && this.activePage <= this.totalPages - 5) ||
              (pageNumber === this.middlePageNumber && this.activePage < this.middlePageNumber - 2))
    },
    hasMobileMarginLeft (pageNumber) {
      return pageNumber === this.totalPages &&
              this.activePage >= this.totalPages - 5 &&
              this.activePage < this.totalPages - 2
    },
    isHidden (pageNumber) {
      return (this.isLeftActualPageNumber(pageNumber)) ||
              (this.isRightActualPageNumber(pageNumber)) ||
              (this.isMiddlePageNumber(pageNumber)) ||
              (this.isFirstPageNumber(pageNumber)) ||
              (this.isPageNumberMoreThanThree(pageNumber)) ||
              (this.isPageNumberBeforeLastThree(pageNumber)) ||
              (this.isPageNumberNotActual(pageNumber))
    },
    isLeftActualPageNumber (pageNumber) {
      return pageNumber === this.activePage - 2 && this.activePage !== this.totalPages
    },
    isRightActualPageNumber (pageNumber) {
      return pageNumber === this.activePage + 2 && pageNumber !== this.totalPages && this.activePage !== 1
    },
    isMiddlePageNumber (pageNumber) {
      return pageNumber === this.middlePageNumber && this.activePage - 1 !== this.middlePageNumber &&
              this.activePage + 1 !== this.middlePageNumber && this.activePage !== this.middlePageNumber &&
              this.totalPages > 5
    },
    isFirstPageNumber (pageNumber) {
      return pageNumber === 1 && this.activePage > 2 && this.activePage !== this.totalPages
    },
    isPageNumberMoreThanThree (pageNumber) {
      return this.activePage < 3 && pageNumber > 3 && pageNumber !== this.totalPages
    },
    isPageNumberBeforeLastThree (pageNumber) {
      return this.activePage === this.totalPages && pageNumber < this.totalPages - 2 && pageNumber !== 1
    },
    isPageNumberNotActual (pageNumber) {
      return (pageNumber < this.activePage - 1 || pageNumber > this.activePage + 1) &&
              pageNumber !== this.totalPages && this.activePage !== this.totalPages && this.activePage !== 1
    }
  }
}
</script>
<style lang="scss" scoped>
.pagination__wrapper {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.page__wrapper {
  display: flex;
  gap: 10px;
  margin-top: 15px;
  .page {
    background-color: #f0f4f9;
    cursor: pointer;
    border-radius: 10px;
    width: 40px;
    height: 40px;
  }
  .active-page {
    background-color: #ff002c;
    color: #fff;
  }
}
a {
  text-decoration: none;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #000;
}
.page-mr-20 {
  margin-right: 20px;
}
.page-ml-20 {
  margin-left: 20px;
}
.mobile-page-ml-20 {
  @media (max-width: 600px) {
    margin-left: 20px;
  }
}
.show-more {
  margin-top: 20px;
  cursor: pointer;
  background-color: #f0f4f9;
  border-radius: 5px;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  border: none;
  @media (max-width: 768px) {
    max-width: 450px;
  }
}
.pagination {
  display: flex;
  justify-content: center;
  align-items: flex-end;
  gap: 20px;
}
.pagination-arrow {
  width: 40px;
  height: 40px;
  background-color: #f2f2f2;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  border-radius: 10px;
}
.arrow-left,
.arrow-right {
  width: 1px;
  height: 1px;
  border-top: 1px solid #000;
  border-right: 1px solid #000;
  transform: rotate(-135deg);
  padding: 5px;
}
.arrow-left {
  margin-left: 5px;
}
.arrow-right {
  transform: rotate(45deg);
  margin-right: 5px;
}
.mobile-hidden-link {
  @media (max-width: 600px) {
    display: none;
  }
}
</style>
