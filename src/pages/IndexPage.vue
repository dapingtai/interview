<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn
          color="primary"
          class="q-mt-md"
          @click="addItem"
          @keyup.enter="addItem"
        >
          新增
        </q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <slot v-if="editIndex === props.pageIndex">
              <q-td
                v-for="col in props.cols"
                :key="col.name"
                :props="props"
                style="min-width: 120px"
              >
                <q-input
                  v-model="editData[col.name]"
                  input-class="text-center"
                  :type="col.type"
                  borderless
                />
              </q-td>
            </slot>
            <slot v-else>
              <q-td
                v-for="col in props.cols"
                :key="col.name"
                :props="props"
                style="min-width: 120px"
              >
                <div>{{ col.value }}</div>
              </q-td>
            </slot>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <slot v-if="props.rowIndex === editIndex">
                <q-btn
                  @click="handleClickOption(btn, props.row, props.rowIndex)"
                  v-for="(btn, index) in editButtons"
                  :key="index"
                  size="sm"
                  color="grey-6"
                  round
                  dense
                  :icon="btn.icon"
                  class="q-ml-md"
                  padding="5px 5px"
                >
                  <q-tooltip
                    transition-show="scale"
                    transition-hide="scale"
                    anchor="top middle"
                    self="bottom middle"
                    :offset="[10, 10]"
                  >
                    {{ btn.label }}
                  </q-tooltip>
                </q-btn>
              </slot>
              <slot v-else>
                <q-btn
                  @click="handleClickOption(btn, props.row, props.rowIndex)"
                  v-for="(btn, index) in tableButtons"
                  :key="index"
                  size="sm"
                  color="grey-6"
                  round
                  dense
                  :icon="btn.icon"
                  class="q-ml-md"
                  padding="5px 5px"
                >
                  <q-tooltip
                    transition-show="scale"
                    transition-hide="scale"
                    anchor="top middle"
                    self="bottom middle"
                    :offset="[10, 10]"
                  >
                    {{ btn.label }}
                  </q-tooltip>
                </q-btn>
              </slot>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
      <div class="full-width flex justify-center h-100">
        <q-btn color="secondary" class="q-mt-md" @click="postData">
          送出表單
        </q-btn>
      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { QTableProps } from 'quasar';
import { onMounted, ref } from 'vue';
import { api } from 'src/boot/axios';

interface btnType {
  label: string;
  icon: string;
  status: string;
}
interface blockItemType {
  id: string;
  name: string;
  age: number | null;
}

const blockData = ref<blockItemType[]>([
  {
    id: 'test-1',
    name: 'test',
    age: 25,
  },
]);

const editIndex = ref<number>(-1);

const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
    type: 'text',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
    type: 'number',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const editButtons = ref([
  {
    label: '儲存',
    icon: 'save',
    status: 'save',
  },
  {
    label: '取消',
    icon: 'cancel',
    status: 'cancel',
  },
]);

const tempData = ref<blockItemType>({
  id: '',
  name: '',
  age: null,
});

const editData = ref<blockItemType>({
  id: '',
  name: '',
  age: null,
});

const clearItem = (): void => {
  tempData.value.name = '';
  tempData.value.age = null;
};

const addItem = (): void => {
  blockData.value.push(JSON.parse(JSON.stringify(tempData.value)));
  clearItem();
};

const fetchData = async (): Promise<void> => {
  const { data } = await api.get('/CRUDTest/a');
  blockData.value = data;
};

const postData = async (): Promise<void> => {
  await api.post('/CRUDTest', blockData.value);
};

const patchData = async (item: blockItemType): Promise<void> => {
  await api.patch('/CRUDTest', item);
};

const deleteData = async (id: string): Promise<void> => {
  await api.delete(`/CRUDTest/${id}`);
};

function handleClickOption(btn: btnType, data: blockItemType, index: number) {
  console.log(btn, data, index);
  switch (btn.status) {
    case 'edit':
      editIndex.value = index;
      editData.value = data;
      break;
    case 'delete':
      deleteData(data.id);
      blockData.value = blockData.value.filter(
        (item: blockItemType) => item.id !== data.id
      );
      break;
    case 'save':
      editIndex.value = -1;
      patchData(data);
    case 'cancel':
      editIndex.value = -1;
      editData.value = {
        id: '',
        name: '',
        age: null,
      };
  }
}

onMounted(() => {
  fetchData();
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
