<template>
  <div class="container">
    <div action="" class="form">
      <div class="title">
        <h1>Тикер</h1>
      </div>
      <input @keydown.enter="add" v-model="ticker" type="text" class="crypto" placeholder="Например doge или BTC">
      <button @click="add" type="button" class="add"><p><span id="mark">✖</span>Добавить</p></button>
    </div>

    <hr class="line" v-if="tickers.length!=0">
    <div class="tickers">
    <div class="ticker" v-for="(tick,index) in tickers" :key="index" @click="select(tick),startTick=true,init()" :class="{'border':sel===tick}">
      <div class="valute">{{tick.valute}}-USD</div>
      <div class="price">{{ tick.price }}</div>
      <div class="delete" @click.stop="deleteTicker(tick)">Удалить</div>
    </div>
  </div>
    <hr v-if="tickers.length!=0">
<!--    <div v-if="sel" class="graphics">-->
<!--      <div class="graph">-->
<!--        {{ sel.valute }}:-USD-->
<!--        <div class="bars">-->
<!--          <div v-for="(bar,idx) in graph" :key="idx" :style="{height:`${bar}%`}" class="bar">-->
<!--          </div>-->
<!--        </div>-->
<!--        <button @click="sel=null" class="extbtn">exit</button>-->
<!--      </div>-->
<!--    </div>-->
    <div v-if="sel" class="graphics">
      <div id="timer"></div>
      <div class="graph">
        <h1 id="titleGraph">{{ sel.valute }}:-USD</h1>
        <div class="bars">
          <div class="priceCoin"></div>
          <div v-for="(bar,idx) in normalizeGraph()" :key="idx" :style="{height:`${bar}%`}" class="bar" @click="showPrice(idx)"></div>
        </div>
        <button @click="sel=null" class="extbtn">✖</button>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data(){
    return{
      ticker:'',
      tickers:[],
      sel:null,
      graph:[],
      min:0,
      hour:0,
      sec:0,
      startTick:false,
      startInterval:0
    }
  },created() {
    const tickersData=localStorage.getItem('cryptolist')

    if (tickersData){
      this.tickers=JSON.parse(tickersData)
      this.tickers.forEach(currentTicker=>{setInterval(async ()=>{
        const f=await fetch(`https://min-api.cryptocompare.com/data/price?fsym=${currentTicker.valute}&tsyms=USD&api_key=b2a0c50f320ffb15bb2144476856e07ddfde53ea81f697fe8792730fff8f5fb2`)
        const data=await f.json();

        this.tickers.find(tick=>tick.valute===currentTicker.valute).price=data.USD>1?data.USD.toFixed(2):data.USD.toPrecision(2)
        if (this.sel?.valute===currentTicker.valute){
          this.graph.push(data.USD)
        }
      },3000)
        this.ticker=''})
    }
  },
  methods:{
    subscribeToUpdate(currentTicker){
      setInterval(async ()=>{
        const f=await fetch(`https://min-api.cryptocompare.com/data/price?fsym=${currentTicker.valute}&tsyms=USD&api_key=b2a0c50f320ffb15bb2144476856e07ddfde53ea81f697fe8792730fff8f5fb2`)
        const data=await f.json();

        this.tickers.find(tick=>tick.valute===currentTicker.valute).price=data.USD>1?data.USD.toFixed(2):data.USD.toPrecision(2)
        if (this.sel?.valute===currentTicker.valute){
          this.graph.push(data.USD)
        }
      },3000)
      this.ticker=''
    },
    add(){
      const newTicker={valute: this.ticker,
                        price:"-"};
      this.tickers.push(newTicker)
      localStorage.setItem('cryptolist',JSON.stringify(this.tickers ))
      this.subscribeToUpdate(newTicker)
    },
    deleteTicker(ticker){
      this.tickers=this.tickers.filter(t=>t!=ticker)
    },
    normalizeGraph(){
      const maxValue=Math.max(...this.graph)
      const minValue=Math.min(...this.graph)
      return this.graph.map(price =>  minValue === maxValue ? 100 : 5 + ((price - minValue) * 95) / (maxValue - minValue) )
    },select(ticker){
      this.sel=ticker;
      this.graph=[]
    },showPrice(bar){
      console.log(this.graph[bar])
      console.log(bar)
      document.querySelector('.priceCoin').innerHTML=this.graph[bar]+' USD'
    },init(){
      this.removeTick()
      if(this.startTick){
      this.startInterval=setInterval(this.tick,1000)}
    },tick(){
      this.sec++
      if (this.sec >= 60) { //задаем числовые параметры, меняющиеся по ходу работы программы
        this.min++;
        this.sec = this.sec - 60;
      }
      if (this.min >= 60) {
        this.hour++;
        this.min = this.min - 60;
      }
      if (this.sec < 10) { //Визуальное оформление
        if (this.min < 10) {
          if (this.hour < 10) {
            document.getElementById('timer').innerHTML ='0' + this.hour + ':0' + this.min + ':0' + this.sec;
          } else {
            document.getElementById('timer').innerHTML = this.hour + ':0' + this.min + ':0' + this.sec;
          }
        } else {
          if (this.hour < 10) {
            document.getElementById('timer').innerHTML = '0' + this.hour + ':' + this.min + ':0' + this.sec;
          } else {
            document.getElementById('timer').innerHTML = this.hour + ':' + this.min + ':0' + this.sec;
          }
        }
      } else {
        if (this.min < 10) {
          if (this.hour < 10) {
            document.getElementById('timer').innerHTML = '0' + this.hour + ':0' + this.min + ':' + this.sec;
          } else {
            document.getElementById('timer').innerHTML = this.hour + ':0' + this.min + ':' + this.sec;
          }
        } else {
          if (this.hour < 10) {
            document.getElementById('timer').innerHTML = '0' + this.hour + ':' + this.min + ':' + this.sec;
          } else {
            document.getElementById('timer').innerHTML = this.hour + ':' + this.min + ':' + this.sec;
          }
        }
      }
    },
    removeTick(){
      clearInterval(this.startInterval)
      this.sec=0
      this.min=0
      this.hour=0
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.line{
  margin-top: 50px;
}
.form{
  position: relative;
  margin-left: 1%;
  height: 150px;
}
.container{
  width: 80%;
  margin: 0 auto;
  background-color: darkseagreen;
}
.title{
  font-size: 20px;
}
.crypto{
  display: block;
  height: 40px;
  width: 380px;
}
@media (max-width: 600px) {
  .crypto{
    width: 300px;
  }
}
.add{
  margin-top: 5px;
  height: 40px;
  width: 200px;
  font-size: 15px;
  background: none;
  border: solid 2px teal;
  color: teal;
  cursor: pointer;
  position: absolute;
  transition: all .5s ease;
  box-shadow: 0 5px darkslategray;
}
.add:hover{
  background-color: #6f946f;
  margin-top: 7px;
  box-shadow: 0 3px darkslategray;
}
.add p{
  text-align: center;
  justify-content: center;
  margin-top: 10px;
}
#mark{
  font-size: 30px;
  position: absolute;
  left: 10px;
  top: 2px;
  transition: all .5s ease;
}
#mark:hover{
  transform: rotate(405deg);
  color: darkslategrey;
}

/*-----------------------*/
.tickers{
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  margin-top: 40px;
  justify-content: center;
}
.ticker{
  margin: 10px;
  width: 180px;
  height: 180px;
  background-color: steelblue;
  text-align: center;
  vertical-align: middle;
  transition: all .2s ease;
  box-shadow: 5px 5px slategray;
}
.ticker:hover{
  background-color: cadetblue;
  box-shadow: 2px 2px slategrey;
  /*margin: 13px 10px 10px 13px ;*/
  transform: translate(3px,3px);
}
@media (max-width: 600px) {
  .ticker{
    width: 200px;
    margin: 0 auto 10px auto;
  }

  .tickers{
    flex-direction: column;

  }
}
.border{
  border: solid 4px darkslategrey;
}
.valute{
  font-size: 20px;
  padding-top: 5px;
}

.price{
  padding-top: 30px;
  font-size: 30px;
  cursor: pointer;
  transition: all .5s ease;
}
html{
  font-family: "Arial Hebrew Scholar";
}
.price:hover{
  color: black;
}

.delete{
  /*margin-top: 40px;*/
  background-color: blue;
  width: 60%;
  justify-content: center;
  margin: 50px auto;
  color: aqua;
  border-radius: 20px;
  cursor: pointer;
}
.delete:hover{
  color: aquamarine;
  background-color: deepskyblue;
}

/*-------------------------------*/
.graphics{
  margin-top: 15px;

}
.graph{
  width: 80%;
  height: 400px;
  background-color: teal;
  margin: 0 auto;
  position: relative;
  outline: 2px solid midnightblue;
}
.extbtn{
  position: absolute;
  right: 0;
  top: 0;
  background:none;
  /*border: solid 3px darkslategrey;*/
  border: none;
  font-size: 30px;
  cursor: pointer;
  transition: all .5s ease;
}
.extbtn:hover{
  transform: rotate(180deg);
  color: darksalmon;
  font-size: 35px;
}

.bar{
  width: 30px;
  background-color: maroon;
  border: solid 1px darkblue;
  /*transform: rotate(180deg);*/
  transition: all .2s ease;
}

.bar:hover{
  background-color: brown;
  width: 100px;
}
.bars{
  width: 100%;
  display: flex;
  align-items: flex-end;
  height: 300px;
  transition: all 1s ease;
  overflow: hidden;
  /*outline: 2px  solid darkslateblue;*/
  border-bottom: 5px solid darkblue;
}
.bars:hover{

}
#titleGraph{
  margin-bottom: 50px;
}
#timer{
  position: absolute;
  margin: 0 auto;
  font-size: 30px;
  left: 50%;
  z-index: 30;
}
.priceCoin{
  font-size: 30px;
  width: 100%;
  height: 60px;
  position: absolute;
  left: 50%;
  top: 30px;
}
@media (max-width: 700px) {
  .priceCoin{
    left: 20%;
  }
}
</style>
