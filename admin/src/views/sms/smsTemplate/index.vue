<template>
  <!--v-if="isLogin"-->
  <div class="divBox" v-if="isLogin">
    <el-card v-loading="fullscreenLoading" class="box-card">
      <div slot="header" class="clearfix">
        <div class="container">
          <el-form :inline="true">
            <el-form-item label="模板状态：" class="mr10">
              <el-select v-model="tableFrom.status" placeholder="请选择" clearable class="filter-item selWidth mr20" @change="userSearchs">
                <el-option value="1" label="可用"></el-option>
                <el-option value="0" label="不可用"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="模板类型：" class="mr10">
              <el-select v-model="tableFrom.type" placeholder="请选择" clearable class="filter-item selWidth mr20" @change="userSearchs">
                <el-option value="1" label="验证码"></el-option>
                <el-option value="2" label="通知"></el-option>
                <el-option value="3" label="推广"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="模板名称：" class="mr10">
              <el-input v-model="tableFrom.title" placeholder="请输入模板名称" class="selWidth">
                <el-button slot="append" icon="el-icon-search" @click="userSearchs" />
              </el-input>
            </el-form-item>
          </el-form>
        </div>
        <el-button size="mini" type="primary" @click="add">添加短信模板</el-button>
      </div>
      <el-table
        v-loading="listLoading"
        :data="tableData.data"
        style="width: 100%"
        size="mini"
        highlight-current-row
      >
        <el-table-column
          prop="id"
          label="ID"
          min-width="50"
        />
        <el-table-column
          prop="templateid"
          label="模板ID"
          min-width="80"
        />
        <el-table-column
          prop="title"
          label="模板名称"
          min-width="120"
        />
        <el-table-column
          prop="content"
          label="模板内容"
          min-width="500"
        />
        <el-table-column
          label="模板类型"
          min-width="100"
        >
          <template slot-scope="{row}">
            <span>{{ row.type | typesFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column label="模板状态">
          <template slot-scope="{row}">
            <span>{{ row.status | statusFilter }}</span>
          </template>
        </el-table-column>
        <el-table-column
          prop="add_time"
          label="添加时间"
          min-width="150"
        />
      </el-table>
      <div class="block">
        <el-pagination
          :page-sizes="[20, 40, 60, 80]"
          :page-size="tableFrom.limit"
          :current-page="tableFrom.page"
          layout="total, sizes, prev, pager, next, jumper"
          :total="tableData.total"
          @size-change="handleSizeChange"
          @current-change="pageChange"
        />
      </div>
    </el-card>

    <!--编辑-->
    <el-dialog
      title="添加模板"
      :visible.sync="dialogVisible"
      width="500px"
      :before-close="handleClose">
      <zb-parser
        v-if="dialogVisible"
        :form-id="110"
        :is-create="isCreate"
        :edit-data="editData"
        @submit="handlerSubmit"
      />
    </el-dialog>
  </div>
</template>

<script>
import { smsTempLstApi, tempCreateApi } from '@/api/sms'
import { roterPre } from '@/settings'
import { mapGetters } from 'vuex'
import zbParser from '@/components/FormGenerator/components/parser/ZBParser'
export default {
  name: 'SmsTemplate',
  components: { zbParser },
  filters: {
    statusFilter(status) {
      const statusMap = {
        0: '不可用',
        1: '可用'
      }
      return statusMap[status]
    },
    typesFilter(status) {
      const statusMap = {
        1: '验证码',
        2: '通知',
        3: '推广'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      isCreate: 0,
      editData: {},
      dialogVisible: false,
      fullscreenLoading: false,
      listLoading: false,
      tableData: {
        data: [],
        total: 0
      },
      tableFrom: {
        page: 1,
        limit: 20,
        status: '',
        title: '',
        type: ''
      }
    }
  },
  computed: {
    ...mapGetters([
      'isLogin'
    ])
  },
  mounted() {
    if (!this.isLogin) {
      this.$router.push('/operation/systemSms/config?url=' + this.$route.path)
    } else {
      this.getList()
    }
  },
  methods: {
    handleClose() {
      this.dialogVisible = false
      this.editData = {}
    },
    handlerSubmit(formValue) {
      tempCreateApi(formValue).then(data => {
        this.$message.success('新增成功')
        this.dialogVisible = false
        this.editData = {}
        this.getList()
      })
    },
    add() {
      this.dialogVisible = true
    },
    // 查看是否登录
    onIsLogin() {
      this.fullscreenLoading = true
      this.$store.dispatch('user/isLogin').then(async res => {
        const data = res
        if (!data.status) {
          this.$message.warning('请先登录')
          this.$router.push( '/operation/systemSms/config?url=' + this.$route.path)
        } else {
          this.getList()
        }
        this.fullscreenLoading = false
      }).catch(res => {
        this.$router.push( '/operation/systemSms/config?url=' + this.$route.path)
        this.fullscreenLoading = false
      })
    },
    // 列表
    getList() {
      this.listLoading = true
      smsTempLstApi(this.tableFrom).then(res => {
        this.tableData.data = res.data
        this.tableData.total = res.count
        this.listLoading = false
      }).catch(res => {
        this.listLoading = false
      })
    },
    pageChange(page) {
      this.tableFrom.page = page
      this.getList()
    },
    handleSizeChange(val) {
      this.tableFrom.limit = val
      this.getList()
    },
    // 表格搜索
    userSearchs() {
      this.tableFrom.page = 1
      this.getList()
    }
  }
}
</script>

<style scoped lang="scss">
  .selWidth{
    width: 350px !important;
  }
</style>
