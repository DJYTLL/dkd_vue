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

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['manage:vm:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['manage:vm:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['manage:vm:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['manage:vm:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="vmList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="序号" width="50"  type="index" align="center" />
      <el-table-column label="设备编号" align="center" prop="innerCode" />
<!--      通过查找到的设备类别列表对应id显示设备类别名称-->
      <el-table-column label="设备型号" align="center" prop="vmTypeId">
        <template #default="scope">
          <span>{{ vmTypeList.find(item => item.id === scope.row.vmTypeId)?.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="详细地址" align="center" prop="addr" />
<!--      通过查找到的合作商中的id显示合作商的名称-->
      <el-table-column label="合作商" align="center" prop="partnerId">
        <template #default="scope">
          <span>{{ partnerList.find(item => item.id === scope.row.partnerId)?.partnerName }}</span>
        </template>
      </el-table-column>
      <el-table-column label="设备状态" align="center" prop="vmStatus">
        <template #default="scope">
          <dict-tag :options="vm_status" :value="scope.row.vmStatus"/>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" @click="handleGoods(scope.row)" v-hasPermi="['manage:vm:edit']">货道</el-button>
          <el-button link type="primary"  @click="handleUpdatePolicy(scope.row)" v-hasPermi="['manage:vm:edit']">策略</el-button>
          <el-button link type="primary"  @click="handleUpdate(scope.row)" v-hasPermi="['manage:vm:edit']">修改</el-button>
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
      <el-form ref="vmRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="设备编号" >
<!--          新增时设备编号自动生成，修改时设备编号自动填入-->
          <span>{{ form.innerCode===null?"系统自动生成":form.innerCode }}</span>
        </el-form-item>

        <el-form-item v-if="form.id" label="供货时间" prop="innerCode">
          <span>{{ parseTime(form.lastSupplyTime, "{y}-{m}-{d}  {h}:{i}:{s}") }}</span>

        </el-form-item>

        <el-form-item v-if="form.id" label="设备类型" prop="innerCode">
          <span>{{ vmTypeList.find(item => item.id === form.vmTypeId)?.name }}</span>
        </el-form-item>

        <el-form-item v-if="form.id" label="设备容量" prop="innerCode">
          <span>{{ form.channelMaxCapacity }}</span>
        </el-form-item>


        <!--       通过查找到的点位列表生成下拉选择框选择点位 -->
        <el-form-item  label="选择点位" prop="nodeId">
          <el-select v-model="form.nodeId" placeholder="请选择">
            <el-option
              v-for="item in nodeList"
              :key="item.key"
              :label="item.nodeName"
              :value="item.id"
            />
          </el-select>
        </el-form-item>

        <el-form-item v-if="form.id" label="合作商" >
          <span>{{ partnerList.find(item => item.id === form.partnerId)?.partnerName }}</span>
        </el-form-item>

        <el-form-item v-if="form.id" label="所属区域" >
          <span>{{ regionList.find(item => item.id === form.regionId)?.regionName }}</span>
        </el-form-item>

        <el-form-item   v-if="form.id"  label="设备地址" >
          <span>{{form.addr}}</span>
        </el-form-item>


<!--        通过查找到的型号列表生成下拉选择框选择设备型号-->
        <el-form-item v-if="!form.id" label="选择型号" prop="vmTypeId">
          <el-select v-model="form.vmTypeId" placeholder="请选择">
            <el-option
              v-for="item in vmTypeList"
              :key="item.key"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>

    <el-dialog title="设备策略" v-model="policyListOpen" width="500px" append-to-body>
      <el-form ref="vmRef" :model="form"  label-width="80px">
        <el-form-item label="选择策略" prop="policyId">
          <el-select v-model="form.policyId" placeholder="请选择">
            <el-option
              v-for="item in policyList"
              :key="item.key"
              :label="item.policyName"
              :value="item.policyId"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>


    <!-- 货道组件 -->
    <ChannelDialog :goodVisible="goodVisible" :goodData="goodData" @handleCloseGood="handleCloseGood"></ChannelDialog>
    <!-- end -->

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
//获取设备策略列表
import {listPolicy} from "@/api/manage/policy.js";

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
  policyListOpen.value = false;
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
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getVm(_id).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "修改设备管理";
  });
}

const policyList = ref([]);
const policyListParams = ref({
  pageNum: 1,
  pageSize: 1000,
});
const policyListOpen = ref(false);

//修改设备策略
function handleUpdatePolicy(row){
  reset();
  form.value.id = row.id || ids.value;
  form.value.policyId = row.policyId;
  form.value.nodeId = row.nodeId;
  listPolicy(policyListParams.value).then(response => {
    policyList.value = response.rows;
    policyListOpen.value = true;
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
          policyListOpen.value = false;
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



// ********************货道********************
// 货道组件
import ChannelDialog from './components/ChannelDialog.vue';
const goodVisible = ref(false); //货道弹层显示隐藏
const goodData = ref({}); //货道信息用来拿取 vmTypeId和innerCode
// 打开货道弹层
const handleGoods = (row) => {
  goodVisible.value = true;
  goodData.value = row;
};
// 关闭货道弹层
const handleCloseGood = () => {
  goodVisible.value = false;
};
// ********************货道end********************



getRegionList();
getNodeList();
getPartnerList();
getVmTypeList();
getList();
</script>
<style lang="scss" scoped src="./index.scss"></style>