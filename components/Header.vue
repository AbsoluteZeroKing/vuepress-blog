<template>
  <header class="header">
    <div class="headerTitle">
      <NavLink link="/" class="headerTitleLink">{{ $site.title }} </NavLink>
    </div>
    <div class="headerTags">
      <ul v-if="$site.themeConfig.nav" class="headerTagsNav">
        <li
          v-for="item in $site.themeConfig.nav"
          :key="item.text"
          class="headerTagsNavItem"
        >
          <NavLink :link="item.link">{{ item.text }}</NavLink>
        </li>
      </ul>
    </div>
    <div id="headerComponent">
      <SearchBox id="searchBox" />
      <Feed class="feed" />
    </div>
  </header>
</template>

<script>
import SearchBox from '@SearchBox'
import Feed from './Feed'

export default {
  components: { SearchBox, Feed },
}
</script>

<style lang="stylus">
@import '~@app/style/config';
.header {
  // 顶部布局
  display: -webkit-flex; /* Safari */
  display: flex;
  justify-content: space-between;
  top: 0;
  width: 100%;
  z-index: 12;
  position: fixed;
  height: 80px;
  background-color: rgba(255 255 255, 1);
  transition: all 1s cubic-bezier(0.25, 0.8, 0.25, 1);

  // 取消列表样式
  ol, ul {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  a {
    text-decoration: none;
    transition: color 0.3s;
    font-weight 500
  }

  &:hover {
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08), 0 6px 6px rgba(0, 0, 0, 0.1);
    color: $yellowColor;
  }

  // 顶部左侧标题
  .headerTitle {
    display: -webkit-flex; /* Safari */
    display: flex;
    justify-content: flex-start;
    height: 80px;
    line-height: 80px;
    margin: 0 0 0 1vw;

    .headerTitleLink {
      font-size: 30px;
    }
  }

  // 顶部中间标签
  .headerTags {
    .headerTagsNav {
      display: -webkit-flex; /* Safari */
      display: flex;
      justify-content: space-around;

      .headerTagsNavItem {
        margin: 0 0 0 1vw;
        line-height: 80px;
        height: 80px;
        font-size: 20px;
      }
    }
  }

  // 顶部右侧组件
  #headerComponent {
    display: -webkit-flex; /* Safari */
    display: flex;
    justify-content: center;
    height: 80px;
    line-height: 80px;
    margin-right: 1vw;

    #searchBox {
      input {
        border-radius: 17px;
        border: 1px solid #cecece;
      }

      .suggestions .suggestion a {
        color: $accentColor;
      }
    }

    .feed {
      margin-right: 0;
    }
  }
}

@media (max-width: $MQMobile) {
  .header {
    display: none;
  }
}
</style>
