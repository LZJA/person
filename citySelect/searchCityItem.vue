<template>
  <div class="chose-citys">
    <div class="city" @click="selectCity( item )">
      <input type="checkbox" name="type" :checked="item.select"/>
      <label class="choose" >{{ item.name }}</label>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {

    }
  },
  props: {
    item: {
      type: Object,
      default: function() {}
    }
  },
  computed: {

  },
  //   components: { vHead, Icon, Flexbox, FlexboxItem, search },
  mounted() {
  },
  methods: {
    selectCity(item) {
      this.$emit('city-item-click', item)
    }
  }
}
</script>

<style lang="scss" scoped>
@import "../style/mixin";
.chose-citys {
    .city {
      background-color: white;
      font-size: .32rem;
      color: #404042;
      letter-spacing: 0;
      line-height: .32rem;
      padding: .36rem .4rem;
        .tips {
          font-size: .32rem;
          color: #3F3F3F;
          letter-spacing: 0;
          text-align: justify;
          line-height: .32rem;
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
</style>

