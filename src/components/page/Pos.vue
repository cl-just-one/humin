<template>
  <div class="pos">
    <el-row>
      <el-col :span='7' class="pos-order" id="order-list">
        <el-tabs v-model="activeName">
          <el-tab-pane label="点餐" name="goods">
             <!-- :key="Math.random()" 修改table数据更新 表格不重绘问题 -->
            <el-table :key="Math.random()" :data="tabData" border style="width:100%;">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="数量" width="50"></el-table-column>
              <el-table-column prop="price" label="价格" widt="70"></el-table-column>
              <el-table-column label="操作" fixed="right">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totalDiv">
              <small>数量：</small>{{totalCount}}
              <small>金额：</small>{{totalMoney}}元
            </div>
            <div class="order-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods">删除</el-button>
              <el-button type="warning" @click="checkout">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单" name="list">挂单</el-tab-pane>
          <el-tab-pane label="外卖" name="buy">外卖</el-tab-pane>
        </el-tabs>
      </el-col>
      <el-col :span="17">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div>
            <ul>
              <li v-for="goods in oftenGoods" :key="goods.id" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="c-price">￥{{goods.price}}元</span>
              </li>
            </ul>
          </div>
        </div>
        <div class="goods-type">
            <el-tabs>
                <el-tab-pane label="汉堡">
                  <ul class='cookList'>
                      <li v-for="goods in type0Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                          <span class="foodImg">
                            <img :src="goods.goodsImg" width="100%">
                          </span>
                          <span class="foodName">{{goods.goodsName}}</span>
                          <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                  </ul>
                </el-tab-pane>
                <el-tab-pane label="小食">
                  <ul class='cookList'>
                      <li v-for="goods in type1Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                          <span class="foodImg">
                            <img :src="goods.goodsImg" width="100%">
                          </span>
                          <span class="foodName">{{goods.goodsName}}</span>
                          <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                  </ul>
                </el-tab-pane>
                <el-tab-pane label="饮料">
                  <ul class='cookList'>
                      <li v-for="goods in type2Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                          <span class="foodImg">
                            <img :src="goods.goodsImg" width="100%">
                          </span>
                          <span class="foodName">{{goods.goodsName}}</span>
                          <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                  </ul>
                </el-tab-pane>
                <el-tab-pane label="套餐">
                  <ul class='cookList'>
                      <li v-for="goods in type3Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                          <span class="foodImg">
                            <img :src="goods.goodsImg" width="100%">
                          </span>
                          <span class="foodName">{{goods.goodsName}}</span>
                          <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                  </ul>
              </el-tab-pane>
            </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'pos',
  data () {
    return {
      activeName: 'goods',
      tabData: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalCount: 0,
      totalMoney: 0
    }
  },
  created: function () {
    // 查询常用商品数据
    axios.get('../../static/oftenGoods.json')
      .then(response => {
        this.oftenGoods = response.data
      }).catch(error => {
        console.log(error)
        alert('网络连接错误。')
      })
    // 查询分类商品数据
    axios.get('../../static/typeGoods.json')
      .then(response => {
        this.type0Goods = response.data[0]
        this.type1Goods = response.data[1]
        this.type2Goods = response.data[2]
        this.type3Goods = response.data[3]
      }).catch(error => {
        console.log(error)
        alert('网络连接错误。')
      })
  },
  mounted: function () {
    var orderHeight = document.body.clientHeight
    document.getElementById('order-list').style.height = orderHeight + 'px'
  },
  methods: {
    // 添加订单列表
    addOrderList (goods) {
      this.totalCount = 0
      this.totalMoney = 0
      var isHave = false
      // 判断商品是否存在列表中
      for (let i = 0; i < this.tabData.length; i++) {
        if (this.tabData[i].goodsId === goods.goodsId) {
          isHave = true
        }
      }
      if (isHave) {
        let arr = this.tabData.filter(o => o.goodsId === goods.goodsId)
        ++arr[0].count
        this.tabData = this.tabData
        // this.tabData.filter(o => o.goodsId === goods.goodsId)[0].count++
      } else {
        // let newGoods = goods
        // newGoods.count = 1
        let newGoods = Object.assign(goods, {count: 1})
        this.tabData.push(newGoods)
      }
      this.getAllMoney()
    },
    delSingleGoods (goods) {
      this.tabData = this.tabData.filter(o => o.goodsId !== goods.goodsId)
      this.getAllMoney()
    },
    // 汇总数量和金额
    getAllMoney () {
      this.totalCount = 0
      this.totalMoney = 0
      if (this.tabData) {
        this.tabData.forEach((element) => {
          this.totalCount += element.count
          this.totalMoney += (element.price * element.count)
        })
      }
    },
    // 删除所有商品
    delAllGoods () {
      this.tableData = []
      this.totalCount = 0
      this.totalMoney = 0
    },
    checkout () {
      if (this.totalCount) {
        this.tabData = []
        this.totalCount = 0
        this.totalMoney = 0
        this.$message({
          message: '结账成功，感谢你又为店里出了一份力!',
          type: 'success'
        })
      } else {
        this.$message.error('不能空结。老板了解你急切的心情！')
      }
    }
  }
}
</script>

<style scoped>
.pos-order {
  background-color: #F9FAFC;
  border-right: 1px solid grey;
}
.order-btn {
  margin-top: 10px;
}
.often-goods ul li {
  list-style: none;
  border: 1px solid #D3DCE6;
  background-color: #fff;
  margin: 5px;
  padding: 10px;
  display: inline-block;
  cursor: pointer;
}
.title {
  height: 20px;
  text-align: left;
  padding: 10px;
  background-color: #fff;
  border-bottom: 1px solid #E4E7ED;
}
.c-price {
  color: blue;
}
.goods-type {
  margin-left: 10px;
}
.cookList li {
   list-style: none;
   width: 23%;
   border: 1px solid #E5E9F2;
   height: auto;
   overflow: hidden;
   background-color: #fff;
   padding: 2px;
   float: left;
   margin: 2px;
   cursor: pointer;
 }
 .cookList li span {
    font-size: 16px;
    display: block;
    float: left;
 }
 .foodImg {
   width: 40%;
 }
 .foodName {
   font-size: 18px;
   padding-left: 10px;
   color: brown;
 }
 .foodPrice {
   font-size: 16px;
   padding-left: 10px;
   padding-top: 10px;
 }
 .totalDiv {
   border-bottom: 1px solid grey;
 }
</style>
