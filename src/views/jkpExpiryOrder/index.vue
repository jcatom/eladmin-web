<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">兑奖状态</label>
        <el-select
          v-model="query.status"
          clearable
          size="small"
          placeholder="兑奖状态"
          class="filter-item"
          style="width: 110px"
          @change="crud.toQuery"
        >
          <el-option
            v-for="item in dict.expiry_status"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
        <label class="el-form-item-label">创建时间</label>
        <date-range-picker v-model="query.createTime" class="date-item" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" size="small" label-width="80px">
          <el-form-item label="订单状态">
            <el-radio-group v-model="form.status" style="width: 280px">
              <el-radio :label="0">待兑奖</el-radio>
              <el-radio :label="1">已兑奖</el-radio>
              <el-radio :label="2">已转账</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="expand">
          <template slot-scope="props">
            <el-form label-position="left" inline class="demo-table-expand">
              <el-form-item>
                <span>
                  <el-image :src="props.row.ticketPhotoUrl" fit="cover" lazy style="width: 100%; height: 100%">
                    <div slot="error">
                      <i class="el-icon-document" />
                    </div>
                  </el-image>
                </span>
              </el-form-item>
              <el-form-item>
                <span>
                  <el-image :src="props.row.ticketPhotoUrl" fit="cover" lazy style="width: 100%; height: 100%">
                    <div slot="error">
                      <i class="el-icon-document" />
                    </div>
                  </el-image>
                </span>
              </el-form-item>
            </el-form>
          </template>
        </el-table-column>
        <el-table-column prop="orderNo" label="订单编号" />
        <el-table-column prop="expiryUserId" label="兑奖用户" />
        <el-table-column prop="ticketPhotoUrl" label="即开票图片">
          <template slot-scope="{row}">
            <el-image
              :src=" row.ticketPhotoUrl"
              fit="cover"
              lazy
            >
              <div slot="error">
                <i class="el-icon-document" />
              </div>
            </el-image>
          </template>
        </el-table-column>
        <el-table-column prop="status" label="兑奖状态">
          <template slot-scope="scope">
            {{ dict.label.expiry_status[scope.row.status] }}
          </template>
        </el-table-column>
        <el-table-column prop="createTime" label=" 创建时间" />
        <el-table-column v-if="checkPer(['admin','jkpExpiryOrder:edit','jkpExpiryOrder:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudJkpExpiryOrder from '@/api/jkpExpiryOrder/jkpExpiryOrder'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from '@/components/DateRangePicker'

const defaultForm = { id: null, orderNo: null, expiryUserId: null, ticketPhotoUrl: null, status: 0, createTime: null, updateTime: null }
export default {
  name: 'JkpExpiryOrder',
  components: { pagination, crudOperation, rrOperation, udOperation, DateRangePicker },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['expiry_status'],
  cruds() {
    return CRUD({
      title: '订单状态',
      url: 'api/jkpExpiryOrder',
      dField: 'id',
      sort: ['createTime,desc'],
      crudMethod: { ...crudJkpExpiryOrder },
      optShow: {
        add: false,
        edit: false,
        del: false,
        reset: true,
        download: true
      }
    })
  },
  data() {
    return {
      permission: {
        add: ['admin', 'jkpExpiryOrder:add'],
        edit: ['admin', 'jkpExpiryOrder:edit'],
        del: ['admin', 'jkpExpiryOrder:del']
      },
      queryTypeOptions: [
        { key: 'status', display_name: '兑奖状态' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
