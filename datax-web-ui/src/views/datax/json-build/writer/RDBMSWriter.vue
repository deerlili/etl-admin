<template>
  <div>
    <el-form label-position="right" label-width="150px" :model="writerForm" :rules="rules">
      <el-form-item label="数据库源：" prop="datasourceId">
        <el-select
          v-model="writerForm.datasourceId"
          filterable
          style="width: 300px;"
          @change="wDsChange"
        >
          <el-option
            v-for="item in wDsList"
            :key="item.id"
            :label="item.datasourceName"
            :value="item.id"
          />
        </el-select>
      </el-form-item>
      <el-form-item v-show="needSchema" label="Schema：" prop="tableSchema">
        <el-select v-model="writerForm.tableSchema" allow-create default-first-option filterable style="width: 300px" @change="schemaChange">
          <el-option
            v-for="item in schemaList"
            :key="item"
            :label="item"
            :value="item"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="数据库表名：" prop="tableName">
        <el-select
          v-model="fromTableName"
          allow-create
          default-first-option
          filterable
          :disabled="writerForm.ifCreateTable"
          style="width: 300px"
          @change="wTbChange"
        >
          <el-option
            v-for="item in wTbList"
            :key="item"
            :label="item"
            :value="item"
          />
        </el-select>
        <el-input v-show="writerForm.ifCreateTable" v-model="writerForm.tableName" style="width: 200px;" :placeholder="readerForm.tableName" />
        <!--<el-input v-model="createTableName" style="width: 195px" />
        <el-button type="primary" @click="createTable">新增</el-button>-->
      </el-form-item>
      <div style="margin: 5px 0;" />
      <el-form-item label="字段：">
        <el-checkbox v-model="writerForm.checkAll" :indeterminate="writerForm.isIndeterminate" @change="wHandleCheckAllChange">全选</el-checkbox>
        <div style="margin: 15px 0;" />
        <el-checkbox-group v-model="writerForm.columns" @change="wHandleCheckedChange">
          <el-checkbox v-for="c in fromColumnList" :key="c" :label="c">{{ c }}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="前置sql语句：">
        <el-input v-model="writerForm.preSql" placeholder="前置sql在insert之前执行" type="textarea" style="width: 42%" />
      </el-form-item>
      <el-form-item label="postSql">
        <el-input v-model="writerForm.postSql" placeholder="多个用;分隔" type="textarea" style="width: 42%" />
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import * as dsQueryApi from '@/api/metadata-query'
import { list as jdbcDsList } from '@/api/datax-jdbcDatasource'
import Bus from '../busWriter'
export default {
  name: 'RDBMSWriter',
  data() {
    return {
      jdbcDsQuery: {
        current: 1,
        size: 200,
        ascs: 'datasource_name'
      },
      needShemaList: ['POSTGRESQL', ,'GREENPLUM', 'ORACLE', 'SQLSERVER', 'DB2', 'OSCAR'],
      wDsList: [],
      schemaList: [],
      fromTableName: '',
      fromColumnList: [],
      wTbList: [],
      dataSource: '',
      needSchema: false,
      createTableName: '',
      writerForm: {
        datasourceId: undefined,
        tableName: '',
        columns: [],
        checkAll: false,
        isIndeterminate: true,
        preSql: '',
        postSql: '',
        ifCreateTable: false,
        tableSchema: ''
      },
      readerForm: this.getReaderData(),
      rules: {
        datasourceId: [{ required: true, message: 'this is required', trigger: 'change' }],
        tableName: [{ required: true, message: 'this is required', trigger: 'change' }],
        tableSchema: [{ required: true, message: 'this is required', trigger: 'change' }]
      }
    }
  },
  watch: {
    'writerForm.datasourceId': function(oldVal, newVal) {
      // 当需要选择schemas时，先选择schemas再加载表
      if (this.needShemaList.includes(this.dataSource)) {
        this.getSchema()
        this.needSchema = true
      } else {
        this.getTables('rdbmsWriter')
        this.needSchema = false
      }
    }
  },
  created() {
    this.getJdbcDs()
  },
  methods: {
    // 获取可用数据源
    getJdbcDs() {
      this.loading = true
      jdbcDsList(this.jdbcDsQuery).then(response => {
        const { records } = response
        this.wDsList = records
        //this.wDsList.push({ id: -1, datasourceName: 'TXT文件', datasource: 'txtfile' })
        this.dataSource = this.wDsList[0].type
        this.writerForm.datasourceId = this.wDsList[0].id
        if (this.needShemaList.includes(this.dataSource)) {
          this.needSchema = true
        } else {
          this.needSchema = false
        }
        this.loading = false
      })
    },
    // 获取表名
    getTables(type) {
      if (type === 'rdbmsWriter') {
        let obj = {}
        if (this.needSchema) {
          obj = {
            datasourceId: this.writerForm.datasourceId,
            tableSchema: this.writerForm.tableSchema
          }
        } else {
          obj = {
            datasourceId: this.writerForm.datasourceId
          }
        }
        // 组装
        dsQueryApi.getTables(obj).then(response => {
          this.wTbList = response
        })
      }
    },
    getSchema() {
      const obj = {
        datasourceId: this.writerForm.datasourceId
      }
      dsQueryApi.getTableSchema(obj).then(response => {
        this.schemaList = response
      })
    },
    // schema 切换
    schemaChange(e) {
      this.writerForm.tableSchema = e
      // 获取可用表
      this.getTables('rdbmsWriter')
    },
    wDsChange(e) {
      // 清空
      this.fromTableName = ''
      this.writerForm.tableName = ''
      this.writerForm.datasourceId = e
      this.wDsList.find((item) => {
        if (item.id === e) {
          this.dataSource = item.type
        }
      })
      Bus.dataSourceId = e
      this.$emit('selectDataSource', this.dataSource)
      // 切换数据源时，清空可用表列表，当需要选择schemas时，先选择schemas再加载表
      this.writerForm.tableSchema = ''
      this.wTbList = []
    },
    // 获取表字段
    getColumns() {
      const obj = {
        datasourceId: this.writerForm.datasourceId,
        tableName: this.writerForm.tableName
      }
      dsQueryApi.getColumns(obj).then(response => {
        this.fromColumnList = response
        this.writerForm.columns = response
        this.writerForm.checkAll = true
        this.writerForm.isIndeterminate = false
      })
    },
    // 表切换
    wTbChange(t) {
      this.writerForm.tableName = t
      this.fromColumnList = []
      this.writerForm.columns = []
      this.getColumns('writer')
    },
    wHandleCheckAllChange(val) {
      this.writerForm.columns = val ? this.fromColumnList : []
      this.writerForm.isIndeterminate = false
    },
    wHandleCheckedChange(value) {
      const checkedCount = value.length
      this.writerForm.checkAll = checkedCount === this.fromColumnList.length
      this.writerForm.isIndeterminate = checkedCount > 0 && checkedCount < this.fromColumnList.length
    },
    getData() {
      if (Bus.dataSourceId) {
        this.writerForm.datasourceId = Bus.dataSourceId
      }
      return this.writerForm
    },
    getReaderData() {
      return this.$parent.getReaderData()
    },
    getTableName() {
      return this.fromTableName
    },
    createTable() {
      const obj = {
        datasourceId: this.writerForm.datasourceId,
        tableName: this.createTableName
      }
      dsQueryApi.createTable(obj).then(response => {
        this.$notify({
          title: 'Success',
          message: 'Create Table Successfully',
          type: 'success',
          duration: 2000
        })
      }).catch(() => console.log('promise catch err'))
    }
  }
}
</script>
