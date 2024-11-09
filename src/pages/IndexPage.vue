<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input
          v-model="tempData.age"
          label="年齡"
          type="number"
        />
        <q-btn color="primary" class="q-mt-md" @click="submitData">{{ btnName }}</q-btn>
      </div>

      <q-input v-model="search" label="使用字串模糊搜尋" outlined dense />

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
        v-model:sort-method="sorted"
        :filter="search"
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
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
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
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { ref, onMounted } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

const $q = useQuasar(); // 引入 Quasar 插件
const blockData = ref([]);
const tableConfig = ref([
  { 
    label: '姓名', 
    name: 'name', 
    field: 'name', 
    align: 'left',
    sortable: true
  },{ 
    label: '年齡', 
    name: 'age', 
    field: 'age', 
    align: 'left',
    sortable: true
  },
]);

const tableButtons = ref([
  { 
    label: '編輯', 
    icon: 'edit', 
    status: 'edit' 
  },{ 
    label: '刪除', 
    icon: 'delete', 
    status: 'delete' },
]);

const tempData = ref({
  id: '',
  name: '',
  age: '',
});

const btnName = ref("新增");

const search = ref(''); // 綁定搜尋字串

function handleClickOption(btn: btnType, data: any) {
  if (btn.label === '刪除') {
    showDeleteDialog(data);
  } else {
    btnName.value = "更新";
    tempData.value = { id: data.id, name: data.name, age: data.age };
  }
}

// 彈出確認刪除對話框
function showDeleteDialog(data: any) {
  $q.dialog({
    title: '刪除確認',
    message: '是否確定刪除該筆資料？',
    ok: {
      label: '確定',
      color: 'primary',
    },
    cancel: {
      label: '取消',
      color: 'negative',
    },
    persistent: true,
  }).onOk(() => {
    deleteData(data.id);
  }).onCancel(() => {
    console.log('刪除操作已取消');
  });
}

// 刪除資料函式
function deleteData(id: string) {
  axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`)
    .then(() => {
      console.log('資料刪除成功！');
      reloadData(); // 刪除成功後重新載入資料
    })
    .catch(error => {
      console.error('刪除失敗', error);
    });
}

const submitData = () => {
  if (!tempData.value.name) {
    alert('名稱不可以是空白！');
    return;
  }
  if (!/^\d+$/.test(tempData.value.age)) {
    tempData.value.age = '';
    alert('年齡請輸入正整數');
    return;
  }

  if (btnName.value === '新增') {
    postData();
  } else {
    patchData();
  }
};

// 新增資料函式
function postData() {
  axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
    .then(() => {
      console.log('成功新增資料');
      resetForm();
      reloadData();
    })
    .catch(error => console.error('新增失敗', error));
}

// 更新資料函式
function patchData() {
  axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', {
      id: tempData.value.id,
      name: tempData.value.name,
      age: tempData.value.age,
    })
    .then(() => {
      console.log('資料已更新');
      resetForm();
      reloadData();
    })
    .catch(error => console.error('更新失敗', error));
}

// 重置表單函式
function resetForm() {
  tempData.value = { id: '', name: '', age: '' };
  btnName.value = '新增';
}

// 重新加載資料
function reloadData() {
  axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    .then(response => {
      blockData.value = response.data;
    })
    .catch(error => console.error('載入資料失敗', error));
}

onMounted(() => reloadData());
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
