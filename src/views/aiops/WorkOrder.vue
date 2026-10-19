<template>
  <div class="page-header-index-wide work-order-page">
    <a-card class="summary-card" :bordered="false">
      <div class="page-header">
        <div>
          <h2>运维工单</h2>
          <p>当前运维任务概览，便于快速掌握现场工单处理状态。</p>
        </div>
        <a-button v-if="canCreate" type="primary" icon="plus" @click="showCreate">新建报修</a-button>
      </div>

      <a-row :gutter="16" class="stat-row">
        <a-col :span="6">
          <a-card class="mini-card">
            <a-statistic title="待受理" :value="summary.pending" :value-style="{ color: '#f5222d' }" />
          </a-card>
        </a-col>
        <a-col :span="6">
          <a-card class="mini-card">
            <a-statistic title="已派单" :value="summary.assigned" :value-style="{ color: '#fa8c16' }" />
          </a-card>
        </a-col>
        <a-col :span="6">
          <a-card class="mini-card">
            <a-statistic title="待验收" :value="summary.waitAcceptance" :value-style="{ color: '#52c41a' }" />
          </a-card>
        </a-col>
        <a-col :span="6">
          <a-card class="mini-card">
            <a-statistic title="总工单" :value="summary.total" :value-style="{ color: '#1890ff' }" />
          </a-card>
        </a-col>
      </a-row>
    </a-card>

    <a-card class="table-card" :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline" @keyup.enter.native="searchQuery">
          <a-row :gutter="24">
            <a-col :xl="6" :lg="8" :md="12" :sm="24">
              <a-form-item label="工单编号">
                <a-input v-model="queryParam.orderNo" placeholder="请输入工单编号" />
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="8" :md="12" :sm="24">
              <a-form-item label="企业编码">
                <a-input v-model="queryParam.companyCode" placeholder="请输入企业编码" />
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="8" :md="12" :sm="24">
              <a-form-item label="监测点ID">
                <a-input v-model="queryParam.pointId" placeholder="请输入监测点ID" />
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="8" :md="12" :sm="24">
              <a-form-item label="状态">
                <a-select v-model="queryParam.orderStatus" allowClear placeholder="全部状态">
                  <a-select-option :value="0">待受理</a-select-option>
                  <a-select-option :value="1">已派单</a-select-option>
                  <a-select-option :value="3">待验收</a-select-option>
                  <a-select-option :value="4">已关闭</a-select-option>
                  <a-select-option :value="5">已退回</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <span class="table-page-search-submitButtons">
                <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                <a-button @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
                <a-button v-if="canCreate" type="primary" icon="plus" @click="showCreate" style="margin-left: 8px">新建报修</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <div class="table-title">
        <span>工单列表</span>
        <a-tag color="blue">{{ list.length }} 条</a-tag>
      </div>

      <a-table
        rowKey="id"
        :columns="columns"
        :dataSource="list"
        :pagination="ipagination"
        :loading="loading"
        :rowClassName="rowClassName"
        @change="handleTableChange">
        <span slot="status" slot-scope="text, record">
          <a-tag :color="statusColor[String(text)] || 'blue'">{{ getStatusText(text) }}</a-tag>
          <a-tag v-if="isSoonTimeout(record)" color="orange" class="soon-timeout-tag">即将超时</a-tag>
        </span>
        <span slot="action" slot-scope="text, record">
          <a @click="showDetail(record)">详情</a>
          <a-divider type="vertical" />
          <template v-if="canAssign(record)">
            <a @click="showAssign(record)">派单</a>
            <a-divider type="vertical" />
          </template>
          <template v-if="canHandle(record)">
            <a @click="showHandle(record)">维修反馈</a>
            <a-divider type="vertical" />
          </template>
          <template v-if="canEvaluate(record)">
            <a @click="showEvaluate(record)">验收评价</a>
          </template>
          <template v-if="!canAssign(record) && !canHandle(record) && !canEvaluate(record)">
            <span class="empty-text">-</span>
          </template>
        </span>
      </a-table>
    </a-card>

    <a-modal
      title="新建报修"
      :visible="createVisible"
      :confirmLoading="submitLoading"
      @ok="submitCreate"
      @cancel="closeCreate"
      width="700px">
      <a-form-model ref="createFormRef" :model="createForm" :label-col="{ span: 5 }" :wrapper-col="{ span: 17 }">
        <a-form-model-item label="企业编码" prop="companyCode">
          <a-input v-model="createForm.companyCode" placeholder="请输入企业编码" />
        </a-form-model-item>
        <a-form-model-item label="监测点ID" prop="pointId">
          <a-input v-model="createForm.pointId" placeholder="请输入监测点ID" />
        </a-form-model-item>
        <a-form-model-item label="报修联系人" prop="repairMan">
          <a-input v-model="createForm.repairMan" placeholder="请输入报修联系人" />
        </a-form-model-item>
        <a-form-model-item label="报修电话" prop="repairPhone">
          <a-input v-model="createForm.repairPhone" placeholder="请输入报修电话" />
        </a-form-model-item>
        <a-form-model-item label="工单编号" prop="orderNo">
          <a-input v-model="createForm.orderNo" placeholder="可不填，系统自动生成" />
        </a-form-model-item>
        <a-form-model-item label="故障描述" prop="faultDesc">
          <a-textarea v-model="createForm.faultDesc" :rows="4" placeholder="请输入故障描述" />
        </a-form-model-item>
      </a-form-model>
    </a-modal>

    <a-modal
      title="派单"
      :visible="assignVisible"
      :confirmLoading="submitLoading"
      @ok="submitAssign"
      @cancel="closeAssign"
      width="500px">
      <a-form-model :model="assignForm" :label-col="{ span: 5 }" :wrapper-col="{ span: 17 }">
        <a-form-model-item label="维修人员">
          <a-input v-model="assignForm.assignTo" placeholder="请输入维修人员账号（username）" />
        </a-form-model-item>
      </a-form-model>
    </a-modal>

    <a-modal
      title="维修反馈"
      :visible="handleVisible"
      :confirmLoading="submitLoading"
      @ok="submitHandle"
      @cancel="closeHandle"
      width="600px">
      <a-form-model :model="handleForm" :label-col="{ span: 5 }" :wrapper-col="{ span: 17 }">
        <a-form-model-item label="处理结果">
          <a-textarea v-model="handleForm.handleResult" :rows="4" placeholder="请输入维修处理结果" />
        </a-form-model-item>
      </a-form-model>
    </a-modal>

    <a-modal
      title="验收评价"
      :visible="evaluateVisible"
      :confirmLoading="submitLoading"
      @ok="submitEvaluate"
      @cancel="closeEvaluate"
      width="600px">
      <a-form-model :model="evaluateForm" :label-col="{ span: 5 }" :wrapper-col="{ span: 17 }">
        <a-form-model-item label="评分">
          <a-rate v-model="evaluateForm.evaluateScore" :count="5" />
        </a-form-model-item>
        <a-form-model-item label="评价意见">
          <a-textarea v-model="evaluateForm.evaluateOpinion" :rows="4" placeholder="请输入验收评价意见" />
        </a-form-model-item>
      </a-form-model>
    </a-modal>

    <a-modal title="工单详情" :visible="detailVisible" :footer="null" @cancel="closeDetail" width="800px">
      <a-alert v-if="detailCountdown" :type="detailCountdown.type" :show-icon="true" class="detail-countdown-alert">
        <template #message>
          {{ detailCountdown.message }}
        </template>
        <template #description>
          {{ detailCountdown.description }}
        </template>
      </a-alert>
      <a-steps :current="detailTimelineCurrent" size="small" class="detail-timeline">
        <a-step title="待受理" :description="getDetailTimeText('待受理')" />
        <a-step title="已派单" :description="getDetailTimeText('已派单')" />
        <a-step title="维修中" :description="getDetailTimeText('维修中')" />
        <a-step title="待验收" :description="getDetailTimeText('待验收')" />
        <a-step :title="detailTimelineFinalTitle" :description="getDetailTimeText(detailTimelineFinalTitle)" />
      </a-steps>
      <a-descriptions bordered :column="1" size="small">
        <a-descriptions-item v-for="item in detailItems" :key="item.key" :label="item.label">
          {{ item.value }}
        </a-descriptions-item>
      </a-descriptions>
    </a-modal>
  </div>
</template>

<script>
import { getAction, postAction } from '@/api/manage'

export default {
  name: 'AiopsWorkOrder',
  data() {
    return {
      loading: false,
      submitLoading: false,
      createVisible: false,
      assignVisible: false,
      handleVisible: false,
      evaluateVisible: false,
      detailVisible: false,
      list: [],
      summary: {
        pending: 0,
        assigned: 0,
        waitAcceptance: 0,
        total: 0
      },
      queryParam: {
        orderNo: '',
        companyCode: '',
        pointId: '',
        orderStatus: undefined
      },
      ipagination: {
        current: 1,
        pageSize: 10,
        total: 0,
        showTotal: (total) => `共 ${total} 条`
      },
      statusMap: {
        '0': '待受理',
        '1': '已派单',
        '3': '待验收',
        '4': '已关闭',
        '5': '已退回'
      },
      detailFieldLabelMap: {
        id: '主键ID',
        orderNo: '工单编号',
        companyCode: '企业编码',
        pointId: '监测点ID',
        faultDesc: '故障描述',
        repairMan: '报修联系人',
        repairPhone: '报修电话',
        orderStatus: '工单状态',
        statusName: '状态名称',
        assignTo: '维修人员账号',
        assignMan: '维修人员',
        assignTime: '派单时间',
        handleStartTime: '开始维修时间',
        handleResult: '处理结果',
        handleTime: '处理时间',
        closeTime: '关闭时间',
        returnTime: '退回时间',
        evaluateScore: '评分',
        evaluateOpinion: '评价意见',
        evaluateTime: '评价时间',
        createBy: '创建人',
        createTime: '创建时间',
        updateBy: '更新人',
        updateTime: '更新时间',
        delFlag: '删除标识',
        tenantId: '租户ID'
      },
      statusColor: {
        '0': 'red',
        '1': 'orange',
        '3': 'green',
        '4': 'blue',
        '5': 'purple'
      },
      columns: [
        { title: '工单编号', dataIndex: 'orderNo', key: 'orderNo', align: 'center', width: 180 },
        { title: '企业编码', dataIndex: 'companyCode', key: 'companyCode', align: 'center', width: 140 },
        { title: '监测点', dataIndex: 'pointId', key: 'pointId', align: 'center', width: 150 },
        { title: '故障描述', dataIndex: 'faultDesc', key: 'faultDesc', width: 260 },
        { title: '报修人', dataIndex: 'repairMan', key: 'repairMan', width: 110, align: 'center' },
        { title: '维修人员', dataIndex: 'assignTo', key: 'assignTo', width: 120, align: 'center' },
        { title: '状态', dataIndex: 'orderStatus', key: 'orderStatus', scopedSlots: { customRender: 'status' }, width: 110, align: 'center' },
        { title: '操作', key: 'action', scopedSlots: { customRender: 'action' }, width: 220, align: 'center' }
      ],
      createForm: {
        companyCode: '',
        pointId: '',
        faultDesc: '',
        repairMan: '',
        repairPhone: '',
        orderNo: ''
      },
      assignForm: {
        orderId: '',
        assignTo: ''
      },
      handleForm: {
        orderId: '',
        handleResult: ''
      },
      evaluateForm: {
        orderId: '',
        evaluateScore: 5,
        evaluateOpinion: ''
      },
      detailRecord: {},
      nowTime: Date.now(),
      detailCountdownTimer: null
    }
  },
  computed: {
    detailItems() {
      const record = this.detailRecord || {}
      const hiddenKeys = ['id', 'sysOrgCode']
      return Object.keys(record).filter(key => !hiddenKeys.includes(key)).map((key) => {
        const value = record[key]
        let displayValue = '-'
        if (key === 'orderStatus') {
          displayValue = this.getStatusText(value)
          return { key, label: this.getDetailFieldLabel(key), value: displayValue }
        }
        if (value !== null && typeof value !== 'undefined') {
          if (Array.isArray(value) || typeof value === 'object') {
            try {
              displayValue = JSON.stringify(value)
            } catch (e) {
              displayValue = String(value)
            }
          } else {
            displayValue = String(value)
          }
        }
        return { key, label: this.getDetailFieldLabel(key), value: displayValue }
      })
    },
    detailTimelineCurrent() {
      const status = String((this.detailRecord && this.detailRecord.orderStatus) || '')
      if (status === '0') return 0
      if (status === '1') return 1
      if (status === '2') return 2
      if (status === '3') return 3
      if (status === '4' || status === '5') return 4
      return 0
    },
    detailTimelineFinalTitle() {
      const status = String((this.detailRecord && this.detailRecord.orderStatus) || '')
      return status === '5' ? '已退回' : '已关闭'
    },
    detailCountdown() {
      const record = this.detailRecord || {}
      const status = String(record.orderStatus || '')
      const now = this.nowTime || Date.now()
      const parseMs = (value) => {
        if (!value && value !== 0) return null
        const time = new Date(value).getTime()
        if (Number.isNaN(time)) return null
        return time
      }
      const getDeadline = () => {
        if (status === '0') {
          const createTime = parseMs(record.createTime)
          if (createTime) return createTime + 24 * 60 * 60 * 1000
          return null
        }
        if (status === '1') {
          const assignTime = parseMs(record.assignTime)
          const fallback = parseMs(record.createTime)
          if (assignTime) return assignTime + 72 * 60 * 60 * 1000
          if (fallback) return fallback + 72 * 60 * 60 * 1000
          return null
        }
        return null
      }
      const deadline = getDeadline()
      if (!deadline) {
        return null
      }
      const diff = deadline - now
      if (diff <= 0) {
        const overText = this.formatDuration(Math.abs(diff))
        if (status === '0') {
          return {
            type: 'error',
            message: '超时未受理提醒',
            description: `已超时 ${overText}，请尽快派单处理。`
          }
        }
        return {
          type: 'error',
          message: '超时未处理提醒',
          description: `已超时 ${overText}，请尽快完成维修。`
        }
      }
      const remainText = this.formatDuration(diff)
      return {
        type: 'warning',
        message: status === '0' ? '距超时派单提醒' : '距超时处理提醒',
        description: `还剩 ${remainText}，请在规定时间内完成${status === '0' ? '派单' : '维修处理'}。`
      }
    },
    currentUser() {
      return this.$store.getters.userInfo || {}
    },
    currentUsername() {
      return this.currentUser.username || this.currentUser.account || this.currentUser.userName || ''
    },
    roleCodes() {
      const info = this.$store.getters.userInfo || {}
      const values = []
      const collect = (value) => {
        if (Array.isArray(value)) {
          value.forEach(item => collect(item))
          return
        }
        if (!value) {
          return
        }
        if (typeof value === 'string') {
          value.split(/[\s,;]+/).filter(Boolean).forEach(item => {
            const cleaned = String(item).trim().replace(/[^0-9A-Za-z_-]/g, '')
            if (cleaned) values.push(cleaned)
          })
          return
        }
        if (typeof value === 'object') {
          const roleCode = value.roleCode || value.rolecode || value.code || value.roleId || value.role_id || value.id || value.role || value.roleName
          if (roleCode) {
            values.push(String(roleCode))
          }
          const nestedKeys = ['roles', 'roleList', 'roleCodes', 'sysRoles', 'userRoles', 'roleSet', 'roleIds', 'role_id']
          nestedKeys.forEach(key => {
            if (Array.isArray(value[key])) {
              value[key].forEach(item => collect(item))
            } else if (value[key]) {
              collect(value[key])
            }
          })
        }
      }
      const candidateKeys = ['roles', 'roleList', 'roleCodes', 'roleCode', 'role', 'sysRoles', 'userRoles', 'roleSet', 'roleIds']
      candidateKeys.forEach(key => collect(info[key]))
      if (!values.length && info.roleId) {
        values.push(String(info.roleId))
      }
      const normalized = values
        .map(item => String(item).trim())
        .filter(Boolean)
        .map(item => item.replace(/^ROLE_/, '').replace(/^role_/, '').replace(/[^0-9]/g, ''))
        .filter(Boolean)
      const finalCodes = Array.from(new Set(normalized))
      if (finalCodes.length) {
        return finalCodes
      }
      const directUsername = String(this.currentUsername || '').trim()
      if (['4', '5', '6'].includes(directUsername)) {
        return [directUsername]
      }
      return []
    },
    canCreate() {
      return this.roleCodes.includes('6') || String(this.currentUsername || '').trim() === '6'
    }
  },
  created() {
    this.loadData()
    this.nowTime = Date.now()
    this.detailCountdownTimer = setInterval(() => {
      this.nowTime = Date.now()
    }, 1000)
  },
  beforeDestroy() {
    if (this.detailCountdownTimer) {
      clearInterval(this.detailCountdownTimer)
      this.detailCountdownTimer = null
    }
  },
  methods: {
    formatDuration(ms) {
      if (!Number.isFinite(ms) || ms < 0) return '0天 0小时 0分钟 0秒'
      const totalSeconds = Math.floor(ms / 1000)
      const days = Math.floor(totalSeconds / 86400)
      const hours = Math.floor((totalSeconds % 86400) / 3600)
      const minutes = Math.floor((totalSeconds % 3600) / 60)
      const seconds = totalSeconds % 60
      return `${days}天 ${hours}小时 ${minutes}分钟 ${seconds}秒`
    },
    rowClassName(record) {
      return this.isSoonTimeout(record) ? 'order-soon-timeout-row' : ''
    },
    isSoonTimeout(record) {
      if (!record || !record.orderStatus) return false
      const status = String(record.orderStatus)
      if (!['0', '1'].includes(status)) return false
      const now = this.nowTime || Date.now()
      const parseMs = (value) => {
        if (!value && value !== 0) return null
        const time = new Date(value).getTime()
        return Number.isNaN(time) ? null : time
      }
      const start = status === '0' ? parseMs(record.createTime) : parseMs(record.assignTime) || parseMs(record.createTime)
      if (!start) return false
      const deadline = status === '0'
        ? start + 24 * 60 * 60 * 1000
        : start + 72 * 60 * 60 * 1000
      if (now >= deadline) return false
      const elapsed = now - start
      const total = deadline - start
      if (total <= 0) return false
      return elapsed / total >= 0.5
    },
    getDetailFieldLabel(key) {
      return this.detailFieldLabelMap[key] || key
    },
    showDetail(record) {
      this.detailRecord = { ...(record || {}) }
      this.detailVisible = true
    },
    getDetailTimeText(stepTitle) {
      const record = this.detailRecord || {}
      const getVal = (...keys) => {
        for (let i = 0; i < keys.length; i += 1) {
          const v = record[keys[i]]
          if (v !== null && typeof v !== 'undefined' && String(v).trim() !== '') {
            return String(v)
          }
        }
        return '—'
      }
      if (stepTitle === '待受理') return getVal('createTime')
      if (stepTitle === '已派单') return getVal('assignTime')
      if (stepTitle === '维修中') return getVal('handleStartTime', 'handleTime')
      if (stepTitle === '待验收') return getVal('handleTime')
      if (stepTitle === '已关闭') return getVal('closeTime', 'evaluateTime', 'updateTime')
      if (stepTitle === '已退回') return getVal('returnTime', 'updateTime')
      return '—'
    },
    closeDetail() {
      this.detailVisible = false
      this.detailRecord = {}
    },
    getStatusText(value) {
      return this.statusMap[String(value)] || '未知'
    },
    canAssign(record) {
      return (this.roleCodes.includes('5') || String(this.currentUsername || '').trim() === '5') && String(record.orderStatus) === '0'
    },
    canHandle(record) {
      return (this.roleCodes.includes('4') || String(this.currentUsername || '').trim() === '4') && String(record.orderStatus) === '1' && String(record.assignTo || '') === String(this.currentUsername || '')
    },
    canEvaluate(record) {
      return (this.roleCodes.includes('6') || String(this.currentUsername || '').trim() === '6') && String(record.orderStatus) === '3'
    },
    handleTableChange(pagination) {
      this.ipagination.current = pagination.current
      this.ipagination.pageSize = pagination.pageSize
      this.loadData()
    },
    searchQuery() {
      this.ipagination.current = 1
      this.loadData()
    },
    searchReset() {
      this.queryParam = {
        orderNo: '',
        companyCode: '',
        pointId: '',
        orderStatus: undefined
      }
      this.ipagination.current = 1
      this.loadData()
    },
    rebuildSummary(rows) {
      const summary = {
        pending: 0,
        assigned: 0,
        waitAcceptance: 0,
        total: rows.length
      }
      rows.forEach((item) => {
        const status = String(item.orderStatus)
        if (status === '0') summary.pending += 1
        if (status === '1') summary.assigned += 1
        if (status === '3') summary.waitAcceptance += 1
      })
      this.summary = summary
    },
    parsePageResult(res) {
      const result = res && res.result ? res.result : {}
      const rows = Array.isArray(result) ? result : (result.records || result.list || [])
      const total = Array.isArray(result) ? rows.length : (result.total || result.count || rows.length)
      return { rows, total }
    },
    loadData() {
      this.loading = true
      const params = {
        ...this.queryParam,
        pageNo: this.ipagination.current,
        pageSize: this.ipagination.pageSize
      }
      if (params.orderStatus === '' || params.orderStatus === null || typeof params.orderStatus === 'undefined') {
        delete params.orderStatus
      }
      const requests = ['/psms/workOrder/queryPageList', '/psms/workOrder/list']

      const run = (index) => {
        if (index >= requests.length) {
          this.list = []
          this.ipagination.total = 0
          this.rebuildSummary([])
          this.$message.warning('暂无工单数据或接口未就绪')
          return Promise.resolve()
        }

        const url = requests[index]
        return getAction(url, params).then((res) => {
          if (res && res.success !== false) {
            const { rows, total } = this.parsePageResult(res)
            this.list = rows
            this.ipagination.total = total
            this.rebuildSummary(rows)
            return true
          }
          return false
        }).catch(() => false).then((done) => {
          if (done) return true
          return run(index + 1)
        })
      }

      run(0).finally(() => {
        this.loading = false
      })
    },
    showCreate() {
      this.createVisible = true
    },
    closeCreate() {
      this.createVisible = false
      this.createForm = {
        companyCode: '',
        pointId: '',
        faultDesc: '',
        repairMan: '',
        repairPhone: '',
        orderNo: ''
      }
    },
    showAssign(record) {
      this.assignForm = {
        orderId: record.id,
        assignTo: ''
      }
      this.assignVisible = true
    },
    closeAssign() {
      this.assignVisible = false
      this.assignForm.assignTo = ''
    },
    showHandle(record) {
      this.handleForm = {
        orderId: record.id,
        handleResult: ''
      }
      this.handleVisible = true
    },
    closeHandle() {
      this.handleVisible = false
      this.handleForm.handleResult = ''
    },
    showEvaluate(record) {
      this.evaluateForm = {
        orderId: record.id,
        evaluateScore: 5,
        evaluateOpinion: ''
      }
      this.evaluateVisible = true
    },
    closeEvaluate() {
      this.evaluateVisible = false
      this.evaluateForm.evaluateOpinion = ''
    },
    submitCreate() {
      if (!this.createForm.companyCode || !this.createForm.pointId || !this.createForm.faultDesc || !this.createForm.repairMan || !this.createForm.repairPhone) {
        this.$message.warning('企业编码、监测点ID、故障描述、报修联系人和报修电话不能为空')
        return
      }
      this.submitLoading = true
      postAction('/psms/workOrder/create', this.createForm).then((res) => {
        if (res && res.success !== false) {
          this.$message.success('报修工单创建成功')
          this.closeCreate()
          this.loadData()
        } else {
          this.$message.error((res && res.message) || '创建失败')
        }
      }).catch((error) => {
        this.$message.error(error && error.message ? error.message : '创建失败')
      }).finally(() => {
        this.submitLoading = false
      })
    },
    submitAssign() {
      if (!this.assignForm.orderId || !this.assignForm.assignTo) {
        this.$message.warning('请选择工单并填写维修人员账号')
        return
      }
      this.submitLoading = true
      postAction('/psms/workOrder/assign', this.assignForm).then((res) => {
        if (res && res.success !== false) {
          this.$message.success('派单成功')
          this.closeAssign()
          this.loadData()
        } else {
          this.$message.error((res && res.message) || '派单失败')
        }
      }).catch((error) => {
        this.$message.error(error && error.message ? error.message : '派单失败')
      }).finally(() => {
        this.submitLoading = false
      })
    },
    submitHandle() {
      if (!this.handleForm.orderId || !this.handleForm.handleResult) {
        this.$message.warning('处理结果不能为空')
        return
      }
      this.submitLoading = true
      postAction('/psms/workOrder/handle', this.handleForm).then((res) => {
        if (res && res.success !== false) {
          this.$message.success('维修反馈成功')
          this.closeHandle()
          this.loadData()
        } else {
          this.$message.error((res && res.message) || '维修反馈失败')
        }
      }).catch((error) => {
        this.$message.error(error && error.message ? error.message : '维修反馈失败')
      }).finally(() => {
        this.submitLoading = false
      })
    },
    submitEvaluate() {
      if (!this.evaluateForm.orderId) {
        this.$message.warning('请选择要评价的工单')
        return
      }
      this.submitLoading = true
      postAction('/psms/workOrder/evaluate', {
        orderId: this.evaluateForm.orderId,
        evaluateScore: this.evaluateForm.evaluateScore,
        evaluateOpinion: this.evaluateForm.evaluateOpinion
      }).then((res) => {
        if (res && res.success !== false) {
          this.$message.success('验收评价提交成功')
          this.closeEvaluate()
          this.loadData()
        } else {
          this.$message.error((res && res.message) || '验收评价失败')
        }
      }).catch((error) => {
        this.$message.error(error && error.message ? error.message : '验收评价失败')
      }).finally(() => {
        this.submitLoading = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
.work-order-page {
  padding: 24px;
}

.summary-card {
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;

  h2 {
    margin: 0;
    font-size: 30px;
    font-weight: 600;
    color: #1f1f1f;
  }

  p {
    margin: 8px 0 0;
    color: #666;
  }
}

.stat-row {
  margin-top: 8px;
}

.mini-card {
  border-radius: 10px;
  background: #f7faff;
}

.table-card {
  border-radius: 12px;
  margin-top: 24px;
}

.table-page-search-wrapper {
  margin-bottom: 16px;
}

.table-page-search-submitButtons {
  display: inline-flex;
  align-items: center;
}

.table-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  font-size: 18px;
  font-weight: 600;
}

.empty-text {
  color: #999;
}

.detail-timeline {
  margin-bottom: 16px;
  padding: 12px 16px;
  background: #fafcff;
  border: 1px solid #e6f4ff;
  border-radius: 8px;
}

.order-soon-timeout-row {
  background-color: #fff7e6 !important;
  box-shadow: inset 3px 0 0 #fa8c16;
}

.soon-timeout-tag {
  margin-left: 8px;
}
</style>
