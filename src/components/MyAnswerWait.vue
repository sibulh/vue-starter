<template>
  <div>
    <template v-if="Data.length > 0">
      <mt-loadmore :top-method="loadTop" :bottom-method="loadBottom"
                   :bottom-all-loaded="allLoaded" :auto-fill="false" ref="loadmore">
        <div class="r_d_head proListData_content cf" v-for="item in Data" >
          <router-link :to="{name: 'ProblemDetail', query: { type: item.typeId, haId: item.haId }}">
            <div class="head_info font-size-28 cf">
              <div class="fl" style="position: relative;">
                <img class="cir-50" :src="item.questAvatar">
              </div>
              <div class="name fl">{{item.questNickName}}</div>
              <div class="fr ">
                <span class="font-color-6" :class="{ green: item.status=='待回答' }">{{item.status}}</span><span class="price m_l_50">{{item.payMoney}}</span>
              </div>
            </div>
            <div class="head-content p_t_20 font-size-30">{{item.cHaInfo}} {{item.questionTxt}}</div>
            <div class="other_info font-small p_t_b_30 cf">
              <span class="fl">{{item.timeText}}</span>
              <div class="fr" v-if="item.typeId == 0">
                旁听 {{item.listenTimes}} <span class="income">收入分成</span>
                <template v-if="item.answererIncome > 0"> ¥{{item.answererIncome}} </template>
                <template v-else>{{item.answererIncome}}</template>
              </div>
              <div class="fr" v-else>
                回答 {{item.answerTimes}}
              </div>
            </div>
          </router-link>
        </div>
      </mt-loadmore>
    </template>
    <template v-else>
      <DataNull :ErrMessage="ErrMessage"></DataNull>
    </template>

    <div class="down_sort font-size-24" >
      <div class="btn-sort-sel btn-sort-sel-1" v-show="show === 1">
        <div class="sort-time p_t_10" @click="getData(5)"><span>快问快答</span></div>
        <div class="sort-pv p_t_10" @click="getData(0)"><span>个人问题</span></div>
        <div class="sort-kaipan p_t_10" style="padding-bottom: .5rem;" @click="getData(2)"><span>帮答问题</span></div>
      </div>
      <div class="btn-sort" @click="showSort()">
        <img src="../assets/img/btn_bot_sel.png">
      </div>
    </div>

  </div>
</template>

<script>
  import { mapGetters } from 'vuex'
  import DataNull from '../components/DataNull'
  export default {
    name: 'MyAnswerWait',
    data () {
      return {
        message: '',
        Data: '',
        searchCondition: {PageIndex: 0, PageSize: 10},
        TypeId: 5,
        pageList: [],
        allLoaded: false,
        scrollMode: 'auto',
        ErrMessage: '暂时没有此类型的问题哦',
        show: 0
      }
    },
    computed: {
      ...mapGetters(['cityId'])
    },
    mounted () {
      // 初始化
      this.init()
    },
    methods: {
      init () {
        let vm = this
        vm.getData(5)
      },
      showSort () {
        this.show = this.show === 1 ? 0 : 1
      },
      getData (TypeId) {
        let vm = this
        vm.show = 0
        // 发起 dispatch 到 action
        vm.$store.dispatch('getMyAnswerWait', {
          cityId: this.cityId,
          'questionTypeId': TypeId,
          'PageIndex': this.searchCondition.PageIndex,
          'PageSize': this.searchCondition.PageSize
        }).then((result) => {
          if (result) {
            vm.Data = result
            // 是否还有下一页，加个方法判断，没有下一页要禁止上拉
            let haveMore = true
            if (result.length < 1 || result == null) {
              haveMore = false
            }
            this.isHaveMore(haveMore)
            this.$nextTick(function () {
              this.scrollMode = 'touch'
            })
          }
        })
      },
      loadTop () {
        // 下拉刷新
        this.getData()
        this.$refs.loadmore.onTopLoaded()
      },
      loadBottom () {
        // 上拉加载更多
        this.more()
        this.$refs.loadmore.onBottomLoaded()
      },
      more () {
        // 分页查询
        this.searchCondition.PageIndex = parseInt(this.searchCondition.PageIndex) + 1
        let vm = this
        // 发起 dispatch 到 action
        vm.$store.dispatch('getMyAnswerWait', {
          cityId: this.cityId,
          'questionTypeId': this.TypeId,
          'PageIndex': this.searchCondition.PageIndex,
          'PageSize': this.searchCondition.PageSize
        }).then((result) => {
          if (result) {
            let haveMore = true
            if (result.length < 1 || result == null) {
              haveMore = false
            }
            vm.isHaveMore(haveMore)
            vm.Data = vm.Data.concat(result)
          }
        })
      },
      isHaveMore (isHaveMore) {
        // 是否还有下一页，如果没有就禁止上拉刷新
        this.allLoaded = true
        if (isHaveMore) {
          this.allLoaded = false
        }
      }
    },
    components: {
      DataNull
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
