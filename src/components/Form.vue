<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-12-23 09:34
 * @LastAuthor : itchaox
 * @LastTime   : 2024-02-02 15:45
 * @desc       : 
-->

<script setup lang="ts">
  import { bitable } from '@lark-base-open/js-sdk';
  import Drawer from './Drawer.vue';
  import { InfoFilled, Upload } from '@element-plus/icons-vue';

  import { LOCAL_STORAGE_KEY } from '@/config/constant';

  import { Save } from '@icon-park/vue-next';

  import { v4 as uuidv4 } from 'uuid';

  // 新增视图抽屉
  const addViewDrawer = ref(false);

  async function addView() {
    drawerStatus.value = 'add';
    addViewDrawer.value = true;
  }

  import { useI18n } from 'vue-i18n';
  const { t } = useI18n();

  onMounted(() => {
    // // 检索存储的数据
    // const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

    // if (storedData) {
    //   // 已存在 key
    //   const retrievedArray = JSON.parse(storedData);
    //   methodList.value = retrievedArray;
    //   filterTableDataList.value = retrievedArray;
    // }

    getData();
  });

  const methodList = ref([]);

  function getData() {
    // 检索存储的数据
    const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

    if (storedData) {
      // 已存在 key
      const retrievedArray = JSON.parse(storedData);
      methodList.value = retrievedArray;
      filterTableDataList.value = retrievedArray;
    }
  }

  const confirmAddView = (item) => {
    // getData();

    if (drawerStatus.value === 'add') {
      // methodList.value.push(item);
      filterTableDataList.value.push(item);
    } else {
      // 新增则 push 数据, 编辑修改数据
      // methodList.value = methodList.value.map((_item) => {
      filterTableDataList.value = filterTableDataList.value.map((_item) => {
        if (_item.id === item.id) {
          _item = item;
        }
        return _item;
      });
    }
  };

  function cancelAddView() {
    addMethodItem.value = '';
  }

  const handleDelete = (index, id) => {
    getData();

    methodList.value.splice(index, 1);
    // filterTableDataList.value.splice(index, 1);

    // 检索存储的数据
    const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

    if (storedData) {
      // 已存在 key
      const retrievedArray = JSON.parse(storedData);
      const _filter = retrievedArray.filter((item) => item.id !== id);
      localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(_filter));

      methodList.value = _filter;
    }

    ElMessage({
      type: 'success',
      message: t('Deleted successfully'),
      duration: 1500,
      showClose: true,
    });
  };

  const selectList = ref([]);

  const handleSelectionChange = (val) => {
    selectList.value = val;
  };

  const isAddTable = ref();
  const addTableName = ref();

  const activeItem = ref();

  async function use(item) {
    isAddTable.value = true;
    activeItem.value = item;
  }

  const addMethodItem = ref();

  async function edit(item) {
    addMethodItem.value = item;
    drawerStatus.value = 'edit';
    addViewDrawer.value = true;
  }

  const popconfirmVisible = ref(false);

  function batchDelete() {
    if (selectList.value.length === 0) {
      ElMessage({
        type: 'warning',
        message: t('Please select the option to delete'),
        duration: 1500,
        showClose: true,
      });
      return;
    }

    popconfirmVisible.value = true;
  }

  function confirmBatchDelete() {
    getData();

    const difference = methodList.value.filter((method) => !selectList.value.some((select) => select.id === method.id));

    methodList.value = difference;
    filterTableDataList.value = difference;

    // 更新存储的数据
    localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(difference));

    ElMessage({
      type: 'success',
      message: t('Batch Delete Success'),
      duration: 1500,
      showClose: true,
    });

    popconfirmVisible.value = false;
  }

  const loading = ref(false);
  async function confirm() {
    if (!addTableName.value) {
      ElMessage({
        type: 'error',
        message: t('Please fill in the name of the data sheet'),
        duration: 1500,
        showClose: true,
      });
      return;
    }

    const tableMetaList = await bitable.base.getTableMetaList();

    // FIXME 数据表名字验重
    const index = tableMetaList.findIndex((item) => item.name === addTableName.value);
    if (index === -1) {
      loading.value = true;
      isAddTable.value = false;

      const { tableId } = await bitable.base.addTable({
        name: addTableName.value,
        fields: [],
      });

      const table = await bitable.base.getTableById(tableId);
      let _list = [];
      for (let i = 0; i < activeItem.value.lineNumber; i++) {
        _list.push({
          fields: {},
        });
      }

      await table.addRecords(_list);

      const fieldIdList = await table.getFieldIdList();
      const field = await table.getFieldById(fieldIdList[0]);

      // 修改首列
      await table.setField(field.id, {
        name: activeItem.value?.list[0]?.name,
        type: activeItem.value?.list[0]?.type,
      });

      ElMessage({
        type: 'success',
        message: t('New data table added successfully'),
        duration: 1500,
        showClose: true,
      });

      // 移动到新数据表位置
      await bitable.ui.switchToTable(tableId);

      loading.value = false;

      // 新增字段列
      const list = activeItem.value.list;

      for (const [index, item] of list.entries()) {
        if (index === 0) {
          continue; // 跳过索引 0
        }

        await table.addField({ type: item.type, name: item.name });
      }

      addTableName.value = '';
    } else {
      ElMessage({
        type: 'error',
        message: t('ta'),
        duration: 1500,
        showClose: true,
      });
    }
  }

  function cancel() {
    isAddTable.value = false;
    addTableName.value = '';
  }

  const drawerStatus = ref('add');

  const methodName = ref('');

  const filterTableDataList = ref();

  function search() {
    // 先重新获取全部数据
    filterTableDataList.value = methodList.value;

    // 筛选插件信息
    filterTableDataList.value = filterTableDataList.value.filter((item) => {
      let _name = item.name;
      const nameMatch = !methodName.value || _name?.includes(methodName.value);
      return nameMatch;
    });

    ElMessage({
      type: 'success',
      message: t('Successful query'),
      duration: 1500,
      showClose: true,
    });
  }

  function reset() {
    methodName.value = '';
    filterTableDataList.value = methodList.value;
  }

  const isSaveMethod = ref(false);

  const newMethodName = ref('');

  let supportedFieldList = [
    1, 2, 3, 4, 5, 7, 11, 13, 15, 17, 20, 22, 23, 1001, 1002, 1003, 1004, 1005, 99001, 99002, 99003, 99004, 99005,
  ];

  async function confirmSave() {
    if (!newMethodName.value) {
      ElMessage({
        type: 'error',
        message: t('Please fill in the name of the program'),
        duration: 1500,
        showClose: true,
      });
      return;
    }

    const table = await bitable.base.getActiveTable();
    const view = await table.getActiveView();
    const fieldMetaList = await view.getFieldMetaList();

    // FIXME 获取当前视图所有字段, 查找引用等不支持的字段使用文本字段替代
    const _list = fieldMetaList.map((item) => ({
      type: supportedFieldList.includes(item.type) ? item.type : 1,
      name: item.name,
    }));

    const index = methodList.value.findIndex((item) => item.name === newMethodName.value);
    if (index === -1) {
      let item = {
        // 处理 id
        id: uuidv4(),
        name: newMethodName.value,
        list: _list,
        lineNumber: 10,
      };

      // 检索存储的数据
      const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

      if (storedData) {
        // 已存在 key
        let retrievedArray = JSON.parse(storedData);

        retrievedArray.push(item);

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(retrievedArray));
      } else {
        // 不存在存在 key

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify([item]));
      }

      ElMessage({
        type: 'success',
        message: `${t('Added successfully')}`,
        duration: 1500,
        showClose: true,
      });

      isSaveMethod.value = false;
      newMethodName.value = '';
      getData();
    } else {
      ElMessage({
        type: 'error',
        message: t('Program name already exis'),
        duration: 1500,
        showClose: true,
      });
    }
  }

  function cancelSave() {
    isSaveMethod.value = false;
    newMethodName.value = '';
  }

  function saveMethod() {
    isSaveMethod.value = true;
  }

  // 下载 json 文件
  function download(item) {
    // 将数据转换为 JSON 格式
    let jsonData = JSON.stringify(item, null, 2);

    // 创建一个 Blob 对象
    let blob = new Blob([jsonData], { type: 'application/json' });

    // 创建一个链接
    let url = URL.createObjectURL(blob);

    // 创建一个链接元素
    let a = document.createElement('a');
    a.href = url;
    a.download = `${item.name}`; // 文件名

    // 模拟点击下载链接
    a.click();

    // 释放资源
    URL.revokeObjectURL(url);
  }

  const isUpload = ref(false);
  function upload() {
    isUpload.value = true;
  }

  function cancelImport() {
    isUpload.value = false;
  }

  // 上传之前的处理
  function beforeUpload(file) {
    // 检查文件类型
    const isJSON = file.type === 'application/json';
    if (!isJSON) {
      ElMessage.error(t('Only supports uploading JSON files'));
    }
    return isJSON;
  }

  const importMethod = ref();

  const importMethodName = ref();

  // 上传成功后的处理
  function handleUploadSuccess(response, file) {
    // 获取上传的文件名
    const fileName = file[0].name;

    // 截取以 .json 结尾的前面的数据
    const lastIndex = fileName.lastIndexOf('.json');
    const truncatedFileName = fileName.slice(0, lastIndex);

    // 获取上传的 JSON 文件
    const reader = new FileReader();

    reader.onload = (e) => {
      // 将文件内容解析为 JavaScript 对象
      importMethod.value = JSON.parse(e.target.result as string);
      importMethodName.value = truncatedFileName;

      ElMessage.success(t('File uploaded successfully'));
    };

    reader.readAsText(file[0].raw);
  }

  const uploadRef = ref();
  async function confirmImport() {
    if (!importMethodName.value) {
      ElMessage({
        type: 'error',
        message: t('Please fill in the name of the program'),
        duration: 1500,
        showClose: true,
      });
      return;
    }

    const index = methodList.value.findIndex((item) => item.name === importMethodName.value);
    if (index === -1) {
      let item = {
        // 处理 id
        id: uuidv4(),
        name: importMethodName.value,
        list: importMethod.value.list,
        lineNumber: importMethod.value.lineNumber,
      };

      // 检索存储的数据
      const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

      if (storedData) {
        // 已存在 key
        let retrievedArray = JSON.parse(storedData);

        retrievedArray.push(item);

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(retrievedArray));
      } else {
        // 不存在存在 key

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify([item]));
      }

      importMethodName.value = '';
      importMethod.value = [];
      isUpload.value = false;

      ElMessage({
        type: 'success',
        message: `${t('Added successfully')}`,
        duration: 1500,
        showClose: true,
      });

      uploadRef.value.clearFiles();

      getData();
    } else {
      ElMessage({
        type: 'error',
        message: t('Program name already exis'),
        duration: 1500,
        showClose: true,
      });
    }
  }

  const collapse = ref('0');
</script>

<template>
  <!-- <div class="tip">
    <div class="tip-text tip-title">操作步骤:</div>
    <div class="tip-text">1. 新增方案: 配置方案名字和字段列表信息</div>
    <div class="tip-text">2. 点击"运行"按钮，输入表名，生成对应方案数据表</div>
    <div class="tip-text">3. 点击"编辑"按钮，修改选定方案名称和字段列表</div>
  </div> -->

  <el-collapse
    v-model="collapse"
    class="collapse"
  >
    <!-- FIXME 筛选 -->
    <el-collapse-item name="1">
      <template #title>
        <el-icon class="mr5"><InfoFilled /></el-icon>
        <span class="collapse-title">{{ $t('More operational information') }}</span>
      </template>
      <div
        class="tip"
        style="margin-bottom: 0"
      >
        <div
          class="tip-text"
          style="margin-bottom: 0"
        >
          {{ $t('11') }}
        </div>
        <div
          class="tip-text"
          style="margin-bottom: 0"
        >
          {{ $t('22') }}
        </div>
        <div
          class="tip-text"
          style="margin-bottom: 0"
        >
          {{ $t('33') }}
        </div>
        <div
          class="tip-text"
          style="margin-bottom: 0"
        >
          {{ $t('44') }}
        </div>
      </div>
    </el-collapse-item>
  </el-collapse>

  <div
    v-loading="loading"
    :element-loading-text="$t('loading')"
  >
    <div class="button mt0">
      <el-button
        type="primary"
        @click="saveMethod"
      >
        <Save
          theme="outline"
          size="16"
          style="margin-right: 5px"
        />
        <span>{{ $t('Save Current View Field Scheme') }}</span>
      </el-button>
    </div>

    <div class="button mt0">
      <el-button
        type="primary"
        @click="addView"
      >
        <el-icon><Plus /></el-icon>
        <span>{{ $t('Additional Programs') }}</span>
      </el-button>

      <el-button
        type="info"
        @click="upload"
      >
        <el-icon><Upload /></el-icon>
        <span>{{ $t('upload Programs') }}</span>
      </el-button>
    </div>

    <el-divider />

    <div class="addView-line">
      <div class="addView-line-label">{{ $t('Program Name') }}</div>
      <el-input
        style="width: 60%"
        v-model="methodName"
        clearable
        :placeholder="$t('Please enter the name of the program')"
        @keydown.enter="search"
      />
    </div>

    <div class="button">
      <el-button
        type="primary"
        @click="search"
      >
        <el-icon><Search /></el-icon>
        <span>{{ $t('search') }}</span>
      </el-button>

      <el-button
        type="info"
        @click="reset"
      >
        <el-icon><Refresh /></el-icon>
        <span>{{ $t('rs') }}</span>
      </el-button>
    </div>

    <div class="view-table">
      <div
        class="total-text"
        v-if="filterTableDataList?.length >= 1"
      >
        {{ $t('zong-shu-filtertabledatalistlength-ge', [filterTableDataList?.length]) }}
      </div>
      <!-- <div
        v-show="loading"
        class="total-text"
      ></div> -->
      <el-table
        ref="tableRef"
        @selection-change="handleSelectionChange"
        :data="filterTableDataList"
        :empty-text="$t('No Data')"
        max-height="55vh"
      >
        <el-table-column
          v-show="filterTableDataList?.length > 0"
          type="selection"
          width="30"
        />

        <el-table-column :label="$t('Program name')">
          <template #default="scope">
            <div :title="scope.row.name">
              <div>{{ scope.row.name }}</div>
            </div>
          </template>
        </el-table-column>

        <el-table-column
          property="name"
          :label="$t('operation')"
          align="center"
        >
          <template #default="scope">
            <div class="operation">
              <div
                @click="download(scope.row)"
                :title="$t('download method')"
                style="color: rgb(20, 86, 240)"
              >
                <el-icon
                  size="20"
                  class="cursor"
                  ><Download
                /></el-icon>
              </div>
              <div
                @click="use(scope.row)"
                :title="$t('Add Data Sheet')"
                style="color: rgb(20, 86, 240)"
              >
                <el-icon
                  size="20"
                  class="cursor"
                  ><VideoPlay
                /></el-icon>
              </div>
              <div
                @click="edit(scope.row)"
                :title="$t('Editorial program')"
              >
                <el-icon
                  size="20"
                  class="cursor"
                  ><Edit
                /></el-icon>
              </div>

              <div
                @click="handleDelete(scope.$index, scope.row.id)"
                :title="$t('Delete program')"
                style="color: #f54a45"
              >
                <el-icon
                  size="20"
                  class="cursor"
                  ><Delete
                /></el-icon>
              </div>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div
      class="select-text"
      v-if="filterTableDataList?.length >= 1"
    >
      {{ $t('yi-xuan-selectlistlength-ge', [selectList?.length]) }}
    </div>

    <div class="delete-button">
      <!-- <el-button
        @click="batchDelete"
        type="danger"
        color="#F54A45"
      >
        <el-icon><Delete /></el-icon>
        <span>{{ $t('batch deletion') }}</span>
      </el-button> -->

      <el-popconfirm
        v-if="filterTableDataList?.length > 0"
        :visible.sync="popconfirmVisible"
        width="60vw"
        :confirm-button-text="$t('confirm')"
        :cancel-button-text="$t('cancel')"
        @confirm="confirmBatchDelete"
        @cancel="() => (popconfirmVisible = false)"
        :icon="InfoFilled"
        icon-color="rgb(20, 86, 240)"
        cancel-button-type="info"
        :hide-after="50"
        :title="$t('Confirm deletion of the selected', [selectList.length])"
      >
        <template #reference>
          <el-button
            @mousedown="(e) => e.preventDefault()"
            @click="batchDelete"
            type="danger"
            color="#F54A45"
          >
            <el-icon><Delete /></el-icon>
            <span>{{ $t('batch deletion') }}</span>
          </el-button>
        </template>
      </el-popconfirm>
    </div>
  </div>

  <!-- FIXME 新建数据表 -->
  <el-dialog
    v-model="isAddTable"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    @close="cancel"
    :title="$t('Add Data Sheet')"
    width="75%"
  >
    <div class="addView">
      <div class="addView-line">
        {{ $t('Current Programs')
        }}<span style="color: rgb(20, 86, 240); font-weight: 500"> {{ activeItem.name }}</span>
      </div>
      <div class="addView-line">
        <div class="addView-line-label addView-line-labelDialog">{{ $t('data table name') }}</div>
        <el-input
          clearable
          v-model="addTableName"
          :placeholder="$t('Please enter a data table name')"
        />
      </div>

      <div>
        <el-button
          type="primary"
          @click="confirm"
          >{{ $t('confirm') }}</el-button
        >

        <el-button
          type="info"
          @click="cancel"
          >{{ $t('cancel') }}</el-button
        >
      </div>
    </div>
  </el-dialog>

  <!-- FIXME 保存当前视图字段方案 -->
  <el-dialog
    v-model="isSaveMethod"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    @close="cancel"
    :title="$t('Save Current View Field Scheme')"
    width="75%"
  >
    <div class="addView">
      <div class="addView-line">
        <div class="addView-line-label addView-line-labelDialog">{{ $t('Program Name') }}</div>
        <el-input
          clearable
          v-model="newMethodName"
          :placeholder="$t('Please enter the name of the program')"
        />
      </div>

      <div>
        <el-button
          type="primary"
          @click="confirmSave"
          >{{ $t('confirm') }}</el-button
        >

        <el-button
          type="info"
          @click="cancelSave"
          >{{ $t('cancel') }}</el-button
        >
      </div>
    </div>
  </el-dialog>

  <!-- 导入方案 -->
  <el-dialog
    v-model="isUpload"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    @close="cancelImport"
    :title="$t('upload Programs')"
    width="75%"
  >
    <div class="addView">
      <el-upload
        ref="uploadRef"
        drag
        :http-request="() => {}"
        :on-change="handleUploadSuccess"
        :before-upload="beforeUpload"
        :limit="1"
      >
        <el-icon class="el-icon--upload"><upload-filled /></el-icon>
        <div class="el-upload__text">
          <em>{{ $t('drag') }}</em> {{ $t('or') }} <em>{{ $t('Click to upload a file') }}</em>
        </div>
      </el-upload>

      <div class="addView-line">
        <div class="addView-line-label addView-line-labelDialog">{{ $t('Program Name') }}</div>
        <el-input
          clearable
          v-model="importMethodName"
          :placeholder="$t('Please enter the name of the program')"
        />
      </div>

      <div>
        <el-button
          type="primary"
          @click="confirmImport"
          >{{ $t('confirm') }}</el-button
        >

        <el-button
          type="info"
          @click="cancelImport"
          >{{ $t('cancel') }}</el-button
        >
      </div>
    </div>
  </el-dialog>

  <Drawer
    v-model:model-value="addViewDrawer"
    :method-list="methodList"
    :addMethodItem="addMethodItem"
    :drawerStatus="drawerStatus"
    @confirmAddView="confirmAddView"
    @cancel-add-view="cancelAddView"
  />
</template>

<style scoped>
  .home {
    font-size: 14px;
  }

  .delete-button {
    margin-top: 10px;
  }

  .addView {
    margin: 10px 0;
  }

  .addView-line {
    margin-bottom: 10px;
    display: flex;
    align-items: center;
    .addView-line-label {
      margin-right: 10px;
      font-size: 14px;
      white-space: nowrap;
    }
  }

  .operation {
    display: flex;

    div {
      margin-right: 12px;
    }
  }

  .cursor {
    &:hover {
      cursor: pointer;
    }
  }

  .tip {
    color: #8f959e;
    font-size: 12px;
    margin-bottom: 14px;
    .tip-text {
      margin-bottom: 6px;
    }

    .tip-info {
      margin-top: 12px;
    }

    .tip-title {
      font-size: 14px;
      margin-bottom: 8px;
    }
  }

  .button {
    margin: 14px 0;
  }

  .mt0 {
    margin-top: 5px !important;
  }

  .total-text {
    height: 14px;
    line-height: 14px;
    font-size: 14px;
  }

  .select-text {
    margin-top: 5px;
    font-size: 14px;
    height: 14px;
    line-height: 14px;
  }

  .el-divider--horizontal {
    margin: 10px 0;
  }

  .collapse {
    margin-bottom: 14px;
  }

  .mr5 {
    margin-right: 5px;
  }
</style>
