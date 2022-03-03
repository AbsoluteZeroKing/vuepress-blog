<template>
  <!-- 文章列表展示 -->
  <div id="base-list-layout">
    <div class="ui-posts" itemscope itemtype="http://schema.org/Blog">
      <article
        v-for="page in pages"
        :key="page.key"
        itemprop="blogPost"
        itemscope
        itemtype="https://schema.org/BlogPosting"
      >
        <meta itemprop="mainEntityOfPage" :content="page.path" />
        <ContentCard
          :size="cardSize"
          :links="page.path"
          :title="page.title"
          :author="page.frontmatter.author"
          :summary="page.frontmatter.summary"
          :date="page.frontmatter.date"
          :tags="page.frontmatter.tags"
          :picture="page.frontmatter.picture"
          :hotty="page.frontmatter.hotty"
        >
        </ContentCard>
      </article>
    </div>

    <component
      :is="paginationComponent"
      v-if="$pagination.length > 1 && paginationComponent"
    ></component>
  </div>
</template>

<script>
/* global THEME_BLOG_PAGINATION_COMPONENT */

import Vue from 'vue'
import ContentCard from '@theme/components/ContentCard'
import {
  Pagination,
  SimplePagination,
} from '@vuepress/plugin-blog/lib/client/components'
export default {
  components: { ContentCard },
  data() {
    return {
      paginationComponent: null,
      cardSize: 'large',
    }
  },
  computed: {
    pages() {
      return this.$pagination.pages
    },
  },
  mounted() {
    let info = navigator.userAgent
    //通过正则表达式的test方法判断是否包含“Mobile”字符串
    let isPhone = /mobile/i.test(info)
    //如果包含“Mobile”（是手机设备）则返回true
    this.cardSize = isPhone ? 'small' : 'large'
  },

  created() {
    this.paginationComponent = this.getPaginationComponent()
  },
  methods: {
    getPaginationComponent() {
      const n = THEME_BLOG_PAGINATION_COMPONENT
      if (n === 'Pagination') {
        return Pagination
      }

      if (n === 'SimplePagination') {
        return SimplePagination
      }

      return Vue.component(n) || Pagination
    },
  },
}
</script>

<style lang="stylus">
#base-list-layout {
  padding-top: 160px;
  min-height: calc(100vh - 80px - 60px - 160px);
  max-width: $contentWidth;
  margin: 0 auto;
  a{
    text-decoration none
  }
}

.common-layout {
  .content-wrapper {
    padding-bottom: 80px;
  }
}

.ui-posts {
  padding-bottom: 25px;
  margin-bottom: 25px;
  border-bottom: 1px solid $borderColor;

  &:last-child {
    border-bottom: 0px;
    margin-bottom: 0px;
  }
}
</style>
