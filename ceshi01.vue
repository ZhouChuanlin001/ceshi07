<template>
  <div class="clinic_search_panel" ref="clinic_search_panel"
       @click.prevent.stop="">
    <div class="clinic_search_panel_input_wrap flex" ref="search_panel_input_wrap">
      <label class="flex_1">
        <input class="clinic_search_panel_input" type="text" :value="value" ref="search_panel_input"
               :class="disabled ?'clinic_search_panel_input_disabled':''"
               :placeholder="showPlaceHolder?placeholder:''"
               v-on="inputListeners"
               :disabled="disabled"
               @focus="focusInput"
               @keydown.down="nextItem"
               @keydown.up="lastItem"
               @keydown.enter="enterSelectItem">
      </label>
      <slot name="append"></slot>
    </div>

    <!--    <transition name="fade" mode="out-in" v-cloak>-->
    <ul v-if="showPanel" class="clinic_search_panel_result_box" id="clinic_search_panel_result_box"
        :style="{height:getPanelHeight,width:searchPanelWidth?searchPanelWidth+'px':''}"
        ref="bodyWrapper" @scroll="onScroll" @click.prevent.stop="" @mousedown.prevent.stop="">
      <li class="flex flex_center_ver clinic_search_panel_result_title"
          :style="'width:'+getTotalWith+'px'"
          ref="searchPanelTitle">
            <span class="clinic_search_panel_result_title_span text-overflow"
                  v-for="(item,index) in panelConfig"
                  :title="item.title"
                  :style="{width:item.width+'px',flexShrink:0,background: '#e4e7ed'}">{{item.title}}</span>
      </li>
      <div style="background: #FFFFFF" :style="'width:'+getTotalWith+'px'">
        <ul class="" style="background: #FFFFFF" v-if="showResult">
          <li class="flex flex_center_ver clinic_search_panel_result_item"
              v-for="(item ,index) in result" :class="{active:index===nowIndex}"
              :key="index"
              @click.prevent.stop="itemClick(item)">
            <span class="clinic_search_panel_result_item_span text-overflow" :title="item[_item.prop]"
                  v-for="(_item,_index) in panelConfig"
                  :style="{width:_item.width+'px',flexShrink:0,borderBottom:'1px solid #DDDDDD'}"
            >{{item[_item.prop]}}</span>
          </li>
        </ul>
        <div style="height: 100px;background: #FFFFFF" class="flex flex_center_all"
             v-show="noResult">
          <span style="color: red;font-size: 20px">未查询到相关数据</span>
        </div>
      </div>
    </ul>
    <!--    </transition>-->

  </div>

</template>

<script>

  export default {
    name: "searchPanel",
    computed: {
      inputListeners: function () {
        let vm = this;
        let vmListeners = vm.$listeners;
        let inputListeners = {
          "focus": function () {
            if (vmListeners.focus && typeof vm.focus === "function") {
              vmListeners.focus();
            }
            // searchOnFocusAgain 与 executeSearchOnfocus 互斥
            if (vm.searchOnFocusAgain && !vm.executeSearchOnfocus) {
              vm.resetPanel();
              let inputContent = vm.needTrim ? event.target.value.trim() : event.target.value;
              vm.$emit("input", inputContent);
              if (inputContent.length < 1) {
                vm.$nextTick(() => {
                  vm.showPanel = false;
                });
              }
              clearTimeout(vm.queryTimer);
              if (vm.querySearchAsync && typeof vm.querySearchAsync === "function"
                && inputContent.length >= vm.searchKeyLength && inputContent) {
                vm.scrollCheck = false;
                vm.queryTimer = setTimeout(async function () {
                  vm.querySearchAsync(vm.callBack, ...vm.extraArguments, 1);
                  vm.pageNumber = 2;
                }, Number(vm.delay));
              }
            } else if (vm.executeSearchOnfocus && !vm.searchOnFocusAgain) {
              if (vm.querySearchAsync && typeof vm.querySearchAsync === "function") {
                vm.resetPanel();
                vm.querySearchAsync(vm.callBack, ...vm.extraArguments, 1);
              }
            } else if (vm.executeSearchOnfocus && vm.searchOnFocusAgain) {
              throw new Error("prop: executeSearchOnfocus and searchOnFocusAgain  all is true");
            }
          },
          "blur": function () {
            //vm.resetPanel();
            if (vmListeners.blur && typeof vm.blur === "function") {
              vmListeners.blur();
            }
          },
          "change": vmListeners.change || function () {
          }
        };
        return Object.assign({}, inputListeners, {
            keyup: function (e) {
              let keyCode = window.event ? e.keyCode : e.which;
              if (vm.excludeKeyCode.includes(keyCode)) return;
              let flag = e.target.isNeedPrevent;
              if (flag) return;
              vm.resetPanel();
              let inputContent = vm.needTrim ? e.target.value.trim() : e.target.value;
              vm.$emit("input", inputContent);
              if (inputContent.length < 1) {
                vm.$nextTick(() => {
                  vm.showPanel = false;
                });
              }
              clearTimeout(vm.queryTimer);
              if (vm.querySearchAsync && typeof vm.querySearchAsync === "function"
                && inputContent.length >= vm.searchKeyLength && inputContent) {
                vm.scrollCheck = false;
                vm.queryTimer = setTimeout(async function () {
                  vm.querySearchAsync(vm.callBack, ...vm.extraArguments, 1);
                  vm.pageNumber = 2;
                  e.target.keyEvent = false;
                }, Number(vm.delay));
              }

            },
            keydown: function (e) {
              e.target.keyEvent = true;
            },
            compositionstart: function (e) {
              e.target.isNeedPrevent = true;
            },
            compositionend: function (e) {
              e.target.isNeedPrevent = false;
              vm.resetPanel();
              let inputContent = vm.needTrim ? e.target.value.trim() : e.target.value;
              vm.$emit("input", inputContent);
              if (inputContent.length < 1) {
                vm.$nextTick(() => {
                  vm.showPanel = false;
                });
              }
              clearTimeout(vm.queryTimer);
              if (vm.querySearchAsync && typeof vm.querySearchAsync === "function"
                && inputContent.length >= vm.searchKeyLength && inputContent) {
                vm.scrollCheck = false;
                vm.queryTimer = setTimeout(async function () {
                  vm.querySearchAsync(vm.callBack, ...vm.extraArguments, 1);
                  vm.pageNumber = 2;
                }, Number(vm.delay));
              }
            },
            input: function (event) {
              if (event.target.isNeedPrevent) return;
              vm.resetPanel();
              let inputContent = vm.needTrim ? event.target.value.trim() : event.target.value;
              vm.$emit("input", inputContent);
              if (inputContent.length < 1) {
                vm.$nextTick(() => {
                  vm.showPanel = false;
                });
              }
              clearTimeout(vm.queryTimer);
              if (vm.querySearchAsync && typeof vm.querySearchAsync === "function"
                && inputContent.length >= vm.searchKeyLength && inputContent) {
                vm.scrollCheck = false;
                vm.queryTimer = setTimeout(async function () {
                  vm.querySearchAsync(vm.callBack, ...vm.extraArguments, 1);
                  vm.pageNumber = 2;
                }, Number(vm.delay));
              }
            }
          }
        );
      },
      panelConfig: function () {
        let vm = this;
        let resultArr = [];
        for (let i = 0; i < vm.resultConfig.length; i++) {
          let obj = vm.resultConfig[i];
          if (obj.width) {
            obj.width = parseInt(obj.width);
          } else {
            obj.width = 100;
          }
          resultArr.push(obj);
        }
        return resultArr;
      },
      getTotalWith: function () {
        let arr = this.panelConfig;
        let width = 0;
        for (let i = 0; i < arr.length; i++) {
          width += arr[i].width;
        }
        return width + arr.length - 1;
      },
      getPanelHeight() {
        let h = Number(this.panelHeight);
        if (Number.isNaN(h) || h < 190) {
          h = "190px";
        } else {
          h = h + "px";
        }
        return h;
      }
    },
    props: {
      "value": {},
      "delay": {default: 200},
      "searchResult": {},
      "handSelect": {},
      "focusInput": {},
      "querySearchAsync": {},
      "resultConfig": {},
      "searchPanelWidth": {default: 0},
      "placeholder": {default: "请输入搜索关键字"},
      "showPlaceHolder": {
        default: "true"
      },
      "panelHeight": {
        default: 190
      },
      "disabled": {default: false},
      "selectFirstItem": {default: true},
      "showNoResult": {default: true},
      "searchKeyLength": {default: 1, type: Number},
      "infiniteScroll": {type: Boolean, default: false},//是否无限滚动
      //el_table 聚焦最后一行第一个input
      "focusFirstInput": {
        default: false
      },
      //额外的回调参数,会在查询和选择的时候注入
      "extraArguments": {
        type: Array,
        default: () => {
          return [];
        }
      },
      "searchOnFocusAgain": {
        type: Boolean,
        default: false
      },
      "needTrim": {
        type: Boolean,
        default: true
      },
      //聚焦时执行搜索
      "executeSearchOnfocus": {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        excludeKeyCode: [13, 16, 17, 20, 37, 38, 39, 40, 46],
        nowIndex: -1,
        showPanel: false,
        showResult: false,
        noResult: false,
        result: [],
        queryTimer: null,
        loadingInstance: null,
        scrollIndex: 21,//滚动条底部与组件底部的距离
        scrollCheck: true,//是否在滚动事件中加载
        scrollCheckIndex: 0,//滚动条长度
        pageNumber: 2//滚动查询第几页
      };
    },
    methods: {
      queryAction: function (str) {
        this.focus();
        this.$emit("input", str);
        this.$nextTick(() => {
          this.querySearchAsync(this.callBack, ...this.extraArguments, 1);
        });
      },
      nextItem() {
        if (!this.showPanel) return;
        this.nowIndex++;
        if (this.nowIndex === this.result.length) {
          this.nowIndex = 0;
          this.$refs.bodyWrapper.scrollTop = 0;
        }
        if (this.nowIndex !== 0) this.$refs.bodyWrapper.scrollTop += 28;
      },
      lastItem() {
        if (!this.showPanel) return;
        this.nowIndex--;
        if (this.nowIndex <= -1) {
          this.nowIndex = this.result.length - 1;
          this.$refs.bodyWrapper.scrollTop = 9999999;
        } else {
          this.$refs.bodyWrapper.scrollTop -= 28;
        }
      },
      onScroll(e) {
        this.$refs.searchPanelTitle.style.top = this.$refs.bodyWrapper.scrollTop + "px";
        this.scrollIndex = e.target.scrollHeight - e.target.scrollTop - e.target.clientHeight;
        this.scrollCheckIndex = e.target.scrollHeight;
        this.scrollCheckFn(e);
      },
      scrollCheckFn(e) {
        let vm = this;
        setTimeout(() => {
          if (vm.scrollCheck && vm.scrollIndex <= 20 && e.target.scrollTop !== 0
            && vm.infiniteScroll && vm.querySearchAsync && typeof vm.querySearchAsync === "function") {
            vm.scrollCheck = false;
            vm.querySearchAsync((res) => {
              vm.result.push(...res);
            }, ...vm.extraArguments, vm.pageNumber++);
          }
        }, Number(vm.delay));
      },
      async enterSelectItem() {
        if (!this.showPanel || this.nowIndex < 0) {
          this.hiddenPanel();
          return;
        }
        if (this.handSelect && typeof this.handSelect === "function") {
          if (this.result[this.nowIndex]) {
            await this.handSelect(this.result[this.nowIndex], ...this.extraArguments);
          }
        }
        if (this.focusFirstInput) {
          this.$nextTick(() => {
            this._focusFirstInput(this.focusFirstInput);
          });
        }
        this.resetPanel();
        this.showPanel = false;
      },
      async itemClick(item) {
        if (this.handSelect && typeof this.handSelect === "function") {
          await this.handSelect(item, ...this.extraArguments);
        }
        if (this.focusFirstInput) {
          this.$nextTick(() => {
            this._focusFirstInput(this.focusFirstInput);
          });
        }
        this.resetPanel();
        this.showPanel = false;
      },
      callBack(res) {
        let inputEle = this.$refs["search_panel_input"];
        let nowEle = document.activeElement;
        if (inputEle !== nowEle) return;
        this.result = res;
        this.showPanel = true;
        if (res.length > 0) {
          this.$nextTick(() => {
            this.showResult = true;
            this.noResult = false;
          });
        } else {
          if (this.showNoResult) {
            this.$nextTick(() => {
              this.noResult = true;
              this.showResult = false;
              this.nowIndex = -1;
            });
          } else {
            this.$nextTick(() => {
              this.showPanel = false;
            });
          }
        }
      },
      getBoundingClientRect($el) {
        let style = $el.style;
        if (style.display === "none") { //display为none的元素没有物理尺寸，所以采用jQuery的方法，先将其脱离文档流设为隐藏，获得尺寸后还原
          let _addCss = {
            position: "absolute",
            visibility: "hidden",
            display: ""
          };
          let _oldCss = {};
          for (let i in _addCss) {
            _oldCss[i] = style[i];
            style[i] = _addCss[i];
          }
          let clientRect = $el.getBoundingClientRect();
          for (let i in _oldCss) {
            style[i] = _oldCss[i];
          }
          return clientRect;
        }
        return $el.getBoundingClientRect();
      },
      focus() {
        this.$refs.search_panel_input.focus();
      },
      select() {
        this.$refs.search_panel_input.select();
      },
      blur() {
        this.$refs.search_panel_input.blur();
      },
      _focusFirstInput(id) {
        let table_wrap = document.getElementById(id);
        let rows = table_wrap.querySelectorAll(".el-table__body-wrapper tbody tr");
        if (rows.length <= 0) return;
        let lastRow = rows[rows.length - 1];
        let rowInputs = lastRow.querySelectorAll("input");
        for (let i = 0; i < rowInputs.length; i++) {
          if (!rowInputs[i].disabled) {
            let firstInput = rowInputs[i];
            firstInput.focus();
            firstInput.select();
            return;
          }
        }
      },
      hiddenPanel() {
        this.showPanel = false;
      },
      resetPanel() {
        this.selectFirstItem ? this.nowIndex = 0 : this.nowIndex = -1;
        this.showResult = false;
        this.noResult = false;
        this.result = [];
      }
    },
    watch: {
      showPanel: function (newValue) {
        let vm = this;
        if (newValue) {
          vm.$nextTick(() => {
            document.body.appendChild(this.$refs.bodyWrapper); //将组件挂载到body中去
            //let inputY = vm.getY();
            let x = vm.getBoundingClientRect(vm.$refs.search_panel_input_wrap);
            let inputY = x.top;
            let inputX = x.left;
            let inputH = vm.$refs.search_panel_input.offsetHeight;
            let panelH = vm.$refs.bodyWrapper.offsetHeight;
            let panelW = vm.$refs.bodyWrapper.offsetWidth;
            let docH = document.body.offsetHeight;
            let docW = document.body.offsetWidth;
            //高度不够,显示到上面
            if (docH - inputY - inputH < panelH + 10) {
              vm.$refs.bodyWrapper.style.top = (x.top - panelH) + "px";
            } else {
              vm.$refs.bodyWrapper.style.top = x.top + x.height + "px";
            }
            //判断宽度
            if (panelW >= docW) {
              vm.$refs.bodyWrapper.style.left = "0px";
            } else if (panelW >= (docW - inputX) && panelW < docW) {
              vm.$refs.bodyWrapper.style.left = (docW - panelW - 10) + "px";
            } else {
              vm.$refs.bodyWrapper.style.left = x.left + "px";
            }
          });
        } else {
          vm.nowIndex = -1;
        }
      },
      scrollCheckIndex: function () {
        this.scrollCheck = true;
      }
    },
    mounted() {
      document.addEventListener("mousedown", this.hiddenPanel);
      document.addEventListener("click", this.hiddenPanel);
    },
    beforeDestroy() {
      document.removeEventListener("mousedown", this.hiddenPanel);
      document.removeEventListener("click", this.hiddenPanel);
    }
  };

</script>

<style scoped>
  * {
    box-sizing: border-box;
  }

  .clinic_search_panel {
    position: relative
  }

  .clinic_search_panel_input_wrap {
    position: relative;
    font-size: 14px;
    display: inline-block;
    width: 100%;
    border-radius: 4px;
  }

  .clinic_search_panel_input {
    -webkit-appearance: none;
    background: #fff none;
    border-radius: 4px;
    border: 1px solid #d9d9d9;
    box-sizing: border-box;
    color: #606266;
    display: inline-block;
    font-size: 12px;
    height: 28px;
    line-height: 28px;
    outline: none;
    padding: 0 8px;
    transition: border-color .2s cubic-bezier(.645, .045, .355, 1);
    width: 100%;
  }

  .clinic_search_panel_input:focus {
    border: 1px solid #409EFF;
  }

  .clinic_search_panel_input_disabled {
    background-color: #f5f7fa !important;
    border-color: #E4E7ED !important;
    color: #C0C4CC !important;
    cursor: not-allowed !important;
  }

  ul {
    margin: 0;
    padding: 0;
  }

  li {
    list-style: none;
    cursor: pointer;
  }

  ul.clinic_search_panel_result_box {
    overflow: auto;
    position: absolute;
    padding-top: 28px;
    border: 1px solid #dddddd;
    background: #FFFFFF;
    z-index: 9999;
    -webkit-box-shadow: #666 0 0 20px;
    -moz-box-shadow: #666 0 0 20px;
    box-shadow: #666 0 0 20px;
  }

  li.clinic_search_panel_result_title {
    background: #e4e7ed;
    height: 28px;
    border-bottom: 1px solid #dddddd;
    position: absolute;
    top: 0;
    left: 0;
  }

  li.clinic_search_panel_result_title :last-child.clinic_search_panel_result_title_span {
    border: none;
  }

  li.clinic_search_panel_result_title .clinic_search_panel_result_title_span {
    display: inline-block;
    height: 100%;
    color: #000000;
    text-align: center;
    border-right: 1px solid #DDDDDD;
    line-height: 28px;
    font-weight: bold;
  }

  li.clinic_search_panel_result_item .clinic_search_panel_result_item_span {
    display: inline-block;
    height: 100%;
    line-height: 28px;
    text-align: left;
    border-right: 1px solid #DDDDDD;
    box-sizing: border-box;
    padding: 0 4px 0 4px;
  }

  li.clinic_search_panel_result_item :last-child.clinic_search_panel_result_item_span {
    border: none;
  }

  li.clinic_search_panel_result_item {
    height: 28px;
    background: #FFFFFF;
    color: #000000;
    /*border-bottom: 1px solid #dddddd;*/
  }

  li.clinic_search_panel_result_item.active, li.clinic_search_panel_result_item:hover {
    background-color: #3f73e0;
    opacity: 80%;
    color: #FFF;
  }

  .text-overflow {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .fade-enter-active, .fade-leave-active {
    transition: all .5s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
    -webkit-transform: translateY(20px);
    -moz-transform: translateY(20px);
    -ms-transform: translateY(20px);
    -o-transform: translateY(20px);
    transform: translateY(20px);
  }

  .flex {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
  }

  .flex_center_all {
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
  }

  .flex_center_ver {
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
  }

  .flex_1 {
    -webkit-box-flex: 1;
    -ms-flex: 1;
    flex: 1;
  }

  input::-webkit-input-placeholder {
    color: #BBBBBB;
  }

  input::-moz-placeholder { /* Mozilla Firefox 19+ */
    color: #BBBBBB;
  }

  input:-moz-placeholder { /* Mozilla Firefox 4 to 18 */
    color: #BBBBBB;
  }

  input:-ms-input-placeholder { /* Internet Explorer 10-11 */
    color: #BBBBBB;
  }
</style>
