<template>
  <div>
    <TypeNav />
    <div class="main">
      <div class="py-container">
        <!--bread-->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <ul class="fl sui-tag">
            <!-- 目录的面包屑 -->
            <li class="with-x" v-if="searchParams.categoryName">
              {{ searchParams.categoryName
              }}<i @click="removeCategoryName">×</i>
            </li>
            <!-- 关键字的面包屑 -->
            <li class="with-x" v-if="searchParams.keyword">
              {{ searchParams.keyword
              }}<i @click="removeKeyword">×</i>
            </li>
            <!-- 品牌的面包屑 -->
            <li class="with-x" v-if="searchParams.trademark">
              {{ searchParams.trademark.split(':')[1]
              }}<i @click="removeTrademark">×</i>
            </li>
            <!-- 售卖属性的面包屑 -->
            <li class="with-x" v-for="(prop, index) in searchParams.props" :key="index">
              {{ prop.split(':')[1]
              }}<i @click="removeProp(index)">×</i>
            </li>
          </ul>
        </div>

        <!--selector-->
        <SearchSelector @trademarkInfo="trademarkInfo" @attrInfo="attrInfo" />

        <!--details-->
        <div class="details clearfix">
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <ul class="sui-nav">
                <li :class="{active: isOne}" @click="changeOrder(1)">
                  <a>综合<span v-show="isOne&&isASC">⬆</span> <span v-show="isOne&&!isASC">⬇</span></a>
                </li>
                <li :class="{active: !isOne}" @click="changeOrder(2)">
                  <a>价格<span v-show="!isOne&&isASC">⬆</span> <span v-show="!isOne&&!isASC">⬇</span></a>
                </li>
              </ul>
            </div>
          </div>
          <div class="goods-list">
            <!-- 商品列表 -->
            <ul class="yui3-g">
              <li class="yui3-u-1-5" v-for="goods in goodsList" :key="goods.id">
                <div class="list-wrap">
                  <div class="p-img">
                    <router-link :to="{ path: '/detail/' + goods.id }">
                      <!-- <img v-lazy="goods.defaultImg"/> -->
                      <img v-lazy="goods.defaultImg">
                    </router-link>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥</em>
                      <i>{{ goods.price }}</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a
                      target="_blank"
                      href="item.html"
                      title="促销信息，下单即赠送三个月CIBN视频会员卡！【小米电视新品4A 58 火爆预约中】"
                      >{{ goods.title }}</a
                    >
                  </div>
                  <div class="commit">
                    <i class="command">已有<span>2000</span>人评价</i>
                  </div>
                  <div class="operate">
                    <a
                      href="success-cart.html"
                      target="_blank"
                      class="sui-btn btn-bordered btn-danger"
                      >加入购物车</a
                    >
                    <a href="javascript:void(0);" class="sui-btn btn-bordered"
                      >收藏</a
                    >
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <!-- 分页器 -->
          <MyPagination :pageNo="searchParams.pageNo" :pageSize="searchParams.pageSize" :total="total" :continus="5" @getPage="getPage"></MyPagination>
          {{total}}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SearchSelector from './SearchSelector/SearchSelector'
import { mapGetters, mapState } from 'vuex'
export default {
  name: 'MySearch',
  data() {
    return {
      // 带给服务器的参数
      searchParams: {
        // 一级分类Id
        category1Id: '',
        // 一级分类Id
        category2Id: '',
        // 一级分类Id
        category3Id: '',
        // 分类名字
        categoryName: '',
        // 关键字
        keyword: '',
        // 排序
        order: '1:desc',
        // 分页器用的，代表的是当前是第几页
        pageNo: 1,
        // 代表每一页展示数据个数
        pageSize: 10,
        // 平台售卖属性操作带的数据
        props: [],
        // 品牌
        trademark: '',
      },
    }
  },
  components: {
    SearchSelector,
  },
  beforeMount() {
    // 在组件挂载完毕之前执行一次
    // console.log(this.$route)
    Object.assign(this.searchParams, this.$route.query, this.$route.params)
  },
  mounted() {
    this.getSearchInfo()
  },
  computed: {
    ...mapGetters(['goodsList']),
    isOne(){
      // console.log(this.searchParams.order.indexOf('1') !== -1);
      return this.searchParams.order.indexOf('1') !== -1
    },
    isASC(){
      return this.searchParams.order.indexOf('asc') !== -1
    },
    // 获取总共有多少条数据
    ...mapState({
      total: state => state.search.searchInfo.total
    })
  },
  methods: {
    getSearchInfo() {
      this.$store.dispatch('getSearchInfo', this.searchParams)
      // 在请求数据回来之后把三个id都置空
      this.searchParams.category1Id = undefined
      this.searchParams.category2Id = undefined
      this.searchParams.category3Id = undefined
    },
    // 点击删除目录名字
    removeCategoryName() {
      this.searchParams.categoryName = undefined
      this.searchParams.category1Id = undefined
      this.searchParams.category2Id = undefined
      this.searchParams.category3Id = undefined
      this.getSearchInfo()
      // 地址栏也需要修改，进行路由跳转(路由自己跳转到自己这里)
      // 严谨：本意是删除query，如果路径当中出现params，不应该删除，路由跳转的时候应该带着
      if (this.$route.params) {
        this.$router.push({ name: 'search', params: this.$route.params })
      }
    },
    // 删除关键字
    removeKeyword(){
      this.searchParams.keyword = ''
      // 再次发数据
      this.getSearchInfo()
      // 通过全局事件总线发送到Header清空input输入框
      this.$bus.$emit('clearInput')
      // 进行路由跳转，自己跳到自己
      // if(this.$route.query){
        this.$router.push({name: 'search', query: this.$route.query})
      // }
    },
    // 自定义事件， 添加品牌
    trademarkInfo(trademark){
      // console.log(trademark);
      this.searchParams.trademark = `${trademark.tmId}:${trademark.tmName}`
      this.getSearchInfo()
    },
    // 删除品牌
    removeTrademark(){
      this.searchParams.trademark = ''
      this.getSearchInfo()
    },
    // 自定义事件，从子组件获取售卖属性
    attrInfo(attrs, attrValue) {
      // 先判断里面props里面有没有
      let str = `${attrs.attrId}:${attrValue}:${attrs.attrName}`
      if(this.searchParams.props.indexOf(str) === -1){
        this.searchParams.props.push(str)
      }
      this.getSearchInfo()
    },
    // 删除售卖属性
    removeProp(i){
      this.searchParams.props.splice(i, 1)
      this.getSearchInfo()
    },
    // 点击切换综合和价格
    changeOrder(flag){
      let originOrder = this.searchParams.order.split(':')
      let originFlag = originOrder[0]
      let originSort = originOrder[1]
      let newOrder = ''
      if(originFlag == flag){
        // 切换升序降序
        newOrder = `${originFlag}:${originSort == 'asc' ? 'desc' : 'asc'}`
      }else{
        // 切换flag
        newOrder = `${originFlag == '1' ? '2' : '1'}:${originSort}`
      }
      this.searchParams.order = newOrder
      this.getSearchInfo()
    },
    // 自定义事件，根据子组件传来的跳转页面的参数，重新获取数据
    getPage(page){
      this.searchParams.pageNo = page
      this.getSearchInfo()
    }
  },
  watch: {
    $route() {
      // 再次发起请求之前要先整理数据
      Object.assign(this.searchParams, this.$route.params, this.$route.query)
      // 发起ajsx请求
      this.getSearchInfo()
    },
  },
}
</script>

<style lang="less" scoped>
.main {
  margin: 10px 0;

  .py-container {
    width: 1200px;
    margin: 0 auto;

    .bread {
      margin-bottom: 5px;
      overflow: hidden;

      .sui-breadcrumb {
        padding: 3px 15px;
        margin: 0;
        font-weight: 400;
        border-radius: 3px;
        float: left;

        li {
          display: inline-block;
          line-height: 18px;

          a {
            color: #666;
            text-decoration: none;

            &:hover {
              color: #4cb9fc;
            }
          }
        }
      }

      .sui-tag {
        margin-top: -5px;
        list-style: none;
        font-size: 0;
        line-height: 0;
        padding: 5px 0 0;
        margin-bottom: 18px;
        float: left;

        .with-x {
          font-size: 12px;
          margin: 0 5px 5px 0;
          display: inline-block;
          overflow: hidden;
          color: #000;
          background: #f7f7f7;
          padding: 0 7px;
          height: 20px;
          line-height: 20px;
          border: 1px solid #dedede;
          white-space: nowrap;
          transition: color 400ms;
          cursor: pointer;

          i {
            margin-left: 10px;
            cursor: pointer;
            font: 400 14px tahoma;
            display: inline-block;
            height: 100%;
            vertical-align: middle;
          }

          &:hover {
            color: #28a3ef;
          }
        }
      }
    }

    .details {
      margin-bottom: 5px;

      .sui-navbar {
        overflow: visible;
        margin-bottom: 0;

        .filter {
          min-height: 40px;
          padding-right: 20px;
          background: #fbfbfb;
          border: 1px solid #e2e2e2;
          padding-left: 0;
          border-radius: 0;
          box-shadow: 0 1px 4px rgba(0, 0, 0, 0.065);

          .sui-nav {
            position: relative;
            left: 0;
            display: block;
            float: left;
            margin: 0 10px 0 0;

            li {
              float: left;
              line-height: 18px;

              a {
                display: block;
                cursor: pointer;
                padding: 11px 15px;
                color: #777;
                text-decoration: none;
              }

              &.active {
                a {
                  background: #e1251b;
                  color: #fff;
                }
              }
            }
          }
        }
      }

      .goods-list {
        margin: 20px 0;

        ul {
          display: flex;
          flex-wrap: wrap;

          li {
            height: 100%;
            width: 20%;
            margin-top: 10px;
            line-height: 28px;

            .list-wrap {
              .p-img {
                padding-left: 15px;
                width: 215px;
                height: 255px;

                a {
                  color: #666;

                  img {
                    max-width: 100%;
                    height: auto;
                    vertical-align: middle;
                  }
                }
              }

              .price {
                padding-left: 15px;
                font-size: 18px;
                color: #c81623;

                strong {
                  font-weight: 700;

                  i {
                    margin-left: -5px;
                  }
                }
              }

              .attr {
                padding-left: 15px;
                width: 85%;
                overflow: hidden;
                margin-bottom: 8px;
                min-height: 38px;
                cursor: pointer;
                line-height: 1.8;
                display: -webkit-box;
                -webkit-box-orient: vertical;
                -webkit-line-clamp: 2;

                a {
                  color: #333;
                  text-decoration: none;
                }
              }

              .commit {
                padding-left: 15px;
                height: 22px;
                font-size: 13px;
                color: #a7a7a7;

                span {
                  font-weight: 700;
                  color: #646fb0;
                }
              }

              .operate {
                padding: 12px 15px;

                .sui-btn {
                  display: inline-block;
                  padding: 2px 14px;
                  box-sizing: border-box;
                  margin-bottom: 0;
                  font-size: 12px;
                  line-height: 18px;
                  text-align: center;
                  vertical-align: middle;
                  cursor: pointer;
                  border-radius: 0;
                  background-color: transparent;
                  margin-right: 15px;
                }

                .btn-bordered {
                  min-width: 85px;
                  background-color: transparent;
                  border: 1px solid #8c8c8c;
                  color: #8c8c8c;

                  &:hover {
                    border: 1px solid #666;
                    color: #fff !important;
                    background-color: #666;
                    text-decoration: none;
                  }
                }

                .btn-danger {
                  border: 1px solid #e1251b;
                  color: #e1251b;

                  &:hover {
                    border: 1px solid #e1251b;
                    background-color: #e1251b;
                    color: white !important;
                    text-decoration: none;
                  }
                }
              }
            }
          }
        }
      }

      .page {
        width: 733px;
        height: 66px;
        overflow: hidden;
        float: right;

        .sui-pagination {
          margin: 18px 0;

          ul {
            margin-left: 0;
            margin-bottom: 0;
            vertical-align: middle;
            width: 490px;
            float: left;

            li {
              line-height: 18px;
              display: inline-block;

              a {
                position: relative;
                float: left;
                line-height: 18px;
                text-decoration: none;
                background-color: #fff;
                border: 1px solid #e0e9ee;
                margin-left: -1px;
                font-size: 14px;
                padding: 9px 18px;
                color: #333;
              }

              &.active {
                a {
                  background-color: #fff;
                  color: #e1251b;
                  border-color: #fff;
                  cursor: default;
                }
              }

              &.prev {
                a {
                  background-color: #fafafa;
                }
              }

              &.disabled {
                a {
                  color: #999;
                  cursor: default;
                }
              }

              &.dotted {
                span {
                  margin-left: -1px;
                  position: relative;
                  float: left;
                  line-height: 18px;
                  text-decoration: none;
                  background-color: #fff;
                  font-size: 14px;
                  border: 0;
                  padding: 9px 18px;
                  color: #333;
                }
              }

              &.next {
                a {
                  background-color: #fafafa;
                }
              }
            }
          }

          div {
            color: #333;
            font-size: 14px;
            float: right;
            width: 241px;
          }
        }
      }
    }
  }
}
</style>
