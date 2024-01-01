<template>
  <div id="app">
    <h2 style="text-align: center;">Trade Computed</h2>
    <div style="margin-left: 40%;">
      <p>选择产品(每次前开单请先刷新，以防数据错误)</p>
      <select style="width: 100px;" @change="changeproduct($event)">
        <option v-for="(item,index) in product" :key="index" :value='item.id'>{{item.title}}</option>
      </select>
      <br/>
      <p>{{op}}：(0.01手)保证金：{{bzj}}</p>
      <h3>您的订单</h3>
      <span>账户总金额:<input style="margin-left: 10px;" v-model="account"></span><br/><br/>
      <span>开单价:<input style="margin-left: 10px;" v-model="currentPrice"></span><br/><br/>   
      <span>StopProfit:<input style="margin-left: 10px;" v-model="StopProfit"></span><br/><br/>
      <span>StopLoss:<input style="margin-left: 10px;" v-model="StopLoss"></span><br/><br/>
      <span>最大止损金额(usd):<input style="margin-left: 10px;" v-model="maxLoss"></span><br/>
      <button @click="compute" style="width: 60px; height: 60px;">计算</button><br/>
      <span>开单手数:(大于0自定义手数,小于0系统提供手数)<input style="margin-left: 10px;" v-model="prhand"></span><br/>
      <span>止损将损失(usd):<span style="font-size: 30px; margin-left: 10px;color: red;">{{accountloss}}</span></span><br/>
      <span>止盈将获得(usd):<span style="font-size: 30px; margin-left:  10px; color:green;">{{accountprofit}}</span></span><br/>
      <span>需要的保证金(usd):<span style="font-size: 30px; margin-left:  10px; color:green;">{{accountbzj}}</span></span><br/>
      <span v-show="isyinkui">盈亏比:<span style="font-size: 30px; margin-left:  10px; color:green;">{{yinkui}}</span></span><br/>
      <span  v-show="!isyinkui">盈亏比:<span style="font-size: 30px; margin-left:  10px; color:red;">{{yinkui}}</span></span><br/>
      <span v-show="isbzj">最小预付款比率(%):<span style="font-size: 30px; margin-left:  10px; color:green;">{{bzjlv}}%</span></span><br/>
      <span v-show="!isbzj">最小预付款比率(%):<span style="font-size: 30px; margin-left:  10px; color:red;">{{bzjlv}}%</span></span><br/>
      <span v-show="isheshi">建议:<span style="font-size: 30px; margin-left:  10px; color:red;">这个单子不合适，不建议下单</span></span><br/>
      <span v-show="!isheshi">建议:<span style="font-size: 30px; margin-left:  10px; color:green;">盈亏比合理，祝你好运</span></span><br/>
      <p>目前手数:{{prhand}}</p>
    </div>
  </div>
</template>

<script>


export default {
  name: 'App',
  data(){
    return{
      isyinkui:false,
      isbzj:false,
      basehande:0.01,
      account:200,
      isheshi:true,
      currentPrice:0.0,
      productId:0,
      maxLoss:0.0,
      StopProfit:0.0,
      StopLoss:0.0,
      accountprofit:0.0,
      accountloss:0.0,
      op:'EURUSD',
      profit:0.001,
      prhand:-1,
      bzj:0.53,
      bzjlv:0,
      yinkui:0,
      accountbzj:0,
      // 修改商品 0.01手盈利1u的浮动价格((0.01手)浮动的价差/(0.01手)浮动的价格)
      product:[{id:1,title:"XAUUSD",bzj:0.96,profit:1},{id:2,title:"USOIL",bzj:4.5,profit:0.1},{id:3,title:"DXYM",bzj:0.53,profit:0.1},{id:4,title:"AUDCAD",bzj:0.32,profit:0.00131783},{id:5,title:"EURJPY",bzj:0.52,profit:0.14285714},{id:6,title:"EURGBP",bzj:0.53,profit:0.00086},{id:7,title:"GBPJPY",bzj:0.62,profit:0.14703557},{id:8,title:"USDCHF",bzj:0.5,profit:0.00086},{id:9,title:"BTCUSD",bzj:0.93,profit:97.64},{id:10,title:"NZDJPY",bzj:0.3,profit:0.14118},{id:11,title:"AUDUSD",bzj:0.33,profit:0.001},{id:12,title:"AUDJPY",bzj:0.33,profit:0.14697674},{id:13,title:"GBPCAD",bzj:0.64,profit:0.0013},{id:14,title:"EURUSD",bzj:0.56,profit:0.001},{id:15,title:"EURAUD",bzj:0.56,profit:0.00148}]
    }
  },
  methods:{
    changeproduct(e){
      this.productId=e.target.value-1
      this.op =this.product[this.productId].title
      this.bzj = this.product[this.productId].bzj
    },
    compute(){
      this.isbzj=false
      this.yinkui=false
      this.isheshi=true
      if(this.prhand<0){
      this.prhand =((this.maxLoss/(Math.abs(this.currentPrice-this.StopLoss).toFixed(5)/this.product[this.productId].profit))*this.basehande).toFixed(2)
      }
      console.log("最大损失金额:",this.maxLoss)
      console.log("波动点差:",Math.abs(this.currentPrice-this.StopLoss).toFixed(5))
      console.log("0.01手波动金额:",(Math.abs(this.currentPrice-this.StopLoss)/this.product[this.productId].profit).toFixed(2))
      this.accountloss = (((Math.abs(this.currentPrice-this.StopLoss)/this.product[this.productId].profit)*((this.prhand/this.basehande)))).toFixed(2)
      this.accountprofit = (((Math.abs(this.currentPrice-this.StopProfit)/this.product[this.productId].profit)*((this.prhand/this.basehande)))).toFixed(2)
      this.yinkui = (this.accountprofit/this.accountloss).toFixed(2)
      this.bzjlv = (((this.account - this.accountloss)/(this.product[this.productId].bzj*(this.prhand/this.basehande)))*100).toFixed(2)
      this.accountbzj = (this.product[this.productId].bzj*(this.prhand/this.basehande)).toFixed(2)
      console.log("保证金：",(this.product[this.productId].bzj*(this.prhand/this.basehande)).toFixed(2))
      console.log("损失金额：",this.accountloss)
      if(this.yinkui>=1&&this.bzjlv>=100){
        this.isheshi = !this.isheshi
      }
      if(this.yinkui>=1){
        this.isyinkui = !this.isyinkui
      }
      console.log("盈亏比:",this.yinkui>=1)
      console.log("盈亏比:",this.yinkui)
      if(this.bzjlv>=500){
        this.isbzj = !this.isbzj
      }
      console.log("预付款比率:",this.bzjlv>=500)
      console.log("预付款比率:",this.bzjlv)
    }
  },
 
}
</script>

<style>

</style>
