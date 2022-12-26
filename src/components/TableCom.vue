<template>
  <div class="table-container">
    <!-- 解析 HTML 标签 -->
    <div v-html="tableVal" class="wrap"></div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'TableCom',
  props: {
    msg: String,
    params: Object
  },
  data() {
    // 定义变量
    return {
      tableVal: ''
    }
  },
  // mounted() {
  //   // 调用初始化函数
  //   this.loadData()
  // },
  watch: {
    // 当传递过来的参数发生改变时，重新调用函数，获取最新的数据
    params(newVal) {
      const curParams = JSON.parse(JSON.stringify(newVal))
      this.loadData(curParams)
    },
  },
  methods: {
    async loadData(params) {
      // 把这个地址替换为正式访问接口，现在是放在本地 public 中
      await axios({
        method: "get",
        url: "http://localhost:8080/data.json",
        data: params
      }).then(res => {
        // console.log(res);
        // 此处如果是封装后的 axios ，应该 res.data 就可以获取到数据，未封装的 axios ，需要 res.data.data
        // if (res.status === 200) {
        this.tableVal = res?.data?.data
        // }
      })
    }
  }
}
</script>

<style>
.table-container {
  width: 700px;
  display: inline-block;
  margin-left: 40px;
  /* margin-top: 180px; */
}

.wrap {
  font-size: 14px;
  text-align: center;
  width: 700px;
  /* float: left; */
  display: inline-block;
  border: 1px solid #ebebeb;
  /* height: 500px; */
  /* overflow: auto; */
}

.wrap .dataframe {
  width: 100%;
  border: none;
  border-collapse: collapse;
}

.wrap th {
  height: 45px;
  border: none;
  padding: 0px;
  border-bottom: 1px solid #ebeef5;
}

.wrap tr:hover {
  background-color: rgb(245, 247, 250);
  transition: background-color .25s ease;
}

.wrap td {
  border: none;
  border-bottom: 1px solid #ebeef5;
}
</style>
