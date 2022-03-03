<template>
  <div id="mainTemplate">
    <div class="mainContent">
      <div class="mainPicture">
        <div class="content content--fixed">
          <button id="openButton" class="changeButton" @click="openGrim">
            Open
          </button>
        </div>
        <!--完整图-->
        <div id="content1" class="content contentSwitch contentSwitchCurrent">
          <h2 class="contentTitle">Escape</h2>
          <div class="contentSubtitle">explore</div>
        </div>
        <div id="content2" class="content contentSwitch">
          <h2 class="contentTitle">Beach</h2>
          <div class="contentSubtitle">Lift</div>
        </div>
        <div id="content3" class="content contentSwitch">
          <h2 class="contentTitle">Desert</h2>
          <div class="contentSubtitle">Find</div>
        </div>
        <div id="content4" class="content contentSwitch">
          <h2 class="contentTitle">Ocean</h2>
          <div class="contentSubtitle">Uncover</div>
        </div>
        <div id="content5" class="content contentSwitch">
          <h2 class="contentTitle">Jungle</h2>
          <div class="contentSubtitle">Feel</div>
        </div>
        <div id="content6" class="content contentSwitch">
          <h2 class="contentTitle">Mountains</h2>
          <div class="contentSubtitle">Explore</div>
        </div>
        <!--缩略图-->
        <nav id="navGrim" class="grim">
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--1"></div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--2"></div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--3"></div>
            <div class="grimItemContent">
              <div class="grimItemInner">
                <button
                  id="closeButton"
                  class="backButton"
                  @click="closeFlag && closeGrim()"
                >
                  back
                </button>
              </div>
            </div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--4"></div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--5"></div>
            <a href="#content1" class="grimLink grimItemContent">
              <div class="grimItemInner">
                <h3
                  class="grimItemTitle"
                  @click="closeFlag && changeBackgroudAndClose(1)"
                >
                  Home
                </h3>
              </div>
            </a>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--6"></div>
            <div class="grimItemImg grimItemImg--1"></div>
            <a href="#content2" class="grimLink grimItemContent">
              <div class="grimItemInner">
                <h3
                  class="grimItemTitle"
                  @click="closeFlag && changeBackgroudAndClose(2)"
                >
                  Beach
                </h3>
                <span class="grimItemDescription">Lift your spirits</span>
              </div>
            </a>
            <div
              class="grimItemBackgroud grimItemBackgroudCover grimItemBackgroud--6"
            ></div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--7"></div>
            <div class="grimItemImg grimItemImg--2"></div>
            <a href="#content3" class="grimLink grimItemContent">
              <div class="grimItemInner">
                <h3
                  class="grimItemTitle"
                  @click="closeFlag && changeBackgroudAndClose(3)"
                >
                  Desert
                </h3>
                <span class="grimItemDescription">Find yourself</span>
              </div>
            </a>
            <div
              class="grimItemBackgroud grimItemBackgroudCover grimItemBackgroud--7"
            ></div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--8"></div>
            <div class="grimItemImg grimItemImg--3"></div>
            <a href="#content4" class="grimLink grimItemContent">
              <div class="grimItemInner">
                <h3
                  class="grimItemTitle"
                  @click="closeFlag && changeBackgroudAndClose(4)"
                >
                  Ocean
                </h3>
                <span class="grimItemDescription">Uncover beauty</span>
              </div>
            </a>
            <div
              class="grimItemBackgroud grimItemBackgroudCover grimItemBackgroud--8"
            ></div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--9"></div>
            <div class="grimItemImg grimItemImg--4"></div>
            <a href="#content5" class="grimLink grimItemContent">
              <div class="grimItemInner">
                <h3
                  class="grimItemTitle"
                  @click="closeFlag && changeBackgroudAndClose(5)"
                >
                  Jungle
                </h3>
                <span class="grimItemDescription">Feel nature</span>
              </div>
            </a>
            <div
              class="grimItemBackgroud grimItemBackgroudCover grimItemBackgroud--9"
            ></div>
          </div>
          <div class="grimItem">
            <div class="grimItemBackgroud grimItemBackgroud--10"></div>
            <div class="grimItemImg grimItemImg--5"></div>
            <a href="#content6" class="grimLink grimItemContent">
              <div class="grimItemInner">
                <h3
                  class="grimItemTitle"
                  @click="closeFlag && changeBackgroudAndClose(6)"
                >
                  Mountains
                </h3>
                <span class="grimItemDescription">Explore your limits</span>
              </div>
            </a>
            <div
              class="grimItemBackgroud grimItemBackgroudCover grimItemBackgroud--10"
            ></div>
          </div>
        </nav>
      </div>
      <div class="selfIntroduce"></div>
      <div class="projectIntroduce"></div>
      <div class="lastIntroduce"></div>
    </div>
  </div>
</template>

<script>
import anime from 'animejs/lib/anime.min.js'
export default {
  components: {},
  data() {
    return {
      animeEndFinish: false,
      closeFlag: true,
    }
  },
  mounted() {
    import('animejs/lib/anime.es.js').then((module) => {
      // use code
    })
    console.log(this.$site)
  },
  methods: {
    animeStart(arrGrimItems, i) {
      return new Promise((resolve, reject) => {
        let arrGrimItemComponent = {}
        arrGrimItemComponent.background =
          arrGrimItems[i].querySelector('.grimItemBackgroud')
        arrGrimItemComponent.background.style.transformOrigin =
          i % 2 === 0 ? '50% 100%' : '0% 50%'
        arrGrimItemComponent.inner = arrGrimItems[i].querySelector(
          '.grimItemContent > .grimItemInner'
        )
        arrGrimItemComponent.img = arrGrimItems[i].querySelector('.grimItemImg')
        arrGrimItemComponent.cover = arrGrimItems[i].querySelector(
          '.grimItemBackgroudCover'
        )
        arrGrimItemComponent.link = arrGrimItems[i].querySelector('a.grimLink')
        if (i % 2 === 0) {
          anime({
            targets: arrGrimItems[i],
            duration: 200 + i * 20,
            delay: 200 + i * 200,
            easing: 'easeInOutQuad',
            opacity: {
              value: 1,
              duration: 1,
            },
            height: [0, '100%'],
            top: ['100%', 0],
            complete: () => {
              if (arrGrimItemComponent.img && arrGrimItemComponent.cover) {
                arrGrimItemComponent.img.style.opacity = 1
                arrGrimItemComponent.cover.style.opacity = 1
                arrGrimItemComponent.cover.style.transformOrigin = '100% 50%'
                anime.remove(arrGrimItemComponent.cover)
                anime({
                  targets: arrGrimItemComponent.cover,
                  duration: 700,
                  easing: 'easeOutQuint',
                  scaleX: [1, 0],
                })
              }
              if (arrGrimItemComponent.inner) {
                anime.remove(arrGrimItemComponent.inner)
                anime({
                  targets: arrGrimItemComponent.inner,
                  duration: 1200,
                  delay: 150,
                  easing: 'easeOutQuint',
                  opacity: {
                    value: 1,
                    duration: 1,
                  },
                  translateY: ['100%', '0%'],
                })
              }
            },
          })
        } else if (i % 2 === 1) {
          anime({
            targets: arrGrimItems[i],
            duration: 200 + i * 20,
            delay: 200 + i * 200,
            easing: 'easeInOutQuad',
            left: {
              value: 0,
              duration: 1,
            },
            opacity: {
              value: 1,
              duration: 1,
            },
            width: [0, '100%'],
            complete: () => {
              if (arrGrimItemComponent.img && arrGrimItemComponent.cover) {
                arrGrimItemComponent.img.style.opacity = 1
                arrGrimItemComponent.cover.style.opacity = 1
                arrGrimItemComponent.cover.style.transformOrigin = '100% 50%'
                anime.remove(arrGrimItemComponent.cover)
                anime({
                  targets: arrGrimItemComponent.cover,
                  duration: 700,
                  easing: 'easeOutQuint',
                  scaleX: [1, 0],
                })
              }
              if (arrGrimItemComponent.inner) {
                anime.remove(arrGrimItemComponent.inner)
                anime({
                  targets: arrGrimItemComponent.inner,
                  duration: 1200,
                  delay: 150,
                  easing: 'easeOutQuint',
                  opacity: {
                    value: 1,
                    duration: 1,
                  },
                  translateY: ['100%', '0%'],
                })
              }
            },
          })
        }
        i++
        if (i < 10) resolve(i)
      }).then((res) => {
        this.animeStart(arrGrimItems, res)
      })
    },
    animeEnd(arrGrimItems, i) {
      return new Promise((resolve, reject) => {
        let arrGrimItemComponent = {}
        arrGrimItemComponent.background =
          arrGrimItems[i].querySelector('.grimItemBackgroud')
        arrGrimItemComponent.inner = arrGrimItems[i].querySelector(
          '.grimItemContent > .grimItemInner'
        )
        arrGrimItemComponent.img = arrGrimItems[i].querySelector('.grimItemImg')
        arrGrimItemComponent.cover = arrGrimItems[i].querySelector(
          '.grimItemBackgroudCover'
        )
        arrGrimItemComponent.link = arrGrimItems[i].querySelector('a.grimLink')
        if (i % 2 == 0) {
          anime({
            targets: arrGrimItems[i],
            duration: 200 + i * 20,
            delay: 200 + i * 200,
            easing: 'easeInOutQuad',
            opacity: {
              value: 0,
              duration: 200 + i * 20,
            },
            bottom: [0, '100%'],
          })
        } else if (i % 2 == 1) {
          anime({
            targets: arrGrimItems[i],
            duration: 200 + i * 20,
            delay: 200 + i * 200,
            easing: 'easeInOutQuad',
            opacity: {
              value: 0,
              duration: 200 + i * 20,
            },
            left: [0, '100%'],
          })
        }
        i++
        if (i < 9) resolve(i)
        else {
          anime({
            targets: arrGrimItems[i],
            duration: 200 + i * 20,
            delay: 200 + i * 200,
            easing: 'easeInOutQuad',
            opacity: {
              value: 0,
              duration: 200 + i * 20,
            },
            width: ['100%', 0],
          })
          resolve(i)
        }
      }).then((res) => {
        console.log(res)
        if (res < 9) this.animeEnd(arrGrimItems, res)
        else {
          this.animeEndFinish = true
        }
      })
    },

    openGrim() {
      console.log('openGrim')
      const navGrim = document.getElementById('navGrim')
      const navGrims = document.querySelectorAll('#navGrim>.grimItem')
      const button = document.getElementById('openButton')
      navGrim.classList.remove('grim')
      navGrim.classList.add('grimClicked')
      button.classList.add('changeButtonOff')
      let arrGrimItems = Array.from(navGrims)
      this.animeStart(arrGrimItems, 0)
    },
    closeGrim() {
      console.log('closeGrim')
      this.closeFlag = false
      const navGrim = document.getElementById('navGrim')
      const navGrims = document.querySelectorAll('#navGrim>.grimItem')
      const button = document.getElementById('openButton')
      let arrGrimItems = Array.from(navGrims)
      this.animeEnd(arrGrimItems, 0)
      setTimeout(() => {
        while (this.animeEndFinish == false) {}
        console.log('endstart')
        navGrim.classList.add('grim')
        navGrim.classList.remove('grimClicked')
        const closeButton = (document.getElementById(
          'closeButton'
        ).style.disable = false)
        button.classList.remove('changeButtonOff')
        this.closeFlag = true
      }, 2000)
    },
    changeBackgroudAndClose(index) {
      let currentActive = document
        .querySelector('.contentSwitchCurrent')
        .classList.remove('contentSwitchCurrent')
      let newElem = '#content' + index
      let active = document
        .querySelector(newElem)
        .classList.add('contentSwitchCurrent')
      this.closeGrim()
    },
  },
}
</script>

<style lang="stylus">
#mainTemplate {
  width: 100%;
  margin 0
  padding 0
  // 取消列表样式
  ol, ul {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  a {
    text-decoration: none;
    transition: color 0.3s;
  }
  .mainContent {
    .mainPicture {
      height: 100vh;
      position: relative;
      //button的布局，左下去除背景样式
      .changeButton {
        position absolute;
        z-index 11
        bottom 1vh
        left 1vh
        background none
        border: none;
        outline: none;
        cursor: pointer;
        color white
        font-size 20px
      }
      .changeButtonOff{
        display none;
      }
      //图片的背景
      #content1 {
        background-image: url(./img/bg.jpg);
      }

      #content2 {
        background-image: url(./img/bg1.jpg);
      }

      #content3 {
        background-image: url(./img/bg2.jpg);
      }

      #content4 {
        background-image: url(./img/bg3.jpg);
      }

      #content5 {
        background-image: url(./img/bg4.jpg);
      }

      #content6 {
        background-image: url(./img/bg5.jpg);
      }
      //默认的content均不显示
      .contentSwitch {
        display none
      }
      //当前所显示的content
      .contentSwitchCurrent{
        position: absolute
        z-index 10
        display flex;
        flex-direction column;
        justify-content center;
        align-items center
        background-size: cover;
        width: 100%;
        height: 100%;
        .contentTitle {
          font-size: 8vw;
          margin: 0;
          font-weight: normal;
        }

        .contentSubtitle {
          font-size: 1.45rem;
          padding: 1rem 0;
          letter-spacing: 2px;
          text-indent: 2px;
        }

        .contentSubtitle::after {
          content: '\2014';
          display: block;
          font-size: 2.75rem;
          text-align: center;
        }
      }

      //grim默认不显示
      .grim {
        display: none;
      }
      //当按钮按下后的grim显示
      .grimClicked {
        display: grid;
        position: absolute;
        z-index: 100;
        bottom: 0;
        left: 0;
        width: 100%;
        height: 100%;
        grid-template-columns: repeat(32,3.125%);
        grid-template-rows: repeat(32,3.125%);
        pointer-events: auto;
        //grim里的元素
        .grimItem {
          position: relative;
          top: 0;
          left: 0;
          display: flex;
          justify-content: center;
          align-items: center;
          opacity 0
        }
        //分配grim下的元素的area大小
        .grimItem:first-child {
          grid-area: 31 / 1 / 33 / 2;
        }

        .grimItem:nth-child(2) {
          grid-area: 31 / 2 / 33 / 3;
        }

        .grimItem:nth-child(3) {
          grid-area: 29 / 1 / 31 / 3;
        }

        .grimItem:nth-child(4) {
          grid-area: 29 / 3 / 33 / 5;
        }

        .grimItem:nth-child(5) {
          grid-area: 25 / 1 / 29 / 5;
        }

        .grimItem:nth-child(6) {
          grid-area: 25 / 5 / 33 / 9;
        }

        .grimItem:nth-child(7) {
          grid-area: 17 / 1 / 25 / 9;
        }

        .grimItem:nth-child(8) {
          grid-area: 17 / 9 / 33 / 17;
        }

        .grimItem:nth-child(9) {
          grid-area: 1 / 1 / 17 / 17;
        }

        .grimItem:nth-child(10) {
          grid-area: 1 / 17 / 33 / 33;
        }
        //默认的每个item的backgroud背景
        .grimItemBackgroud {
          display inline-block
          position: absolute;
          width: 100%;
          height: 100%;
          bottom: 0;
          left: 0;
          box-shadow: 0 0 0 2px currentColor;
          background: currentColor;
        }
        .grimItemBackgroud--1 { color: #f1d3b9; }
        .grimItemBackgroud--2 { color: #df9e98; }
        .grimItemBackgroud--3 { color: #fcfdff; }
        .grimItemBackgroud--4 { color: #9ed4d4; }
        .grimItemBackgroud--5 { color: #000; }

        .grimItemBackgroud--6 { color: #7296de; }
        .grimItemBackgroud--7 { color: #7c95b5; }
        .grimItemBackgroud--8 { color: #86bfbf; }
        .grimItemBackgroud--9 { color: #af9b9b; }
        .grimItemBackgroud--10 { color: #c57d76; }

        .grimItemImg {
          position: absolute;
          width: 100%;
          height: 100%;
          opacity: 1;
          background-repeat: no-repeat;
          background-size: 100% auto;
          background-position: 50% 100%;
          background-color: currentColor;
        }

        .grimItemImg--1 {
          color: #fcfdff;
          background-image: url(./img/1.jpg)}
        .grimItemImg--2 {
          color: #a9bdd6;
          background-image: url(./img/2.jpg)}
        .grimItemImg--3 {
          color: #9ed4d4;
          background-image: url(./img/3.jpg)}
        .grimItemImg--4 {
          color: #d0bebe;
          background-image: url(./img/4.jpg)}
        .grimItemImg--5 {
          color: #df9e98;
          background-image: url(./img/5.jpg)
        }

        .grimItemContent {
          position: relative;
          color: #000;
          overflow: hidden;
        }

        .grimItemContent:hover,
        .grimItemContent:focus {
          opacity: 0.8;
          color: inherit;
          transition: opacity 0.3s;
        }

        .grimItemInner {
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          text-align: center;
          position: relative;
        }
        //grim返回键
        .backButton {
          background none
          border: none;
          outline: none;
          cursor: pointer;
          font-size 20px
          color: #000;
        }

        .grimItemTitle {
          font-weight: normal;
          margin: 0;
          font-size: 4vmax;
        }

        .grimItem:nth-child(-n+6) .grimItemTitle {
          font-size: 2vmax;
        }

        .grimItem:nth-child(5) .grimItemTitle {
          color: #fff;
        }

        .grimItemDescription {
          font-size: 0.85rem;
          margin: 0.5rem 0 0 0;
        }

        .grimItemDescription::after {
          content: '\2014';
          position: relative;
          display: block;
          font-size: 1.75rem;
        }
      }
    }
    .selfIntroduce {
    }

    .projectIntroduce {
    }
  }
}
</style>
