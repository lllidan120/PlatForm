<template>
  <div class="manyAreaSelect">
    <div class="item">
      <div class="item-title"> 
        <span> 选择省</span>
      </div>
      <div class="item-content">
        <ul>
          <li v-for="(item,index) in chinaArea" :class="item.selected?'active':''" :key="index" @click="getCityList(item)">{{item.Name}}</li>
        </ul>
      </div>
      <div class="item-footer"></div>
    </div>
    <div class="item">
      <div class="item-title">
        <span>选择市</span>
      </div>
      <div class="item-content">
        <ul v-show="cityList.length===0">
          <li><<请选择省份</li>
        </ul>
        <ul v-show="cityList.length!==0">
          <li v-for="(item,index) in cityList" :class="item.selected ? 'active':''" :key="index" @click="getDistricList(item)">{{item.Name}}</li>
        </ul>
      </div>
   <!--    <div class="item-footer">
        <button type="button" class="button" :class="notLimitButton.notLimitCity?'success':''" @click="cityNotLitmit({Name:'不限',Code:'0'})"  :disabled="!selectItem.province.Name">不限城市</button>
      </div> -->
    </div>
    <div class="item">
      <div class="item-title">
        <span>选择地区</span>
      </div>
      <div class="item-content">
        <ul v-show="districList.length===0">
          <li>
            <<请选择城市</li>
          </ul>
          <ul v-show="districList.length!==0">
            <li v-for="(item,index) in districList" :class="item.selected?'active':''" :key="index" @click="getAreaCombineID(item)">{{item.Name}}</li>
          </ul>
        </div>
    <!--   <div class="item-footer">
        <button type="button" class="button" :class="notLimitButton.notLimitDistrict ?'success':''" @click="districNotLitmit({Name:'不限',Code:'0'})" :disabled="!selectItem.city.Name ||!selectItem.province.Name || notLimitButton.notLimitCity ">不限地区</button>
      </div> -->
    </div>
    <div class="trangle">
      <div class="trangle-wrap">
        <div class="left">
          <button type="button" class="button" @click="transferToRight" :disabled="direactionStatusToRight">
            <Icon type="ios-arrow-forward"></Icon>
          </button>
        </div>
        <div class="right">
          <button type="button" class="button" @click="transferToLeft" :disabled="direactionStatusToLeft">
            <Icon type="ios-arrow-back"></Icon>
          </button>
        </div>
      </div>
    </div>
    <div class=" item ">
      <div class="item-title ">
        <span>已选道路</span>
      </div>
      <div class="item-content ">
        <ul class="selectedContent">
          <li v-for="(item,index) in selectedList" :class="item.selected?'active':''" :key="index" @click="selectedAreaSingle(item)">{{item.Name}}</li>
        </ul>
      </div>
      <div class="item-footer">
        <button type="button" class="button" @click="selectedAllArea()" :disabled="rightDataList.length=== 0" :class="selectedAllButtonStatus?'success':''">{{selectedAllButtonStatus?'反选':'全部'}}</button>
      </div>
    </div>
  </div>
</template>

<script>
  import Util from '@/libs/util';
  import _ from 'lodash';
  export default {
    name: 'manyAreaSelect',
    data: function () {
      return {
        chinaArea: JSON.parse(window.localStorage.getItem('chinaArea')) || [], // 这是地区联动的JSON
        notLimitButton: {
          notLimitCity: false,  // 城市不限
          notLimitDistrict: false,  // 地区不限
        },
        selectedAllButtonStatus: false, // 已选位置列表全部按钮的状态
        selectItem: {
          province: {},
          city: {},
          distric: {}
        },
        cityList: [], // 城市列表
        districList: [], // 区域列表
        rightDataList: [], // 选中项目组合成的渲染列表
        rightData: [], // 选中需要移除的
        leftData: [], // 左边选中的转发
      }
    },
    props: {
      selectedData: {
        type: [String, Object, Array]
      },
      iconDirection: {
        type: Object,
        default: function () { // 箭头图标
          return {
            left: 'fzicon fz-ad-you',
            right: 'fzicon fz-ad-right'
          }
        }
      }
    },
    computed: {
      selectedList () {  // 已选中列表
        if (this.selectedData && this.selectedData !== '') {
          this.rightDataList = this.selectedData;
          return this.rightDataList;
        } else {
          return this.rightDataList;
        }

      },
      direactionStatusToRight () { // 控制可以转移的箭头状态
        if (this.notLimitButton.notLimitCity || this.notLimitButton.notLimitDistrict) {
          if (this.notLimitButton.notLimitCity) {
            this.removeAllSelected(this.cityList);
            this.removeAllSelected(this.districList);
            return false;
          } else {
            if (this.notLimitButton.notLimitDistrict) {
              this.removeAllSelected(this.districList);
              return false;
            }
          }
          return false;
        } else {
          if (this.selectItem.distric.Name) {
            return false;
          }
          return true;
        }
      },
      direactionStatusToLeft () { // 控制可以转移的箭头状态
        if (this.rightData.length === 0) {
          return true
        } else {
          return false
        }
      }
    },
    methods: {
      mapSelect (list, value, type) {  // 高亮选中
        if (type) {
          return list.map(pitem => {
            if (pitem.Code === value.Code) {
              if (value.selected && value.selected === true) {
                this.$delete(pitem, 'selected');
              } else {
                this.$set(pitem, 'selected', true)
              }
            }
          })
        } else {
          return list.map(pitem => {
            if (pitem.Code === value.Code) {
              if (value.selected && value.selected === true) {
                this.$delete(pitem, 'selected');
              } else {
                this.$set(pitem, 'selected', true)
              }
            } else {
              this.$delete(pitem, 'selected');
            }
          })
        }
      },
      resetToDefault () {
        this.leftData = []; // 清空需要转移的数组
        this.notLimitButton = {  // 重置按钮状态
          notLimitCity: false,  // 城市不限
          notLimitDistrict: false,  // 地区不限
        };
        this.selectItem.city = {};
        this.selectItem.distric = {}
        this.removeAllSelected(this.cityList);  // 清除选中状态
        this.removeAllSelected(this.districList); // 清除选中状态
        this.cityList = [];
        this.districList = [];
      },
      getCityList (item) {
        var _this = this;
        this.resetToDefault();
        if (item) {
          _this.getAreaInfo(item.Code).then((res) => {
            _this.cityList = res
            _this.selectItem.province = item;
            _this.mapSelect(_this.chinaArea, item);
          })
        }
      },
      getDistricList (item) {
        var _this = this;
        this.leftData = []; // 清空需要转移的数组
        this.notLimitButton.notLimitDistrict = false; // 重置按钮状态
        this.removeAllSelected(this.districList); // 清除选中状态
        this.selectItem.distric = {};
        this.districList = [];
        if (item) {
          _this.getAreaInfo(item.Code).then((res) => {
            _this.districList = res
            _this.selectItem.city = item
            _this.mapSelect(_this.cityList , item)

            _this.leftRightDataMixin()

          })





        }

      },
      getAreaCombineID (item) {  // 获取组合ID
        if(!item.selected ) {
          this.selectItem.distric = item;
          this.mapSelect(this.districList , item , 'manySelect');

          this.leftData.push({
            Name: item.Name,
            Code: item.Code
          })
          // this.leftData = _.uniqBy(this.leftData , 'Code');
          this.rightDataList.push({
            Name: item.Name,
            Code: item.Code
          })
        }


      },
      cityNotLitmit (item) {  // 城市不限
        this.leftData = []; // 请空数组
        this.selectItem.distric = {};
        this.removeAllSelected(this.districList);
        this.notLimitButton.notLimitCity = !this.notLimitButton.notLimitCity; // 不限按钮状态
        this.leftData.push({
          Name: this.selectItem.province.Name + '-不限-不限',
          Code: this.selectItem.province.Code + '-0-0'
        })
      },
      districNotLitmit (item) { // 区域不限
        this.leftData = []; // 请空数组
        this.notLimitButton.notLimitDistrict = !this.notLimitButton.notLimitDistrict; // 不限按钮状态
        this.leftData.push({
          Name: this.selectItem.province.Name + '-' + this.selectItem.city.Name + '-不限',
          Code: this.selectItem.province.Code + '-' + this.selectItem.city.Code + '-0'
        })
        if (!this.notLimitButton.notLimitDistrict) {
          this.leftData = [];
        }
      },
      transferToRight () { // 选中推入到已选中列表区域

        if (this.leftData && this.leftData.length !== 0) {
          if (this.leftData.length === 1) {  // 长度只有1,那就只有不限城市或者地区了
            let limitId = this.leftData[0].Code.split('-');  // 比对比对,切割成数组
            debugger
            this.rightDataList.map(item => {
              // let id = item.Code.split('-');
              let id = item.Code
              if (limitId[0] === id[0]) {
                if (limitId[1] === '0') {  // 不限城市
                  this.rightDataList = this.rightDataList.filter(ritem => {
                    let rid = ritem.Code.split('-');
                    if (limitId[0] !== rid[0]) {
                      return ritem;
                    }
                  })
                } else {
                  if (limitId[2] === '0') { // 不限地区
                    this.rightDataList = this.rightDataList.filter(ritem => {
                      let rid = ritem.Code.split('-');
                      if ((limitId[0] === rid[0] && limitId[1] === rid[1])) {
                        if (ritem[2] === '0') {
                          return ritem;
                        }
                      } else {
                        if (limitId[0] !== rid[0] || limitId[1] !== rid[1]) {
                          return ritem;
                        }
                      }
                    })
                  } else {
                    this.rightDataList = this.rightDataList.filter(ritem => {
                      let rid = ritem.Code.split('-');
                      if (limitId[0] === rid[0]) {
                        if (limitId[1] === rid[1]) {
                          if (!(rid[2] === '0')) {
                            return ritem;
                          }
                        } else {
                          if (!(rid[1] === '0')) {
                            return ritem
                          }
                        }
                      } else {
                        return ritem
                      }
                    })
                  }
                }

              }
            })
          } else {
            // let limitId = this.leftData[0].Code.split('-');  // 比对比对,切割成数组
            this.rightDataList = this.rightDataList.filter(ritem => {
              // let rid = ritem.Code.split('-');
              let rid = ritem.Code;

              return ritem
            })
          }
          this.leftData.map(item => {
            this.rightDataList.push(item);
          })
          this.rightDataList = _.uniqBy(this.rightDataList, 'Code');
          console.log(this.rightDataList)

          this.resetToDefault();
        }


      },
      selectedAreaSingle (item) {  // 已选择区域单个选择
        if (item) {
          this.rightData = [];
          // this.mapSelect(this.rightDataList, item, 'manySelect'); // 区域高亮选择,多选
          // this.rightDataList.map((ritem) => {
          //   if(item.Code != ritem.Code) {
          //     debugger
          //     this.rightData.push(ritem)
          //   }
          // })

          // this.rightDataList = this.rightData
          var result = _.result(_.find(this.rightDataList, function(chr) {
            console.log()
            return chr.Code != item.Code;
          }), 'user');
          console.log(result)
        }

      },
      selectedAllArea () { // 已选中区域全选反选
        if (this.selectedAllButtonStatus) {
          this.removeAllSelected(this.rightDataList);
          this.rightData = [];
        } else {
          this.rightDataList.map(item => this.$set(item, 'selected', true));
          this.rightData = this.rightDataList;
        }
        this.selectedAllButtonStatus = !this.selectedAllButtonStatus;
      },
      transferToLeft () {  // 从已选中列表区域退回待转发区域
        if (this.rightData && this.rightData.length !== 0) {
          this.rightDataList = this.rightDataList.filter(item => {
            if (!item.selected) {
              return item;
            }
          })
          this.rightData = [];
        }
      },
      removeAllSelected (list) {  // 清空选中状态
        list.map(item => this.$delete(item, 'selected'));
      },
      getAreaInfo(code) {
        var par = {
          $PCode: code ? code : 0
        }
        return Util.getAjaxData(par , 'GetAreaInfo' , undefined).then((res)=>{
          return Promise.resolve(res)
        })
      },
      leftRightDataMixin () {
        var leftData = this.leftData;
        var rightDataList = this.rightDataList;
        var distric = this.districList;
        this.leftData = []
        distric.map((lItem)=>{
          rightDataList.map((rItem)=>{
            if(rItem.Code == lItem.Code){
              this.$set(lItem, 'selected', true)
            }
          })
        })
      }
    },
    created () {
      var _this = this;
      _this.getAreaInfo(0).then((res)=>{
        _this.chinaArea = res
      })
    },
    watch: {
      'rightDataList' (newValue, oldValue) {  // 选择列表的值变动响应外部值的变动
        if (newValue.length !== this.rightData.length) {
          this.selectedAllButtonStatus = false;
        } else {
          if (newValue.length === 0) {
            this.selectedAllButtonStatus = false;
          } else {
            this.selectedAllButtonStatus = true;
          }
        }
        this.$emit('update:selectedData', newValue);
      },
      'districList' (newValue, oldValue) {
        if(newValue.length != 0) {
          // this.leftRightDataMixin()
        }
      }

    }
  }
</script>

<style scoped lang="less">
  ul {
    padding: 0;
    margin: 0;
    max-height: 100%;
    overflow-y: auto;
    li {
      cursor: pointer;
      text-align: center;
      padding: 5px;
      &.active,
      &:hover {
        background: #e4e8f1;
        color: #48576a;
      }
    }
  }

  .manyAreaSelect {
    position: relative;
    z-index: 2005;
    .item {
      border: 1px solid #d1dbe5;
      background: #fff;
      box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);
      display: inline-block;
      vertical-align: middle;
      min-width: 180px;
      box-sizing: border-box;
      position: relative;
      height: 100%;
      min-height: 260px;
      margin: 5px;
    }
    .item-title {
      height: 36px;
      line-height: 36px;
      background: #fbfdff;
      margin: 0;
      border-bottom: 1px solid #d1dbe5;
      box-sizing: border-box;
      color: #1f2d3d;
      text-align: center;
    }
    .trangle {
      background: transparent;
      display: inline-block;
      vertical-align: middle;
      width: 40px;
      box-sizing: border-box;
      height: 100%;
      position: relative;
      .trangle-wrap {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
      .left,
      .right {
        margin: 10px 5px;
      }
      ;
    }

    .item-content {
      font-size: 13px;
      height: 190px;
      padding: 8px 2px;
    }
    .item-footer {
      padding: 5px 0;
      height: 40px;
      text-align: center;
    }
  }

  .selectedContent {
    li {
      text-align: left;
      padding-left: 25px;
    }
  }



  .button {
    display: inline-block;
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    background: #fff;
    border: 1px solid #c4c4c4;
    color: #1f2d3d;
    margin: 0;
    border-radius: 4px;
    padding: 8px;
    font-size: 12px;
    -webkit-appearance: button;
    outline: none;
    &.success {
      background: #42d885;
      border-color: #42d885;
      color: #fff;
    }

    &:disabled {
      color: #bfcbd9;
      cursor: not-allowed;
      background-image: none;
      background-color: #eef1f6;
      border-color: #d1dbe5;
    }
  }
</style>