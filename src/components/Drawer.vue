<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-12-16 09:57
 * @LastAuthor : itchaox
 * @LastTime   : 2024-02-02 15:46
 * @desc       : 抽屉
-->

<script setup lang="ts">
  import { bitable } from '@lark-base-open/js-sdk';
  import { Close } from '@element-plus/icons-vue';
  import FieldIcon from './fieldIcon.jsx';
  import { v4 as uuidv4 } from 'uuid';
  import { LOCAL_STORAGE_KEY } from '@/config/constant';

  import { Drag, Lock } from '@icon-park/vue-next';

  import { VueDraggable } from 'vue-draggable-plus';

  const base = bitable.base;

  import _ from 'lodash';

  import { useI18n } from 'vue-i18n';
  const { t } = useI18n();

  interface Props {
    modelValue: Boolean;
    methodList: any[];
    addMethodItem?: any;
    drawerStatus?: any;
  }

  const props = defineProps<Props>();

  const emits = defineEmits(['update:model-value', 'confirmAddView', 'cancelAddView']);

  let table;
  let view;

  // 字段列表
  const fieldList = ref([]);

  // 筛选的字段列表
  // FIXME 支持大部分常见字段
  const filterFieldList = ref([
    {
      name: 'Text',
      type: 1,
    },
    {
      name: 'Number',
      type: 2,
    },
    {
      name: 'single',
      type: 3,
    },
    {
      name: 'multiple',
      type: 4,
    },
    {
      name: 'date',
      type: 5,
    },
    {
      name: 'checkbox',
      type: 7,
    },
    {
      name: 'User',
      type: 11,
    },
    {
      name: 'Phone',
      type: 13,
    },
    {
      name: 'Url',
      type: 15,
    },
    {
      name: 'attachment',
      type: 17,
    },
    {
      name: 'formula',
      type: 20,
    },
    {
      name: 'location',
      type: 22,
    },
    {
      name: 'group chat',
      type: 23,
    },
    {
      name: 'Creation time',
      type: 1001,
    },
    {
      name: 'Modify Time',
      type: 1002,
    },
    {
      name: 'Create User',
      type: 1003,
    },
    {
      name: 'Modify User',
      type: 1004,
    },
    {
      name: 'auto-numbering',
      type: 1005,
    },
    {
      name: 'barcodes',
      type: 99001,
    },
    {
      name: 'speed',
      type: 99002,
    },
    {
      name: 'money',
      type: 99003,
    },
    {
      name: 'score',
      type: 99004,
    },
    {
      name: 'Email',
      type: 99005,
    },
  ]);

  const firstFilterFieldList = ref([
    {
      name: 'Text',
      type: 1,
    },
    {
      name: 'Number',
      type: 2,
    },
    {
      name: 'date',
      type: 5,
    },
    {
      name: 'Phone',
      type: 13,
    },
    {
      name: 'Url',
      type: 15,
    },
    {
      name: 'formula',
      type: 20,
    },
    {
      name: 'location',
      type: 22,
    },
    {
      name: 'barcodes',
      type: 99001,
    },
    {
      name: 'speed',
      type: 99002,
    },
    {
      name: 'money',
      type: 99003,
    },
    {
      name: 'auto-numbering',
      type: 1005,
    },
    {
      name: 'Email',
      type: 99005,
    },
  ]);

  onMounted(async () => {
    init();
  });

  base.onSelectionChange(async (event) => {
    init();
  });

  // 文本类字段的集合
  // 1 文本; 13 电话号码; 15 超链接; 22 地理位置; 99001 条形码; 99005 Email
  const textMap = ref([1, 13, 15, 22, 99001]);

  // 数字类字段的集合
  // 2 数字; 1005 自动编号; 99002 进度; 99003 货币; 99004 评分
  const numberMap = ref([2, 1005, 99002, 99003, 99004]);

  // 选择类字段的集合
  // 3 单选; 4多选;
  const selectMap = ref([3, 4]);

  // 引用类型
  // 11 人员; 18 单向关联; 23 群组;

  // 日期类型
  // 5 日期; 1001 创建时间; 1002 修改时间

  async function init() {
    table = await base.getActiveTable();
    view = await table.getActiveView();
    // fieldList.value = await view.getFieldMetaList();

    // filterFieldList.value = fieldList.value;

    // filterFieldList.value = fieldList.value.filter((item) =>
    //   [1, 3, 4, 13, 15, 22, 99001, 2, 1005, 99002, 99003, 99004].includes(item.type),
    // );
  }

  const addMethodName = ref();

  const addViewType = ref(1);

  /**
   * @desc  : 确认新增/修改方案
   */
  async function confirmAddView() {
    if (!addMethodName.value) {
      ElMessage({
        type: 'error',
        message: t('Please fill in the name of the program'),
        duration: 1500,
        showClose: true,
      });
      return;
    }

    if (filterList.value.length === 0) {
      ElMessage({
        type: 'error',
        message: t('Please add a list of fields'),
        duration: 1500,
        showClose: true,
      });
      return;
    }

    const index = props.methodList
      .filter((i) => i.name !== props.addMethodItem?.name)
      .findIndex((item) => item.name === addMethodName.value);
    if (index === -1) {
      // addMethodName.value = '';
      // addViewType.value = 1;
      // emits('update:model-value', false);

      let item = {
        // 处理 id
        id: props.drawerStatus === 'add' ? uuidv4() : props.addMethodItem?.id,
        name: addMethodName.value,
        list: filterList.value,
        lineNumber: lineNumber.value,
      };

      // 检索存储的数据
      const storedData = localStorage.getItem(LOCAL_STORAGE_KEY);

      if (storedData) {
        // 已存在 key
        let retrievedArray = JSON.parse(storedData);

        if (props.drawerStatus === 'add') {
          retrievedArray.push(item);
        } else {
          // 新增则 push 数据, 编辑修改数据
          retrievedArray = retrievedArray.map((_item) => {
            if (_item.id === item.id) {
              _item = item;
            }
            return _item;
          });
        }

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(retrievedArray));
      } else {
        // 不存在存在 key

        // 更新数据
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify([item]));
      }

      reset();
      emits('confirmAddView', item);

      ElMessage({
        type: 'success',
        message: `${props.drawerStatus === 'add' ? t('Added successfully') : t('Edit Success')}`,
        duration: 1500,
        showClose: true,
      });
    } else {
      ElMessage({
        type: 'error',
        message: t('Program name already exis'),
        duration: 1500,
        showClose: true,
      });
    }
  }

  function cancel() {
    reset();
  }

  function reset() {
    emits('update:model-value', false);
    emits('cancelAddView');

    addMethodName.value = '';
    filterList.value = [];
  }

  // FIXME 筛选
  const filterList = ref([]);

  const lineNumber = ref(10);

  watch(
    () => props.addMethodItem,
    (newValue, oldValue) => {
      addMethodName.value = newValue?.name;
      lineNumber.value = newValue?.lineNumber || 10;
      filterList.value = _.cloneDeep(newValue?.list) || [];
    },
    {
      immediate: true,
      deep: true,
    },
  );

  const addFilter = () => {
    filterList.value.push({
      type: filterFieldList.value?.[0]?.type,
      name: '',
    });
  };

  // 折叠面板
  const collapse = ref('1');

  const el = ref();
</script>

<template>
  <div>
    <el-drawer
      :model-value="modelValue"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      @closed="cancel"
      :show-close="false"
      size="85%"
    >
      <template #header="{ close, titleId }">
        <div
          class="drawer-title"
          :id="titleId"
          v-if="drawerStatus === 'add'"
        >
          {{ $t('Additional Programs') }}
        </div>
        <div
          class="drawer-title"
          :id="titleId"
          v-else
        >
          {{ $t('Editorial program') }}
        </div>
        <el-button
          type="danger"
          @click="close"
        >
          <el-icon class="el-icon--left"><CircleCloseFilled /></el-icon>
          {{ $t('Close') }}
        </el-button>
      </template>

      <div class="addView">
        <div class="addView-line">
          <div class="addView-line-label theme-view-text-color">{{ $t('Program Name') }}</div>
          <el-input
            style="width: 60%"
            clearable
            v-model="addMethodName"
            :placeholder="$t('Please enter the name of the program')"
          />
        </div>

        <el-collapse
          v-model="collapse"
          class="collapse"
        >
          <!-- FIXME 筛选 -->
          <el-collapse-item name="1">
            <template #title>
              <el-icon><Filter /></el-icon>
              <span class="collapse-title">{{ $t('Configuring field information') }}</span>
              <span
                v-if="filterList?.length > 0"
                style="color: #5c82f3"
                >（{{ filterList?.length }}）</span
              >
            </template>

            <VueDraggable
              ref="el"
              v-model="filterList"
              animation="150"
              handle=".handle"
              class="collapse-line-list"
            >
              <!-- <div class="collapse-line-list"> -->
              <div
                class="collapse-line"
                v-for="(item, index) in filterList"
                :key="index"
              >
                <div class="drag">
                  <Lock
                    :title="$t('idx')"
                    class="lock"
                    v-if="index === 0"
                    theme="outline"
                    size="16"
                    fill="#333"
                  />

                  <Drag
                    v-else
                    class="handle cursor-move"
                    theme="outline"
                    size="16"
                    fill="#656a72"
                    strokeLinejoin="miter"
                    strokeLinecap="butt"
                  />

                  <field-icon :fieldType="item.type" />

                  <!-- 字段名 -->
                  <el-select
                    v-model="item.type"
                    filterable
                    style="width: 50%"
                  >
                    <el-option
                      v-for="(field, i) in index === 0 ? firstFilterFieldList : filterFieldList"
                      :key="i"
                      :label="$t(field.name)"
                      :title="field.name"
                      :value="field.type"
                    >
                      <field-icon :fieldType="field.type" />
                      <span>
                        {{ $t(field.name) }}
                      </span>
                    </el-option>
                  </el-select>

                  <div class="collapse-line-other">
                    <!-- 值 -->
                    <div
                      class="collapse-line-value"
                      style="width: 100%"
                    >
                      <!-- TODO 输入框数据验重 -->
                      <el-input
                        v-model="item.name"
                        :title="item.name"
                        :placeholder="$t('Please enter a field name')"
                      />
                    </div>
                    <div
                      v-if="index === 0"
                      class="btn-delete"
                    ></div>
                    <el-button
                      v-else
                      :icon="Close"
                      class="collapse-delete"
                      @click="() => filterList.splice(index, 1)"
                      text
                    />
                  </div>
                </div>
              </div>
              <!-- </div> -->
            </VueDraggable>
            <el-button
              text
              @click="addFilter"
            >
              <el-icon style="margin-right: 5px"><Plus /></el-icon>{{ $t('Adding Fields') }}
            </el-button>
          </el-collapse-item>
        </el-collapse>

        <div class="default-line">
          <div>{{ $t('Default number of rows') }}</div>
          <el-input-number
            v-model="lineNumber"
            :min="1"
            :max="1000"
          />
        </div>

        <div>
          <el-button
            type="primary"
            @click="confirmAddView"
            >{{ $t('confirm') }}</el-button
          >

          <el-button
            type="info"
            @click="cancel"
            >{{ $t('cancel') }}</el-button
          >
        </div>
      </div>
    </el-drawer>
  </div>
</template>

<style scoped>
  .addView {
    /* margin: 10px 0; */
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

    .addView-line-labelDialog {
      width: 125px;
    }
  }

  .collapse {
    margin: 20px 0;
    /* max-height: 60vh; */
    /* overflow: scroll; */
  }

  .collapse-title {
    margin-left: 5px;
  }

  .collapse-line-list {
    margin: 10px 0;
  }

  .collapse-line {
    display: flex;
    flex-wrap: nowrap;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
  }

  .collapse-line-filed {
    /* margin-right: 10px; */
  }

  .collapse-line-other {
    display: flex;
    justify-content: flex-end;
  }

  .collapse-line-value {
    margin: 0 5px;
  }

  .btn-delete {
    width: 30px;
  }

  .collapse-delete {
    padding: 5px;
  }

  .sync {
    display: flex;
    align-items: center;
    font-size: 14px;
    margin-bottom: 10px;
    span {
      margin-right: 5px;
    }
  }

  .view-name-icon {
    position: relative;
    top: 2px;
  }

  .default-line {
    display: flex;
    align-items: center;
    margin-bottom: 24px;
  }

  .drawer-title {
    color: rgb(20, 86, 240);
    font-weight: 500;
    font-size: 18px;
  }

  .drag {
    display: flex;
    justify-content: center;
    align-items: center;

    .handle,
    .lock {
      position: relative;
      padding-top: 5px;
      margin-right: 5px;
    }
  }

  .cursor-move {
    cursor: grab;
  }
</style>
