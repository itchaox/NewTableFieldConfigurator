<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-12-16 09:57
 * @LastAuthor : itchaox
 * @LastTime   : 2024-01-25 22:27
 * @desc       : 抽屉
-->

<script setup lang="ts">
  import { bitable } from '@lark-base-open/js-sdk';
  import { Close } from '@element-plus/icons-vue';
  import FieldIcon from './fieldIcon.jsx';
  import { v4 as uuidv4 } from 'uuid';
  import { LOCAL_STORAGE_KEY } from '@/config/constant';

  const base = bitable.base;

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
      name: '文本',
      type: 1,
    },
    {
      name: '数字',
      type: 2,
    },
    {
      name: '单选',
      type: 3,
    },
    {
      name: '多选',
      type: 4,
    },
    {
      name: '日期时间',
      type: 5,
    },
    {
      name: '复选框',
      type: 7,
    },
    {
      name: '用户',
      type: 11,
    },
    {
      name: '电话',
      type: 13,
    },
    {
      name: '网址',
      type: 15,
    },
    {
      name: '附件',
      type: 17,
    },
    // {
    //   name: '单链接',
    //   type: 18,
    // },
    // {
    //   name: '查找',
    //   type: 19,
    // },
    {
      name: '公式',
      type: 20,
    },
    // {
    //   name: '双向链接',
    //   type: 21,
    // },
    {
      name: '位置',
      type: 22,
    },
    {
      name: '群聊',
      type: 23,
    },
    {
      name: '创建时间',
      type: 1001,
    },
    {
      name: '修改时间',
      type: 1002,
    },
    {
      name: '创建用户',
      type: 1003,
    },
    {
      name: '修改用户',
      type: 1004,
    },
    {
      name: '自动编号',
      type: 1005,
    },
    {
      name: '条形码',
      type: 99001,
    },
    {
      name: '进度',
      type: 99002,
    },
    {
      name: '货币',
      type: 99003,
    },
    {
      name: '评分',
      type: 99004,
    },
    {
      name: '电子邮件',
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
        message: '请填写方案名字',
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
        message: `${props.drawerStatus === 'add' ? '新增' : '编辑'}成功`,
        duration: 1500,
        showClose: true,
      });
    } else {
      ElMessage({
        type: 'error',
        message: '方案名字已存在,请重新输入!',
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

  watch(
    () => props.addMethodItem,
    (newValue, oldValue) => {
      addMethodName.value = newValue?.name;
      filterList.value = newValue?.list || [];
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
          :id="titleId"
          v-if="drawerStatus === 'add'"
        >
          新增方案
        </div>
        <div
          :id="titleId"
          v-else
        >
          编辑方案
        </div>
        <el-button
          type="danger"
          @click="close"
        >
          <el-icon class="el-icon--left"><CircleCloseFilled /></el-icon>
          关闭
        </el-button>
      </template>

      <div class="addView">
        <div class="addView-line">
          <div class="addView-line-label theme-view-text-color">方案名字:</div>
          <el-input
            style="width: 60%"
            clearable
            v-model="addMethodName"
            placeholder="请输入方案名字"
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
              <span class="collapse-title">配置字段信息</span>
              <span
                v-if="filterList?.length > 0"
                style="color: #5c82f3"
                >（{{ filterList?.length }}）</span
              >
            </template>

            <div class="collapse-line-list">
              <div
                class="collapse-line"
                v-for="(item, index) in filterList"
                :key="index"
              >
                <!-- 字段名 -->
                <el-select
                  v-model="item.type"
                  filterable
                  style="width: 50%"
                >
                  <el-option
                    v-for="(field, i) in filterFieldList"
                    :key="i"
                    :label="field.name"
                    :title="field.name"
                    :value="field.type"
                  >
                    <field-icon :fieldType="field.type" />
                    <span>
                      {{ field.name }}
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
                      placeholder="请输入字段名字"
                    />
                  </div>
                  <el-button
                    :icon="Close"
                    class="collapse-delete"
                    @click="() => filterList.splice(index, 1)"
                    text
                  />
                </div>
              </div>
            </div>
            <el-button
              text
              @click="addFilter"
            >
              <el-icon><Plus /></el-icon>添加字段
            </el-button>
          </el-collapse-item>
        </el-collapse>

        <div>
          <el-button
            type="primary"
            @click="confirmAddView"
            >确定</el-button
          >

          <el-button
            type="info"
            @click="cancel"
            >取消</el-button
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
    max-height: 60vh;
    overflow: scroll;
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

  .collapse-delete {
    padding: 5px;
    height: 24px;
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
</style>
