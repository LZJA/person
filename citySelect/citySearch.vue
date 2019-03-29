<template>
  <div class="company-chose-citys">
    <v-head is-fixed="true" :left-part="true" class="fied-head" v-if="headerShowStatus">
      <div slot="left-part" @click="cancel">
        <div class="backIcon"></div>
      </div>
      <div slot="center-part">选择城市</div>
      <div slot="right-part" class="determine-num">
        <span @click="determineCitys">确定<span v-show="!selectAll">({{ determineNum }})</span></span>
      </div>
    </v-head>
    <div class="chinese-select" v-if="headerShowStatus">
      <div class="select-charaset">
        <a class="city-charaset-lists" v-for="(item, i) in firstCityCharacter" :key="i" href="javascript:void(0)" @click="jump(item)" @touchstart.stop.prevent="jump(item)" >{{ item.toLocaleUpperCase() }}</a>
      </div>
    </div>
    <search
      :class="{'mt-18':headerShowStatus}"
      :results.sync="searchResults"
      v-model="city_id"
      :city-search="true"
      position="absolute"
      auto-scroll-to-top
      :placeholder="'请输入城市名（如杭州，hangzhou)'"
      @on-focus="onFocus"
      @on-cancel="onCancel"
      @on-change="getResult"
      @on-search-item-click="selectCity"
      ref="search"
    ></search>
    <div v-if="headerShowStatus">
      <div class="had-chose-citys">
        <p>已选择的城市：</p>
        <div class="chose-citys-lists">
          <div class="lists-items" v-for="(item, index) in select" :key="index" v-show="!selectAll && select && select.length !== 0">
            <span @click.stop="cancelCity(item)"><icon type="clear" class="cancelChose" ></icon></span>
            {{ item.name }}
          </div>
          <div class="lists-no-items" v-show="selectAll">
            您已选择全国范围内所有城市
          </div>
        </div>
      </div>
    </div>
    <div class="position-hot" v-if="headerShowStatus">
      <p class="position hot-city">热门城市</p>
      <div class="hot-list">
        <Flexbox :gutter="0" wrap="wrap">
          <flexbox-item :span="1/3" v-for="(item, i) in hotCity" :key="i"><div :class="selectAll ? 'noSelect' : item.select ? 'hot-city-lists-active' : 'hot-city-lists'" @click="selectCity(item)">{{ item.name }}</div></flexbox-item>
        </Flexbox>
      </div>
    </div>
    <div class="lists" v-if="headerShowStatus">
      <div class="city" @click="selectAllCitys">
        <div>不限<span class="tips">（全国范围内都可接受）</span><input type="checkbox" name="type" />
          <input type="checkbox" name="type" :checked="selectAll"/>
          <label class="choose" ></label>
        </div>
      </div>
      <div v-for="(item, index) in citylists" class="_jump" :id="item.lettler" :key="index">
        <div class="chartes">
          <p>{{ item.lettler.toLocaleUpperCase() }}</p>
        </div>
        <div :class="{'city': true, 'noSelectCitys': selectAll}" v-for="(item1, index1) in item.items" @click="selectCity( item1 )" :key="index1">
          <p v-show="selectAll">{{ item1.name }} </p>
          <div v-show="!selectAll">
            <input type="checkbox" name="type" :checked="item1.select"/>
            <label class="choose" >{{ item1.name }}</label>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { getCityLists } from 'src/service/getData';
import search from 'src/components/search'
import vHead from 'src/components/vHead'
import { Icon, Flexbox, FlexboxItem } from 'vux'
import { getStorage, setStorage } from 'src/config/tools';

export default {
  data() {
    return {
      headerShowStatus: true,
      city_id: null, // 搜索城市
      determineNum: 0,
      selectAll: false,
      firstCityCharacter: [],
      hotCity: [],
      cityArr: [],
      citylists: [],
      select: [],
      results: [],
      searchResults: []
    }
  },
  props: {
    selectedCity: {
      type: Array,
      default: function() {}
    }
  },
  computed: {

  },
  components: { vHead, Icon, Flexbox, FlexboxItem, search },
  mounted() {
    this.initAllCity()
  },
  methods: {
    initAllCity() {
      this.citylists = getStorage('companyCityChose')
      this.firstCityCharacter = getStorage('companyCityFirstCharaet')
      this.hotCity = getStorage('companyHotCity')
      if (!this.citylists || !this.firstCityCharacter || !this.hotCity) {
        getCityLists().then(res => {
          this.hotCity = this._handleChangeSelectData(res.result.hot_cities)
          this.cityArr = res.result.cities
          this._handleAddFirstChinese(this.cityArr)
          this._handleGetCityList()
          setStorage('companyHotCity', this.hotCity)
          setStorage('companyCityFirstCharaet', this.firstCityCharacter)
          setStorage('companyCityChose', this.citylists)
        })
      } else {
        //   传入已选的城市并比对选中
        if (this.selectedCity[0] === 0) {
          this.selectAll = !this.selectAll
        } else {
          this.determineNum = this.selectedCity.length
          this.selectedCity.forEach(item => {
            this.hotCity.forEach(item2 => {
              if (item === item2.id) {
                item2.select = !item2.select
                this.select.push(item2)
              }
            })
            this.citylists.forEach(item3 => {
              item3.items.forEach(item4 => {
                if (item === item4.id) {
                  item4.select = !item4.select
                  this.select.push(item4)
                }
              })
            })
          })
        }
      }
    },
    getResult(val) {
      console.log('on-change', val)
      this.searchResults = this.results.filter((item) => {
        return item.name.indexOf(val) !== -1
      })
    },
    onFocus() {
      console.log('on focus')
      if (this.headerShowStatus) {
        const arr = [];
        this.citylists.forEach((item) => {
          arr.push(...item.items)
        })
        this.results = this.hotCity.concat(arr);
        this.searchResults = this.results;
        // this.initSearchItemCheck()
      }
      this.headerShowStatus = false
    },
    onCancel() {
      console.log('on cancel')
      this.headerShowStatus = true

    //   this.initItemCheck()
    },
    cancel() {
      this.$emit('update:cityShow', false);
      this.$nextTick(() => {
        window.scrollTo(0, 0);
      })
    },
    selectCity(item) {
      if (!this.selectAll) {
        item.select = !item.select
        if (item.select) {
          this.determineNum++
          this.select.push(item)
        } else {
          for (let i = 0; i < this.select.length; i++) {
            if (this.select[i].name === item.name) {
              this.select.splice(i, 1)
            }
          }
          this.determineNum--
        }
      }
    },
    cancelCity(item) {
      for (let i = 0; i < this.select.length; i++) {
        if (this.select[i].name === item.name) {
          this.select.splice(i, 1)
        }
      }
      const len = this.hotCity.length
      for (let i = 0; i < len; i++) {
        if (this.hotCity[i].name === item.name) this.hotCity[i].select = !this.hotCity[i].select
      }
      const len2 = this.citylists.length
      for (let i = 0; i < len2; i++) {
        const len1 = this.citylists[i].items.length
        for (let j = 0; j < len1; j++) {
          if (this.citylists[i].items[j].name === item.name) {
            this.citylists[i].items[j].select = !this.citylists[i].items[j].select
          }
        }
      }
      this.determineNum--
    },
    selectAllCitys() {
      this.selectAll = !this.selectAll;
      if (this.selectAll) {
        this.select = [0];
      } else {
        this.select = [];
      }
    },
    selsectHotCity(item) {
      item.select = !item.select

    },
    determineCitys() {
      if (this.select[0] !== 0) {
        const city_ids = this.select.map((item) => {
          return item.id
        })
        const city_name = this.select.map((item) => {
          return item.name
        })
        this.$emit('on-change', city_ids, city_name);
      } else {
        this.$emit('on-change', [0], []);
      }
      this.$emit('update:cityShow', false);
      this.$nextTick(() => {
        window.scrollTo(0, 0);
      })
    },
    jump(item) {
      this.$vux.toast.text(item.toLocaleUpperCase(), 'middle')
      if (item === '热') {
        document.body.scrollTop = 0
        document.documentElement.scrollTop = 0
        window.pageYOffset = 0
        return
      }
      const jump = document.getElementById(`${item}`)
      const total = jump.offsetTop + -45

      document.body.scrollTop = total
      document.documentElement.scrollTop = total
      window.pageYOffset = total
    },
    _handleAddFirstChinese(arr) {
      let len3 = this.cityArr.length
      for (let i = 0; i < len3; i++) {
        this.hotCity.forEach(item => {
          if (this.cityArr[i].name === item.name) {
            this.cityArr.splice(i, 1)
            len3--
          }
        })
      }
      let single = []
      const newSet = new Set()
      arr.forEach((v, i) => {
        newSet.add(v.en_name[0])
      });
      single = Array.from(newSet).sort()
      single.unshift(...['热'])
      this.firstCityCharacter = single
    },
    _handleGetCityList() {
      let len3 = this.cityArr.length
      for (let i = 0; i < len3; i++) {
        this.hotCity.forEach(item => {
          if (this.cityArr[i].name === item.name) {
            this.cityArr.splice(i, 1)
            len3--
          }
        })
      }
      const cityArr = []
      for (var i in this.firstCityCharacter) {
        cityArr.push({ lettler: this.firstCityCharacter[i], items: [] })
      }
      const len1 = this.cityArr.length
      // 垃圾代码
      const len = cityArr.length;
      for (var j = 1; j < len; j++) {
        for (var e = 0; e < len1; e++) {
          if (cityArr[j].lettler === this.cityArr[e].en_name[0]) {
            cityArr[j].items.push({ name: this.cityArr[e].name, id: this.cityArr[e].id, select: false })
          }
        }
      }
      // end
      this.citylists = cityArr.splice(1, len)
    },
    _handleChangeSelectData(obj) {
      for (const item of obj) {
        Object.assign(item, { 'select': false });
      }
      return obj;
    }
  }
}
</script>
<style lang="scss">
@import "../style/mixin";
.company-chose-citys {
  .mt-18{
      margin-top:.9rem;
  }
  .header{
    background-color: #fff;
    color: #282a2a;
    .backIcon{
      @include wh(.5rem,.5rem);
      @include bg-image( '../images/ic_back_black');
    }
  }
}
</style>
<style lang="scss" scoped>
@import "../style/mixin";
.company-chose-citys {
  @include allcover;
  z-index: 100;
  background-color: #f8f8f8;
  @include wh(100%, 100%);
  .determine-num {
    font-size: .32rem;
    color: #0077FF;
    letter-spacing: 0;
    text-align: right;
    font-weight: bold;
  }
  .chinese-select {
    width: .58rem;
    height: 100%;
    background-color: white;
    position: fixed;
    right: 0;
    .select-charaset {
      margin-top: 2.14rem;
      .city-charaset-lists {
        display: block;
        text-align: center;
        padding-bottom: .14rem;
        color: #0077FF;
        font-size: .24rem;
        font-weight: bold;
        line-height: .24rem;
      }
    }
  }

  .had-chose-citys {
    padding: .3rem .64rem .4rem 0;
    background-color: white;
    > p {
      font-size: .24rem;
      color: #999999;
      letter-spacing: 0;
      line-height: .24rem;
      padding-left: .26rem;
    }
    .chose-citys-lists {
      display: flex;
      flex-wrap: wrap;
      height: auto;
      width: 100%;
      .lists-items {
        position: relative;
        width: 2rem;
        height: .8rem;
        background: #0077FF;
        border: 0 solid #EEEEEE;
        border-radius: .08rem;
        font-size: .28rem;
        color: #FFFFFF;
        letter-spacing: 0;
        text-align: center;
        line-height: .8rem;
        margin-top: .52rem;
        margin-left: .26rem;
        > span {
          display: inline-block;
          .cancelChose {
            position: absolute;
            cursor: pointer;
            right: -.22rem;
            top: -.18rem;
            color: rgba(40, 42, 42, 0.7);
            font-size: .36rem;
          }
        }
      }
      .lists-no-items {
        font-size: .32rem;
        letter-spacing: 0;
        text-align: center;
        margin-top: .52rem;
        margin-left: .26rem;
        font-weight: bold;
      }
    }
  }
  .position-hot {
    padding: .3rem .84rem .2rem .26rem;
    .position {
      font-size: .24rem;
      color: #999999;
      letter-spacing: 0;
      line-height: .24rem;
    }
    .close-position {
      padding-top: .3rem;
      font-size: .28rem;
      color: #D2D2D2;
      letter-spacing: 0;
      line-height: .28rem;
      > span {
        color: #0077FF
      }
    }
    .hot-city {
      padding-bottom: .1rem
    }
    .hot-city-lists {
      width: 2rem;
      height: .8rem;
      background-color: white;
      margin-top: .2rem;
      border: 0 solid #EEEEEE;
      border-radius: .08rem;
      text-align: center;
      line-height: .8rem;
      font-size: .28rem;
      color: #404042;
      letter-spacing: 0;
    }
    .hot-city-lists-active {
      width: 2rem;
      height: .8rem;
      background-color: white;
      margin-top: .2rem;
      border: 1px solid #B2D6FF;
      border-radius: .08rem;
      text-align: center;
      line-height: .8rem;
      font-size: .28rem;
      color: #0077FF;
      letter-spacing: 0;
    }
    .noSelect {
      width: 2rem;
      height: .8rem;
      background: #F0F0F0;
      margin-top: .2rem;
      border: 1 solid #B2D6FF;
      border-radius: .08rem;
      text-align: center;
      line-height: .8rem;
      font-size: .28rem;
      color: #D2D2D2;
      letter-spacing: 0;
    }
  }
  .lists {
    margin-top: .4rem;
    .city {
      background-color: white;
      font-size: .32rem;
      color: #404042;
      letter-spacing: 0;
      line-height: .32rem;
      padding: 0 .84rem 0 .26rem;
      > p{
          padding: .36rem 0;
          // background: #F0F0F0;
          // color: #D2D2D2;
        }
      > div{
        padding: .36rem 0;
        .tips {
          font-size: .32rem;
          color: #3F3F3F;
          letter-spacing: 0;
          text-align: justify;
          line-height: .32rem;
        }
      }
    }
    .noSelectCitys {
      background: #F0F0F0 !important;
      color: #D2D2D2 !important;
    }
    .chartes {
      font-size: .32rem;
      color: #404042;
      letter-spacing: 0;
      line-height: .32rem;
      padding: 0 .84rem 0 .26rem;
      > p{
        padding: .36rem 0;
        font-size: .28rem;
        color: #999999;
        letter-spacing: 0;
        line-height: .28rem;
      }
    }
  }
}

input[type="checkbox"] + label::before {
  content: "\a0"; /*不换行空格*/
  display: inline-block;
  vertical-align: middle;
  font-size: .4rem;
  @include bg-image( '../images/ic_unselected_three_four_small');
  width: 1.1em;
  height: 1.1em;
  position: relative;
  top: -.06rem;
  float: right;
}
input[type="checkbox"]:checked + label::before {
  font-size: .4rem;
  @include bg-image( '../images/ic_selected_three_four_small');
  width: 1.1em;
  height: 1.1em;

}
input[type="checkbox"] {
  position: absolute;
  clip: rect(0, 0, 0, 0);
}
.fied-head {
  position: fixed;
  top: 0;
  width: 100%;
  z-index: 1;
}

</style>

