<template>
  <el-card class="contentmaster" v-loading="loading">
    <breadcrumb slot="header">
      <template slot="title">内容管理</template>
    </breadcrumb>
    <el-form style="margin-left:50px;">
      <el-form-item label="文章状态：">
        <el-radio-group v-model="searchForm.radio" @change="getChange">
          <el-radio :label="5">全部</el-radio>
          <el-radio :label="0">草稿</el-radio>
          <el-radio :label="1">待审核</el-radio>
          <el-radio :label="2">审核通过</el-radio>
          <el-radio :label="3">审核失败</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="频道列表：">
        <el-select v-model="searchForm.channel" placeholder="请选择频道" @change="getChange">
          <el-option v-for="item in channels" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="时间选择：">
        <el-date-picker
          v-model="searchForm.dateRange"
          type="daterange"
          @change="getChange"
          value-format="yyyy-MM-dd"
        ></el-date-picker>
      </el-form-item>
    </el-form>
    <el-row type="flex" align="middle" class="finddata">
      <p>共找到{{page.total}}条数据</p>
    </el-row>
    <div class="contentlist" v-for="item in contentList" :key="item.id.toString()">
      <div class="left">
        <img :src="item.cover.images.length?item.cover.images[0]:defaultImg" alt />
        <div class="info">
          <span>{{item.title}}</span>
          <el-tag :type="item.status | filterButton" class="status">{{item.status | filterStatus}}</el-tag>
          <span class="date">{{item.pubdate}}</span>
        </div>
      </div>
      <div class="right">
        <span @click="editContent(item.id.toString())">
          <i class="el-icon-edit"></i>修改
        </span>
        <span @click="delContent(item.id.toString())">
          <i class="el-icon-delete"></i>删除
        </span>
      </div>
    </div>
    <el-row type="flex" justify="center" align="middle" style="height:60px;">
      <el-pagination background layout="prev, pager, next"
        @current-change="changePage"
        :current-page="page.currentPage"
        :page-size="page.pageSize"
        :total="page.total"
      ></el-pagination>
    </el-row>
  </el-card>
</template>

<script>
import { delContent, getContentList, getChannel } from '../../actions/articles'
export default {
  data () {
    return {
      // 搜索取数据
      searchForm: {
        radio: 5,
        channel: null,
        dateRange: []
      },
      // 页码参数
      page: {
        currentPage: 1,
        pageSize: 10,
        total: 0
      },
      // 内容列表数据
      contentList: [],
      // 频道数据
      channels: [],
      loading: false,
      defaultImg: require('../../assets/img/back.jpeg')
    }
  },
  methods: {
    // 修改内容
    editContent (id) {
      this.$router.push(`/home/publish/${id}`)
    },
    // 删除内容列表
    async delContent (id) {
      try {
        await this.$confirm('确定真的要删除这条内容么')
        await delContent(id)
        this.$message({
          message: '删除成功',
          type: 'success'
        })
        this.getContentList()
      } catch (error) {

      }
    },
    // 搜索区域发生改变
    getChange () {
      this.page.currentPage = 1
      this.getChangeContent()
    },
    // 页码点击
    changePage (newPage) {
      this.page.currentPage = newPage
      this.getChangeContent()
    },
    // 获取变化条件页码后的内容列表
    getChangeContent () {
      let params = {
        page: this.page.currentPage,
        per_page: this.page.pageSize,
        status: this.searchForm.radio === 5 ? null : this.searchForm.radio,
        channel_id: this.searchForm.channel,
        begin_pubdate: this.searchForm.dateRange.length ? this.searchForm.dateRange[0] : null,
        end_pubdate: this.searchForm.dateRange.length > 1 ? this.searchForm.dateRange[1] : null
      }
      this.getContentList(params)
    },
    // 获取内容列表
    async getContentList (params) {
      this.loading = true
      let result = await getContentList(params)
      this.contentList = result.data.results
      this.page.total = result.data.total_count
      this.loading = false
    },
    // 获取频道列表
    async getChannel () {
      let result = await getChannel()
      this.channels = result.data.channels
    }
  },
  filters: {
    // 过滤文章发表状态按钮
    filterButton (value) {
      switch (value) {
        case 0:
          return 'info'
        case 1:
          return 'warning'
        case 2:
          return 'success'
        case 3:
          return 'danger'
        default:
          break
      }
    },
    // 过滤文章发表状态
    filterStatus (value) {
      switch (value) {
        case 0:
          return '草稿'
        case 1:
          return '待审核'
        case 2:
          return '已发布'
        case 3:
          return '审核失败'
        default:
          break
      }
    }
  },
  created () {
    this.getChannel()
    // 第一次请求文章列表数据不需要传递params参数
    this.getContentList({ page: 1, per_page: 10 })
  }
}
</script>

<style lang="less" scoped>
.contentmaster {
  .finddata {
    height: 60px;
    border-bottom: 1px dashed #cccccc;
  }
  .contentlist {
    padding: 20px 0;
    display: flex;
    justify-content: space-between;
    border-bottom: 1px solid #f2f3f5;
    .left {
      display: flex;
      img {
        width: 160px;
        height: 120px;
      }
      .info {
        margin-left: 10px;
        height: 120px;
        display: flex;
        flex-direction: column;
        justify-content: space-around;
        .status {
          width: 60px;
          text-align: center;
        }
        .date {
          font-size: 12px;
          color: #999;
        }
      }
    }
    .right {
      display: flex;
      span {
        font-size: 14px;
        margin-right: 8px;
        cursor: pointer;
      }
    }
  }
}
</style>
