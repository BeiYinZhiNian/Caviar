<template>
  <div>
    <el-form ref="form" :model="selectData" inline>
      <el-form-item label="账号名称">
        <el-input v-model="selectData.name" maxlength="20" placeholder="请输入账号名称" show-word-limit />
      </el-form-item>
      <el-form-item label="联系方式">
        <el-input v-model="selectData.phoneNumber" placeholder="请输入手机号/邮箱" maxlength="100" show-word-limit />
      </el-form-item>
      <el-form-item label="角色">
        <el-select
          v-model="selectData.roleIds"
          multiple
          collapse-tags
          placeholder="请选择"
        >
          <el-option
            v-for="item in rolesOption"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="success" @click="loadData()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-permission="['SystemSettings_Users_Create']" type="primary" @click="$refs.formData.setFormData()">创建</el-button>
      </el-form-item>
    </el-form>
    <div>
      <el-table
        :data="tableData.items"
        stripe
        border
        :header-cell-style="()=>{return 'text-align:center;background:#e1e4e5;color:#80878f'}"
        :cell-style="()=>{return 'text-align:center'}"
        style="width: 100%"
      >
        <el-table-column label="序号" type="index" width="80" align="center" />
        <el-table-column :show-overflow-tooltip="true" prop="name" label="账号名称" />
        <el-table-column prop="isActive" label="是否启用">
          <template slot-scope="scope">
            <table-bool :flag="scope.row.isActive" />
          </template>
        </el-table-column>
        <el-table-column :show-overflow-tooltip="true" prop="phoneNumber" label="手机号" />
        <el-table-column :show-overflow-tooltip="true" prop="emailAddress" label="邮箱地址" />
        <el-table-column :show-overflow-tooltip="true" prop="creationTime" label="创建时间" />
        <el-table-column :show-overflow-tooltip="true" prop="roleNames" label="角色">
          <template slot-scope="scope">
            <span>{{ scope.row.roleNames.join('、') }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="checkPermission(['SystemSettings_Users_ResetPassword','SystemSettings_Users_Delete','SystemSettings_Users_Edit'])" :show-overflow-tooltip="true" label="操作" width="200">
          <template slot-scope="scope">
            <el-button
              v-permission="['SystemSettings_Users_Edit']"
              size="mini"
              type="primary"
              @click="$refs.formData.setFormData(scope.row)"
            >编辑</el-button>
            <el-dropdown v-if="checkPermission(['SystemSettings_Users_ResetPassword','SystemSettings_Users_Delete'])" style="margin-left: 10px;">
              <el-button type="success" size="mini">
                更多<i class="el-icon-arrow-down el-icon--right" />
              </el-button>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item v-permission="['SystemSettings_Users_ResetPassword']" @click.native="$refs.resetPasswordDto.setFormData(scope.row)">重置密码</el-dropdown-item>
                <el-dropdown-item v-permission="['SystemSettings_Users_Delete']" @click.native="handleDelete(scope.row)">删除</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div>
      <el-pagination
        background
        layout="total, sizes, prev, pager, next, jumper"
        :current-page.sync="selectData.skipCount"
        :total="tableData.totalCount"
        :page-size="selectData.maxResultCount"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </div>
    <formData ref="formData" :success="loadData" />
    <resetPasswordDto ref="resetPasswordDto" />
  </div>
</template>
<script>
import formData from '@/views/systemManagement/user/formData.vue'
import resetPasswordDto from '@/views/systemManagement/user/resetPasswordDto.vue'
import { getAll, deleteData } from '@/api/user'
import { getRoles } from '@/api/role'
export default {
  components: { formData, resetPasswordDto },
  data() {
    return {
      loading: false,
      tableData: {},
      rolesOption: [],
      selectData: {
        maxResultCount: 10,
        skipCount: 1,
        roleIds: []
      }
    }
  },
  created() {
    this.loadData()
    getRoles().then(response => {
      this.rolesOption = response.result.items
    })
  },
  methods: {
    loadData(selectData) {
      this.loading = true
      if (!selectData) {
        selectData = this.selectData
      }
      getAll(selectData).then(response => {
        this.tableData = response.result
      }).finally(() => {
        this.loading = false
      })
    },
    handleDelete(row) {
      this.$confirm('此操作将删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteData({ id: row.id }).then(response => {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          this.loadData()
        })
      }).catch(() => {
      })
    },
    handleSizeChange(val) {
      this.selectData.maxResultCount = val
      this.loadData(this.selectData)
    },
    handleCurrentChange(val) {
      this.selectData.skipCount = val
      this.loadData(this.selectData)
    }
  }
}
</script>
