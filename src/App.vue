<template>
  <div id="app">
    <v-header class="header" :seller="seller">
      i an header
    </v-header>
    <div class="tab border-1px">
      <div class="tab-item">
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <div class="content">
      <router-view :seller="seller"></router-view>
    </div>
  </div>
</template>

<script>

import Header from '@/components/header/header'

const ERR_OK = 0;

export default {
  name: 'app',
  components: {
    'v-header': Header
  },
  data() {
    return {
      seller: {}
    }
  },
  created() {
    this.$http.get('/api/seller').then(res =>{
      let msg = res.body;
      if(msg.errno === ERR_OK){
        this.seller = msg.data
      }
    })
  }
}
</script>

<style lang="stylus">
@import "./conmmon/stylus/mixin";

#app
  .tab
    display: flex
    width: 100%
    height: 40px
    line-height: 40px
    border-1px(rgba(7,17,27,0.1))
    .tab-item
      flex: 1
      text-align: center
      & > a
        display: block
        font-size: 14px
        color: rgb(77,85,93)
        &.active
          color: rgb(240,20,20)

</style>
