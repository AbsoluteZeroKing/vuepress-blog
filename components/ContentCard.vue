<template>
  <div class="card" :class="size">
    <div class="cardPicture">
      <a :href="links">
        <img :src="picture" alt="picture" />
      </a>
    </div>
    <div class="cardContent">
      <div class="mainContent">
        <div class="cardContentHeader">
          <img src="./img/time.svg" alt="time" />
          &nbsp;发布于{{ resolvedDate }}&nbsp;&nbsp;
          <img src="./img/user.svg" alt="user" />
          {{ author }}
          <img v-if="hotty" src="./img/hot.svg" alt="hot" />
        </div>
        <div class="cardContentTitle">
          <a :href="links">{{ title }}</a>
        </div>
        <div v-if="size != 'small'" class="cardContentSummary">
          {{ summary }}
        </div>
        <div class="cardContentToolbar">
          <a :href="links">
            <img src="./img/24gf-ellipsis.svg" alt="ellipsis" />
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import dayjs from 'dayjs'
import dayjsPluginUTC from 'dayjs/plugin/utc'
dayjs.extend(dayjsPluginUTC)
export default {
  name: 'ContentCard',
  props: {
    size: {
      type: String,
      default: 'small',
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['small', 'medium', 'large'].indexOf(value) !== -1
      },
    },
    tags: {
      type: [Array, String],
    },
    author: {
      type: String,
      default: 'leesdog',
    },
    date: {
      type: String,
    },
    title: {
      type: String,
    },
    summary: {
      type: String,
    },
    hotty: {
      type: Boolean,
    },
    picture: {
      type: String,
      default:
        'https://leesdog.oss-cn-shanghai.aliyuncs.com/zhenbai1.webp?versionId=CAEQOxiBgIC16erI.BciIDM2ZjEyNWFlNzZiYjQ2YWM4NzMyYmM4Yjg3ZjhhMmU0',
    },
    links: {
      type: String,
    },
  },
  data() {
    return {}
  },
  computed: {
    resolvedDate() {
      return this.date.includes(':')
        ? dayjs.utc(this.date).format('YYYY-MM-DD HH:mm')
        : dayjs.utc(this.date).format('YYYY-MM-DD')
    },
  },
  methods: {},
}
</script>

<style lang="stylus">
.card {
  display: inline-block;
  border-radius: 10px;
  margin 20px 0 20px
  box-shadow: 0 1px 20px -6px rgba(0, 0, 0, 0.5);
  transition: box-shadow 0.3s ease;
}

.small {
  width: 364px;
  height: 140px;
  .cardPicture{
    float right
    width: 55%;
    height 100%
    overflow:hidden
    img {
      width: 100%
      cursor: pointer;
      transition: all 0.6s;
    }
    img:hover {
      transform: scale(1.4);
    }
  }
  .cardContent{
    float right
    width: 40%;
    height 100%
    display flex
    justify-content center
    align-items center
    .mainContent{
      width:90%
      height 90%
      display flex
      flex-direction column
      justify-content space-around
      .cardContentHeader{
        display flex
        justify-content center
        align-items center
        font-size 10px
      }
      .cardContentTitle{
        a{
          color #504e4e
          font-size 18px
          font-weight 900
        }
        a:hover{
          color $accentColor
          transition: all 0.3s;
        }
      }
      .cardContentSummary{
        font-size 15px
        max-height 21px
        overflow hidden
      }
    }
  }
}
.medium {
  width: 540px;
  height: 200px;
  .cardPicture{
    float right
    width: 55%;
    height 100%
    overflow:hidden
    img {
      width: 100%
      cursor: pointer;
      transition: all 0.6s;
    }
    img:hover {
      transform: scale(1.4);
    }
  }
  .cardContent{
    float right
    width: 40%;
    height 100%
    display flex
    justify-content center
    align-items center
    .mainContent{
      width:90%
      height 90%
      display flex
      flex-direction column
      justify-content space-between
      .cardContentHeader{
        display flex
        justify-content start
        align-items center
        font-size 8px
      }
      .cardContentTitle{
        a{
          color #504e4e
          font-size 18px
          font-weight 900
        }
        a:hover{
          color $accentColor
          transition: all 0.3s;
        }
      }
      .cardContentSummary{
        font-size 13px
        max-height 43px
        overflow hidden
      }
    }
  }
}
.large {
  width: 780px;
  height: 300px;
  .cardPicture{
    float right
    width: 58%;
    height 100%
    overflow:hidden
    img {
      width: 100%
      cursor: pointer;
      transition: all 0.6s;
    }
    img:hover {
      transform: scale(1.4);
    }
  }
  .cardContent{
    float right
    width: 40%;
    height 100%
    display flex
    justify-content center
    align-items center
    .mainContent{
      width:90%
      height 90%
      display flex
      flex-direction column
      justify-content space-between
      .cardContentHeader{
        display flex
        justify-content start
        align-items center
        font-size 12px
      }
      .cardContentTitle{
        a{
          color #504e4e
          font-size 20px
          font-weight 900
        }
        a:hover{
          color $accentColor
          transition: all 0.3s;
        }
      }
      .cardContentSummary{
        max-height 65px
        overflow hidden
        font-size 16px
      }
    }
  }
}
</style>
