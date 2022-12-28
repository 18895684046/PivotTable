<!-- 先安装 npm i vuedraggable -S 拖拽的库 -->
<template>
  <div id="app">
    <div style="float: left">
      <div v-for="item in arr1" :key="item.id" class="single-filter">
        <span>{{ item.name }}</span>
        <el-popover :popper-options="{
          boundariesElement: 'viewport',
          removeOnDestroy: true,
        }" placement="bottom" width="400" trigger="click">
          <el-checkbox v-model="item.checkAll" @change="(val) => handleCheckAllChange(val, item.name)">全选</el-checkbox>
          <div style="margin: 15px 0"></div>
          <el-checkbox-group v-model="item.checkedCities" @change="(val) => handleCheckedCitiesChange(val, item.name)">
            <!-- 【计数】字段数据量过多，暂时截取前200条数据，后续根据实际修改 -->
            <el-checkbox v-for="city in originData[item.name].length > 200
            ? originData[item.name].slice(0, 200)
            : originData[item.name]" :label="city" :key="city">{{ city }}</el-checkbox>
          </el-checkbox-group>
          <i slot="reference" style="float: right; cursor: pointer" class="el-icon-coin"></i>
          <!-- <el-button size="small" type="primary">确认</el-button>
          <el-button size="small">取消</el-button> -->
        </el-popover>
      </div>
    </div>

    <!-- 渲染 TableCom 组件 -->
    <TableCom :params="params" />

    <!--使用draggable组件-->
    <div class="itxst">
      <div class="col">
        <div class="title">筛选器</div>
        <draggable @start="onStart" v-model="arr1" @add="(e) => dragAdd(e, '1')" @sort="onPointUpdate" :group="groupA"
          animation="200" dragClass="dragClass" ghostClass="ghostClass" chosenClass="chosenClass">
          <transition-group :style="style">
            <div class="item" v-for="item in arr1" :key="item.id">
              {{ item.name }}
            </div>
          </transition-group>
        </draggable>
      </div>
      <div class="col">
        <div class="title">行区域</div>
        <draggable v-model="arr2" :group="groupB" @add="(e) => dragAdd(e, '2')" @sort="onPointUpdate" animation="200"
          dragClass="dragClass" ghostClass="ghostClass" chosenClass="chosenClass">
          <transition-group :style="style">
            <div class="item" v-for="item in arr2" :key="item.id">
              {{ item.name }}
            </div>
          </transition-group>
        </draggable>
      </div>
      <div class="col">
        <div class="title">列区域</div>
        <draggable v-model="arr3" :group="groupC" @add="(e) => dragAdd(e, '3')" @sort="onPointUpdate" animation="200"
          dragClass="dragClass" ghostClass="ghostClass" chosenClass="chosenClass">
          <transition-group :style="style">
            <div class="item" v-for="item in arr3" :key="item.id">
              {{ item.name }}
            </div>
          </transition-group>
        </draggable>
      </div>
      <div class="col">
        <div class="title">求值区域</div>
        <draggable v-model="arr4" :group="groupD" @add="(e) => dragAdd(e, '4')" @sort="onPointUpdate" animation="300"
          dragClass="dragClass" ghostClass="ghostClass" chosenClass="chosenClass">
          <transition-group :style="style">
            <div class="item" v-for="item in arr4" :key="item.id">
              {{ item.name }}
            </div>
          </transition-group>
        </draggable>
      </div>
      <div class="fields-wrap">
        <div class="title">字段列表</div>
        <el-checkbox-group v-model="filedCheckList" @change="changeChckBox">
          <draggable v-model="allFiledList" :group="groupE" @add="dragAdd" @sort="onPointUpdate" animation="300"
            dragClass="dragClass" ghostClass="ghostClass" chosenClass="chosenClass">
            <transition-group :style="style">
              <div class="item" v-for="item in allFiledList" :key="item.id">
                <el-checkbox :label="item.name"></el-checkbox>
              </div>
            </transition-group>
          </draggable>
        </el-checkbox-group>
      </div>
    </div>
    <!-- 添加字段列表区域 -->
    <!-- <div class="fields-wrap">
      <h5>字段列表</h5>
      <el-checkbox-group v-model="filedCheckList">
        <el-checkbox v-for="(item, index) in allFiledList" :key="item + index" :label="item"></el-checkbox>
      </el-checkbox-group>
    </div> -->
  </div>
</template>
<script>
import TableCom from "@/components/TableCom.vue";
//导入draggable组件
import draggable from "vuedraggable";
import axios from "axios";
export default {
  name: "HomeView",
  //注册draggable组件
  components: {
    TableCom,
    draggable,
  },
  data() {
    return {
      point: false,
      allFiledList: [],
      filedCheckList: [],
      originData: [],
      checkAll: false,
      checkedCities: [],
      isIndeterminate: false,
      params: {},
      drag: false,
      groupA: {
        name: "itxst",
        put: true, //可以拖入
      },
      groupB: {
        name: "itxst",
        put: true,
      },
      groupC: {
        name: "itxst",
        put: true,
      },
      groupD: {
        name: "itxst",
        put: true,
      },
      groupE: {
        name: "itxst",
        pull: "clone",
        put: false,
      },
      //定义要被拖拽对象的数组
      arr1: [],
      arr2: [],
      arr3: [],
      arr4: [],
      arr5: [],
      //空数组之在的样式，设置了这个样式才能拖入
      style: "min-height:120px;display: block;",
      valueList: [],
      lastCheckArr: [], //用于记录每次勾选的旧值的数组
    };
  },
  // 初始化执行
  mounted() {
    this.loadMetaData();
  },
  // 计算属性，用来同时监听多个数据的改变
  computed: {
    // 监听所有需要变化的数据
    listenChange() {
      const { arr1, arr2, arr3, arr4 } = this;
      return { arr1, arr2, arr3, arr4 };
    },
    // 处理名称显示
    filterData() {
      return this.arr1?.map((it) => it?.name);
    },
  },

  // 监听多个数据的改变，然后根据条件调取接口，更新页面
  watch: {
    // point 只需要监听位置的改变,这样左侧筛选器监听不到数据改变，需要结合实际项目来做
    listenChange() {
      // 在这里拼接需要传递给后端的参数, 获取到最新的数据后传递给 TableCom 组件
      const filterNames = this.formatData(this.arr1)?.map((it) => it?.name); // 筛选器名称集合
      const aggNames = this.formatData(this.arr4)?.map((it) => it?.name); // 求值区域名称集合
      const index = this.formatData(this.arr2)?.map((it) => it?.name); // 行参数(名称集合)
      const columns = this.formatData(this.arr3)?.map((it) => it?.name); // 列参数(名称集合)

      // 当出现重复字段时，阻止后续执行
      const concatNames = [...filterNames, ...aggNames, ...index, ...columns]
      if (!this.noRepeat(concatNames)) {
        return
      }
      const handleFilter = this.formatData(this.arr1)?.map((it) => {
        return {
          [it.name]: it.checkedCities,
        };
      });
      const filters = Object.assign({}, ...handleFilter); // 过滤器参数
      const aggs = this.formatData(this.arr4)?.map((it) => {
        return {
          [it.name]: "sum",
        };
      });
      const aggfunc = Object.assign({}, ...aggs); // 求值的参数
      // 拼接出的最终参数
      const bodyParams = {
        index,
        columns,
        aggfunc,
        filters,
      };
      // console.log(index, columns, filterNames, aggNames);
      // console.log(bodyParams, "bodyParams--拼接出的参数");
      this.params = JSON.parse(JSON.stringify(bodyParams));
    },
    // 监听字段列表是否被选中，暂时这样截取默认显示数据
    // filedCheckList(newVal) {
    //   this.arr1 = this.handleDragField(newVal, 0, 4);
    //   this.arr2 = this.handleDragField(newVal, 4, 6);
    //   this.arr3 = this.handleDragField(newVal, 6, 7);
    //   this.arr4 = this.handleDragField(newVal, 7, 8);
    // },
  },
  methods: {
    // 判断字符串中是否出现重复的字符
    noRepeat(str) {
      if (str.length === new Set(str).size) {
        return true
      }
      return false
    },
    //选中checkbox
    changeChckBox(e) {
      //判断是否选中
      let isChecked =
        this.formatData(e).length > this.formatData(this.lastCheckArr).length;
      // 选中状态
      if (isChecked) {
        //找差集
        let value = e.filter((item) => !this.lastCheckArr?.includes(item));
        let RedomNumber = Math.floor(Math.random() * 3 + 1); // 随机数 模拟智能推荐
        let tmpObj = {
          id: Math.random() * 6,
          name: value?.[0],
          checkedCities: [],
          checkAll: false,
        };

        // 赋值
        this[`arr${RedomNumber}`] = [...this[`arr${RedomNumber}`], tmpObj];
      } else {
        // 取消选中状态 //找到当前项

        //找差集
        let value = this.formatData(this.lastCheckArr).filter((item) => !this.formatData(e)?.includes(item));
        // 去重
        value = [...new Set([...value])];
        // console.log('value', value);
        // 删除筛选项存在的值
        [1, 2, 3, 4].forEach((index) => {
          for (let i in this[`arr${index}`]) {
            let tmpArr = this.formatData(this[`arr${index}`]);
            if (this[`arr${index}`][i].name == value) {
              tmpArr.splice(i, 1);
              this[`arr${index}`] = tmpArr;
              break;
            }
          }
        });
      }

      // 记录上一次值
      this.lastCheckArr = e;
    },
    // 格式化获取的数据
    formatData(data) {
      return JSON.parse(JSON.stringify(data));
    },
    // 处理 拖拽字段
    handleDragField(fieldList, sta, end) {
      const dragData = fieldList?.slice(sta, end)?.map((it) => {
        return {
          id: Math.random() * 6,
          name: it,
          checkedCities: [],
          checkAll: false,
        };
      });
      return dragData;
    },
    // 获取初始化的数据
    async loadMetaData() {
      // 把这个地址替换为正式访问接口，获取 meta.json 对应的数据，现在是放在本地 public 中
      await axios.get("http://localhost:8080/meta.json").then((res) => {
        // console.log(res);
        if (res.status === 200) {
          this.originData = res?.data?.data;
          // 获取字段名列表
          const fieldList = Object.keys(res?.data?.data);
          const cloneList = JSON.parse(JSON.stringify(fieldList));
          this.allFiledList = cloneList.map((it) => {
            return {
              id: Math.random() * 6,
              name: it,
              checkedCities: [],
              checkAll: false,
            };
          });
          // this.allFiledList = cloneList
          // console.log("&&", this.allFiledList);
          // 筛选器字段，暂时这么处理，根据实际情况改变
          // this.arr1 = this.handleDragField(fieldList, 0, 4)
          // this.arr2 = this.handleDragField(fieldList, 0, 2)
          // this.arr3 = this.handleDragField(fieldList, 0, 1)
          // this.arr4 = this.handleDragField(fieldList, 0, 1)
        }
      });
    },
    //开始拖拽事件
    onStart() {
      this.drag = true;
    },
    //  拖拽的 add 事件
    dragAdd(e, name) {
      let value = e?.item?.innerText?.split(" ")?.[0]; // 获取当前拖拽的字段
      // console.log(this.formatData(this[`arr${name}`]).filter(item => item.name == value)?.length, '@@@@@@@', value, this.formatData(this[`arr${name}`]));
      //相同目标区域，重复拖拽，不执行
      if (this.formatData(this[`arr${name}`]).filter(item => item.name == value)?.length > 1) {
        let arr = this.formatData(this[`arr${name}`]);
        let res = [...new Set(arr.map(v => JSON.stringify(v)))].map(s => JSON.parse(s))
        this[`arr${name}`] = res;
        return false;
      }

      //拖拽的时候，将目标区域以外的筛选数组，包含的name项删掉

      let filterIndex = ["1", "2", "3", "4"].filter((item) => item != name);
      filterIndex.forEach((index) => {
        let copyArr = JSON.parse(JSON.stringify(this[`arr${index}`]));
        copyArr.forEach((item, key) => {
          if (item.name == value) {
            copyArr.splice(key, 1);
          }
        });
        this.$nextTick(() => {
          this[`arr${index}`] = copyArr;
        });
      });
      // console.log('copyArr', this.filedCheckList)

      this.filedCheckList = [...new Set([...this.formatData(this.filedCheckList), value])];

      // 记录上一次的checkbox
      this.lastCheckArr = [...this.lastCheckArr, value];
    },
    //拖拽位置发生变化事件
    onPointUpdate() {
      this.point = !this.point;
    },
    // 全选按钮事件，更新选中的数据
    handleCheckAllChange(val, name) {
      const cloneData = JSON.parse(JSON.stringify(this.arr1));
      cloneData.forEach((it) => {
        if (it.name === name) {
          if (val) {
            it.checkedCities = this.originData[name];
          } else {
            it.checkedCities = [];
          }
        }
      });
      this.arr1 = cloneData;
      this.isIndeterminate = false;
    },
    // 选择 checkbox 组的事件
    handleCheckedCitiesChange(val, name) {
      const cloneData = JSON.parse(JSON.stringify(this.arr1));
      let checkedCount = val.length;
      cloneData.forEach((it) => {
        if (it.name === name) {
          it.checkAll = checkedCount === this.originData[name].length;
        }
      });
      this.arr1 = cloneData;
    },
  },
};
</script>
<!-- 这里另外写一个 去掉 scoped  的 style 标签，是为了让覆盖 element ui 的样式生效，-->
<!-- 也可以写在全局样式中，注意 要区分好 类名 -->
<style>
.el-popper {
  max-height: 500px;
  overflow: auto;
}

/*里面的代码可以根据自己需求去进行更改*/
/* 设置滚动条的样式 */
.el-popper::-webkit-scrollbar {
  width: 6px;
}

/* 滚动槽 */
.el-popper::-webkit-scrollbar-track {
  -webkit-box-shadow: inset006pxrgba(0, 0, 0, 0.3);
  border-radius: 3px;
}

/* 滚动条滑块 */
.el-popper::-webkit-scrollbar-thumb {
  border-radius: 5px;
  background: rgba(0, 0, 0, 0.1);
  -webkit-box-shadow: inset006pxrgba(0, 0, 0, 0.5);
}
</style>
<style scoped lang="scss">
/*定义要拖拽元素的样式*/
.ghostClass {
  font-size: 14px;
  background-color: blue !important;
}

.chosenClass {
  background-color: rgb(159, 205, 179) !important;
  opacity: 1 !important;
}

.dragClass {
  background-color: rgb(159, 205, 179) !important;
  opacity: 1 !important;
  box-shadow: none !important;
  outline: none !important;
  background-image: none !important;
}

.itxst {
  position: relative;
  margin-right: 10px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  min-height: 200px;
  width: 250px;
  float: right;
}

.title {
  padding-left: 12px;
  height: 30px;
  line-height: 30px;
  background-color: #ddd;
}

.col {
  flex: 1;
  border: solid 1px #eee;
  border-radius: 5px;
  float: left;
  margin-bottom: 10px;
  border: 1px solid gray;
}

.item {
  padding: 6px 12px;
  border: solid 1px #eee;
  background-color: #f1f1f1;
  font-size: 14px;
}

.item:hover {
  background-color: #fdfdfd;
  cursor: move;
}

.item+.item {
  border-top: none;
  margin-top: 6px;
}

.single-filter {
  font-size: 14px;
  padding: 8px;
  position: relative;
  border: 1px solid #ddd;
  width: 150px;
  padding: 10px;
  margin: 10px;
}

.fields-wrap {
  position: absolute;
  width: 100px;
  height: 200px;
  left: -120px;
  float: right;
  margin-right: 50px;

  .el-checkbox {
    height: 35px;
    line-height: 35px;
  }
}
</style>
