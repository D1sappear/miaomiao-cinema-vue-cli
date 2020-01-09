<template>
  <div class="city_body">
    <!--<div class="city_list">
      <div class="city_hot">
        <h2>热门城市</h2>
        <ul class="clearfix">
          <li>上海</li>
        </ul>
      </div>
      <div class="city_sort">
        <div>
          <h2>A</h2>
          <ul>
            <li>阿拉善盟</li>
          </ul>
        </div>
      </div>
    </div>
    <div class="city_index">
      <ul>
        <li>A</li>
        <li>B</li>
        <li>C</li>
        <li>D</li>
        <li>E</li>
      </ul>
    </div>-->
    <div class="city_list">
      <Loading v-if="isLoading" />
      <Scroller v-else :handleToScroll="handleToScroll" :handleToTouchEnd="handleToTouchEnd" ref="city_list">
        <div>
          <div class="city_hot">
            <h2>热门城市</h2>
            <ul class="clearfix">
              <li v-for="item in hotCityList" :key="item.id" @tap="handleToCity(item.nm, item.id)">{{item.nm}}</li>
            </ul>
          </div>
          <div class="city_sort" ref="city_sort" >
            <div v-for="item in cityList" :key="item.index" >
              <h2>{{item.index}}</h2>
              <ul>
                <li v-for="itemList in item.list" :key="itemList.id" @tap="handleToCity(itemList.nm, itemList.id)">{{itemList.nm}}</li>
              </ul>
            </div>
          </div>
        </div>
      </Scroller>
    </div>
    <div class="city_index">
      <ul>
        <li v-for="(item, index) in cityList" :key="item.index" @touchstart="changeScroll(index)">{{item.index}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'City',
    data() {
      return {
        cityList: [],
        hotCityList: [],
        isLoading: true
      }
    },
    mounted() {
      var cityList = window.localStorage.getItem('cityList')
      var hotCityList = window.localStorage.getItem('hotCityList')

      if (cityList && hotCityList) {
        this.cityList = JSON.parse(cityList)
        this.hotCityList = JSON.parse(hotCityList)
        this.isLoading = false
      } else {
        this.axios.get('/api/cityList').then((res) => {
          // console.log(res);
          const msg = res.data.msg
          if (msg === 'ok') {
            const cities = res.data.data.cities
            // [ {index: 'A', list: [ {nm: '阿城', id: 1} ]} ]
            const {cityList, hotCityList} = this.formatCities(cities)
            this.cityList = cityList
            this.hotCityList = hotCityList
            // console.log(this.hotCityList);
            this.isLoading = false
            window.localStorage.setItem('cityList', JSON.stringify(cityList))
            window.localStorage.setItem('hotCityList', JSON.stringify(hotCityList))
          }
        })
      }
    },
    methods: {
      formatCities(cities) {
        const cityList = []
        const hotCityList = []

        for (var i=0; i<cities.length; i++) {
          if (cities[i].isHot === 1) {
            hotCityList.push(cities[i])
          }
        }

        for (var i=0; i<cities.length; i++) {
          const firstLetter = cities[i].py.substring(0,1).toUpperCase()
          if(toCom(firstLetter)) { // cityList内不存在
            cityList.push({index: firstLetter, list: [{nm: cities[i].nm, id: cities[i].id}]})
          } else { // cityList内已存在
            for (var j=0; j<cityList.length; j++) {
              if (cityList[j].index === firstLetter) {
                cityList[j].list.push({nm: cities[i].nm, id: cities[i].id}) // 更新数据
              }
            }
          }
        }

        cityList.sort((n1,n2) => {
          if (n1.index > n2.index) {
            return 1
          } else if (n1.index < n2.index) {
            return -1
          } else {
            return 0
          }
        })

        function toCom(firstLetter) {
          for (var i=0; i<cityList.length; i++) {
            if (cityList[i].index === firstLetter) { // 已存在字母索引
              return false
            }
          }
          return true
        }
        // console.log(cityList);
        // console.log(hotCityList);

        return {
          cityList,
          hotCityList
        }
      },
      changeScroll(index) {
        const h2 = this.$refs.city_sort.getElementsByTagName('h2')
        // this.$refs.city_sort.parentNode.scrollTop = h2[index].offsetTop
        this.$refs.city_list.toScrollTop(-h2[index].offsetTop)
      },
      handleToScroll (pos) {
        if (pos.y > 30) {
          this.pullDownMsg = '正在更新中'
        }
      },
      handleToTouchEnd (pos) {
        if (pos.y > 30) {
          this.axios.get('/api/movieOnInfoList?cityId=10').then((res) => {
            var msg = res.data.msg
            if (msg === 'ok') {
              this.pullDownMsg = '更新完成'
              setTimeout(() => {
                this.movieList = res.data.data.movieList
                this.pullDownMsg = ''
              }, 1000)
            }
          })
        }
      },
      handleToCity(nm, id) {
        this.$store.commit('city/CITY_INFO', {nm, id})
        window.localStorage.setItem('nowNm', nm)
        window.localStorage.setItem('nowId', id)
        this.$router.push('/movie/nowPlaying')
      }
    }
  }
</script>

<style scoped>
  #content .city_body{ margin-top: 45px; display: flex; width:100%; position: absolute; top: 0; bottom: 0;}
  .city_body .city_list{ flex:1; overflow: auto; background: #FFF5F0;}
  .city_body .city_list::-webkit-scrollbar{
    background-color:transparent;
    width:0;
  }
  .city_body .city_hot{ margin-top: 20px;}
  .city_body .city_hot h2{ padding-left: 15px; line-height: 30px; font-size: 14px; background:#F0F0F0; font-weight: normal;}
  .city_body .city_hot ul li{ float: left; background: #fff; width: 29%; height: 33px; margin-top: 15px; margin-left: 3%; padding: 0 4px; border: 1px solid #e6e6e6; border-radius: 3px; line-height: 33px; text-align: center; box-sizing: border-box;}
  .city_body .city_sort div{ margin-top: 20px;}
  .city_body .city_sort h2{ padding-left: 15px; line-height: 30px; font-size: 14px; background:#F0F0F0; font-weight: normal;}
  .city_body .city_sort ul{ padding-left: 10px; margin-top: 10px;}
  .city_body .city_sort ul li{ line-height: 30px;}
  .city_body .city_index{ width:20px; display: flex; flex-direction:column; justify-content:center; text-align: center; border-left:1px #e6e6e6 solid;}
</style>
