<template>
  <div class="app-container">
    <el-form :model="queryParams" @submit.navite.prevent ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="设备编号" prop="innerCode">
        <el-input
          v-model="queryParams.innerCode"
          placeholder="请输入设备编号"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>

<!--      region  desc=""-->
<!--      <el-form-item label="点位Id" prop="nodeId">-->
<!--        <el-input-->
<!--          v-model="queryParams.nodeId"-->
<!--          placeholder="请输入点位Id"-->
<!--          clearable-->
<!--          @keyup.enter="handleQuery"-->
<!--        />-->
<!--      </el-form-item>-->
<!--      <el-form-item label="上次补货时间" prop="lastSupplyTime">-->
<!--        <el-date-picker clearable-->
<!--          v-model="queryParams.lastSupplyTime"-->
<!--          type="date"-->
<!--          value-format="YYYY-MM-DD"-->
<!--          placeholder="请选择上次补货时间">-->
<!--        </el-date-picker>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="区域Id" prop="regionId">-->
<!--        <el-input-->
<!--          v-model="queryParams.regionId"-->
<!--          placeholder="请输入区域Id"-->
<!--          clearable-->
<!--          @keyup.enter="handleQuery"-->
<!--        />-->
<!--      </el-form-item>-->
<!--      <el-form-item label="合作商Id" prop="partnerId">-->
<!--        <el-input-->
<!--          v-model="queryParams.partnerId"-->
<!--          placeholder="请输入合作商Id"-->
<!--          clearable-->
<!--          @keyup.enter="handleQuery"-->
<!--        />-->
<!--      </el-form-item>-->
<!--      <el-form-item label="设备型号" prop="vmTypeId">-->
<!--        <el-input-->
<!--          v-model="queryParams.vmTypeId"-->
<!--          placeholder="请输入设备型号"-->
<!--          clearable-->
<!--          @keyup.enter="handleQuery"-->
<!--        />-->
<!--      </el-form-item>-->
<!--      <el-form-item label="设备状态，0:未投放;1-运营;3-撤机" prop="vmStatus">-->
<!--        <el-select v-model="queryParams.vmStatus" placeholder="请选择设备状态，0:未投放;1-运营;3-撤机" clearable>-->
<!--          <el-option-->
<!--            v-for="dict in vm_status"-->
<!--            :key="dict.value"-->
<!--            :label="dict.label"-->
<!--            :value="dict.value"-->
<!--          />-->
<!--        </el-select>-->
<!--      </el-form-item>-->
<!--      <el-form-item label="客户端连接Id,做emq认证用" prop="clientId">-->
<!--        <el-input-->
<!--          v-model="queryParams.clientId"-->
<!--          placeholder="请输入客户端连接Id,做emq认证用"-->
<!--          clearable-->
<!--          @keyup.enter="handleQuery"-->
<!--        />-->
<!--      </el-form-item>-->
<!--      <el-form-item label="策略id" prop="policyId">-->
<!--        <el-input-->
<!--          v-model="queryParams.policyId"-->
<!--          placeholder="请输入策略id"-->
<!--          clearable-->
<!--          @keyup.enter="handleQuery"-->
<!--        />-->
<!--      </el-form-item>-->
<!--      endregion-->


      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-table v-loading="loading" :data="vmList" @selection-change="handleSelectionChange">
      <el-table-column label="序号" width="50"  type="index" align="center" />
      <el-table-column label="设备编号" align="center" prop="innerCode" />
<!--      通过查找到的设备类别列表对应id显示设备类别名称-->
      <el-table-column label="设备型号" align="center" prop="vmTypeId">
        <template #default="scope">
          <span>{{ vmTypeList.find(item => item.id === scope.row.vmTypeId)?.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="详细地址" align="center" prop="addr" />

      <el-table-column label="设备状态" align="center" prop="vmStatus">
        <template #default="scope">
          <span v-if="scope.row.runningStatus!=null">
            {{JSON.parse(scope.row.runningStatus).status?"正常":"异常"}}
          </span>
          <span v-else> 异常 </span>
        </template>
      </el-table-column>

      <el-table-column label="运营状态" align="center" prop="vmStatus">
        <template #default="scope">
          <dict-tag :options="vm_status" :value="scope.row.vmStatus"/>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary"  @click="getVmInfo(scope.row)" v-hasPermi="['manage:vm:query']">查看详情</el-button>

        </template>
      </el-table-column>
    </el-table>
    
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改设备管理对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>

    </el-dialog>
  </div>
</template>

<script setup name="Vm">
import { listVm, getVm, delVm, addVm, updateVm } from "@/api/manage/vm";
//引入获取设备类型的列表方法
import { listVmType } from "@/api/manage/vmType";
//引入获取合作商的列表方法
import { listPartner } from "@/api/manage/partner";
//引入点位列表方法
import {listNode} from "@/api/manage/node.js";
import {parseTime} from "../../../utils/ruoyi.js";
// 获取区域列表
import { listRegion } from "@/api/manage/region";

const { proxy } = getCurrentInstance();
const { vm_status } = proxy.useDict('vm_status');

const vmList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    innerCode: null,
    nodeId: null,
    lastSupplyTime: null,
    businessType: null,
    regionId: null,
    partnerId: null,
    vmTypeId: null,
    vmStatus: null,
    runningStatus: null,
    clientId: null,
    policyId: null,
  },
  rules: {
    innerCode: [
      { required: true, message: "设备编号不能为空", trigger: "blur" }
    ],
    nodeId: [
      { required: true, message: "点位Id不能为空", trigger: "blur" }
    ],
    vmTypeId: [
      { required: true, message: "设备型号不能为空", trigger: "blur" }
    ],
  }
});

const { queryParams, form, rules } = toRefs(data);

/** 查询设备管理列表 */
function getList() {
  loading.value = true;
  listVm(queryParams.value).then(response => {
    vmList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}

// 取消按钮
function cancel() {
  open.value = false;
  reset();
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    innerCode: null,
    channelMaxCapacity: null,
    nodeId: null,
    addr: null,
    lastSupplyTime: null,
    businessType: null,
    regionId: null,
    partnerId: null,
    vmTypeId: null,
    vmStatus: null,
    runningStatus: null,
    longitudes: null,
    latitude: null,
    clientId: null,
    policyId: null,
    createTime: null,
    updateTime: null
  };
  proxy.resetForm("vmRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加设备管理";
}

/** 修改按钮操作 */
function getVmInfo(row) {
  reset();
  const _id = row.id || ids.value
  getVm(_id).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "查看设备详情";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["vmRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateVm(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addVm(form.value).then(response => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal.confirm('是否确认删除设备管理编号为"' + _ids + '"的数据项？').then(function() {
    return delVm(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('manage/vm/export', {
    ...queryParams.value
  }, `vm_${new Date().getTime()}.xlsx`)
}

/**
 * 获取设备类型列表
 */
const vmTypeList = ref([]);
// 设置参数
const vmTypeParams = ref({
  pageNum: 1,
  pageSize: 100,
});
// 获取设备类型列表
function getVmTypeList() {
  listVmType(vmTypeParams.value).then(response => {
    vmTypeList.value = response.rows;
  });
}


/**
 * 获取合作商列表
 */
const partnerList = ref([]);
// 设置参数
const partnerParams = ref({
  pageNum: 1,
  pageSize: 100,
});
// 获取合作商列表
function getPartnerList() {
  listPartner(partnerParams.value).then(response => {
    partnerList.value = response.rows;
  });
}

/**
 * 获取点位列表
 */
const nodeList = ref([]);
// 设置参数
const nodeParams = ref({
  pageNum: 1,
  pageSize: 100,
});
// 获取点位列表
function getNodeList() {
  listNode(nodeParams.value).then(response => {
    nodeList.value = response.rows;
  });
}

/**
 * 获取区域列表
 */
const regionList = ref([]);
// 设置参数
const regionParams = ref({
  pageNum: 1,
  pageSize: 100,
});
// 获取区域列表
function getRegionList() {
  listRegion(regionParams.value).then(response => {
    regionList.value = response.rows;
  });
}



getRegionList();
getNodeList();
getPartnerList();
getVmTypeList();
getList();
</script>
