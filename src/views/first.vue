<template>
  <div class="first">
    <div class="search">
      <el-form :inline="true" :model="searchData" size="small" class="demo-form-inline">
        <el-form-item label="县（村）名称">
          <el-input @keyup.enter.native="onSubmit" v-model="searchData.cname" placeholder="县（村）名称"></el-input>
        </el-form-item>
        <el-form-item label="乡（镇）名称">
          <el-input @keyup.enter.native="onSubmit" v-model="searchData.townshtp" placeholder="乡（镇）名称"></el-input>
        </el-form-item>
        <el-form-item label="县名称">
          <el-input @keyup.enter.native="onSubmit" v-model="searchData.county" placeholder="县名称"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">查询</el-button>
          <el-button @click="export2Excel" type="success">导出Excel</el-button>
        </el-form-item>
      </el-form>
    </div>
    <el-table
      :data="dataList.slice((currentPage-1)*pageSize,currentPage*pageSize)"
      :height="tableHeight"
      border
      stripe
      style="width: 100%"
      v-loading="loading"
      id="rebateSetTable1"
      >
      <!-- <el-table-column
        type="selection"
        width="55">
      </el-table-column> -->
      <el-table-column
        label="序号"
        fixed
        width="80" 
        align="center">
        <template slot-scope="scope"><span>{{scope.$index+(currentPage - 1) * pageSize + 1}} </span></template>
      </el-table-column>
      <el-table-column
        prop="province"
        label="省名称"
        width="180">
      </el-table-column>
      <el-table-column
        prop="city"
        label="市名称"
        width="180"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        prop="county"
        label="县名称"
        width="180"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        prop="townshtp"
        label="乡（镇）名称"
        width="180"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        prop="village"
        label="村名称"
        width="180"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        prop="ida"
        label="县（村）编号IDA"
        width="180">
        <template slot-scope="scope">
          <span>{{ scope.row.ida }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="cname"
        label="县（村）名称"
        min-width="180"
        show-overflow-tooltip>
      </el-table-column>
    </el-table>
    <el-pagination
      style="text-align:center;margin-top:30px;margin-left:-50px"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="pageSizes"
      :page-size="pageSize"
      prev-text="上一页"
      next-text="下一页"
      layout="total, sizes, prev, pager, next, jumper"
      :total="totalCount">
    </el-pagination>
  </div>
</template>

<script>
import qs from 'qs'
// 导出excel
require('script-loader!file-saver');                    //保存文件用
require('script-loader!vendor/Blob');                   //转二进制用
require('script-loader!xlsx/dist/xlsx.core.min');       //xlsx核心

const axios = require('axios');
export default {
  name: 'first',
  watch: {
      
  },
  data () {
    return {
      tableHeight: window.innerHeight - 200,    // 表格高度自适应
      dataList:[],   // 总数据
      searchData: {   // 搜索
        cname:'',
        townshtp:'',
        county:''
      },
      currentPage: 1,  // 当前页数
      pageSize:10,   // 每页数
      totalCount:200,  // 总条数
      pageSizes:[10, 20, 50, 100],  // 页数
      loading: true,    // 懒加载
    }
  },
  methods: {

    // 获取表格数据
    getTableData() {
      var self = this;
      axios.get('xjbxxTbMapper')
      .then(function (response) {
        self.dataList = response.data.data;  // 数据dataList
        
        self.totalCount = response.data.data.length  // 将数据的长度赋值给totalCount
        
        // 关闭loading
        self.loading = false;
      })
      .catch(function (error) {
        
      });
    },

    // 查询
    onSubmit() {
      // 判断查询条件是否为空
      if (this.searchData.cname == "" && this.searchData.townshtp == "" && this.searchData.county == "") {
        this.$message({
          message: '查询条件为空显示全部数据！',
          type: 'warning',
          showClose: true,
          duration:2000
        });
      }
      // 向后台发送请求获取查询结果数据
      let formData = {
        "cname":this.searchData.cname,
        "townshtp":this.searchData.townshtp,
        "county":this.searchData.county,
      };
      var self = this;
      axios.post('xjbxxTbMapper',qs.stringify(formData))
      .then(res=>{
        self.dataList = res.data.data;  // 数据dataList
        self.totalCount = res.data.data.length  // 将数据的长度赋值给totalCount        
      })
    },

    // 分页 pageSize 改变时会触发
    handleSizeChange(val) {
      // 改变每页显示的条数 
      this.pageSize=val
      // 注意：在改变每页显示的条数时，要将页码显示到第一页
      this.currentPage=1
    },
    // 分页 currentPage 改变时会触发
    handleCurrentChange(val) {
      // 改变默认的页数
      this.currentPage=val
    },

    // 导出excel
    export2Excel() {
　　　　require.ensure([], () => {
　　　　　　const { export_json_to_excel } = require('../vendor/Export2Excel');  // 新建文件夹下的Export2Excel.js文件
　　　　　　const tHeader = ['省名称','市名称','县名称','镇名称','村名称','县（村）编号IDA', '县（村）名称'];            // 对应表格输出的title
　　　　　　const filterVal = ['province','city','county','townshtp','village','ida', 'cname'];                             // 对应表格输出的数据
　　　　　　const list = this.dataList;
　　　　　　const data = this.formatJson(filterVal, list);
　　　　　　export_json_to_excel(tHeader, data, '县（村）基本信息Excel表');                   // 对应下载excel文件的名字
　　　　})
　　},
　　formatJson(filterVal, jsonData) {
　　　　return jsonData.map(v => filterVal.map(j => v[j]))
　　}
  },
  mounted(){
    this.getTableData();
  }
}
</script>

<style>
  .app-search .search{
    width: 100%;
    height: 60px;
    border-radius: 5px;
  }
</style>
