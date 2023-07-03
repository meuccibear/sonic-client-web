<script setup>
import {onMounted, ref} from 'vue';
import {useRoute} from 'vue-router';
import {ElMessage} from 'element-plus';

import {useI18n} from 'vue-i18n';
import axios from '../http/axios';

const {t: $t} = useI18n();

const route = useRoute();
const dialogVisible = ref(false);
const pageData = ref([]);
const updateGlobal = ref(null);
const accounts = ref({
  id: null,
  projectId: route.params.projectId,
  name: '',
  password: '',
  status: 1
});
const editaccounts = async (id) => {
  await open();
  await getGlobalInfo(id);
};
const robotList = [
  {name: 'Tiktok', value: 'Tiktok'},
];
const open = () => {
  accounts.value = {
    id: null,
    projectId: route.params.projectId,
    paramsKey: '',
    paramsValue: '',
  };
  dialogVisible.value = true;
};
const getaccountsList = () => {
  axios
      .get('/controller/accounts/list', {
        params: {
          projectId: route.params.projectId,
        },
      })
      .then((resp) => {
        pageData.value = resp.data;
      });
};
const getGlobalInfo = (id) => {
  axios
      .get('/controller/accounts', {
        params: {
          id,
        },
      })
      .then((resp) => {
        accounts.value = resp.data;
      });
};
const deleteGlobal = (id) => {
  axios
      .delete('/controller/accounts', {
        params: {
          id,
        },
      })
      .then((resp) => {
        if (resp.code === 2000) {
          ElMessage.success({
            message: resp.message,
          });
          getaccountsList();
        }
      });
};
const summit = () => {
  updateGlobal.value.validate((valid) => {
    if (valid) {
      axios.put('/controller/accounts', accounts.value).then((resp) => {
        if (resp.code === 2000) {
          ElMessage.success({
            message: resp.message,
          });
          dialogVisible.value = false;
          getaccountsList();
        }
      });
    }
  });
};
onMounted(() => {
  getaccountsList();
});
</script>

<template>
  <el-dialog
      v-model="dialogVisible"
      :title="$t(accounts.id ? 'accountsTs.dialogVisible.updateInfo' : 'accountsTs.dialogVisible.addInfo')"
      width="600px"
  >
    <!--    <el-alert-->
    <!--      style="margin-bottom: 10px"-->
    <!--      :title="$t('accountsTs.dialogVisible.specialUse')"-->
    <!--      :description="$t('accountsTs.dialogVisible.messageInfo')"-->
    <!--      type="info"-->
    <!--      show-icon-->
    <!--      close-text="Get!"-->
    <!--    />-->
    <el-form
        ref="updateGlobal"
        :model="accounts"
        size="small"
        class="demo-table-expand"
        label-width="90px"
        label-position="left"
    >
      <el-form-item
          prop="appName"
          label="App"
          :rules="{
          required: true,
          message: $t('accountsTs.dialogVisible.valueNameMessage'),
          trigger: 'blur',
        }"
      >
        <el-select
            v-model="accounts.appName"
            style="width: 100%"
            :placeholder="$t('robot.robotTypePlaceholder')"
        >
          <el-option
              v-for="item in robotList"
              :key="item.name"
              :value="item.value"
              :label="item.name"
              :disabled="item['disabled']"
          >
            <div style="display: flex; align-items: center">
              <!--              <el-avatar-->
              <!--                  style="margin-right: 10px"-->
              <!--                  :size="30"-->
              <!--                  :src="getImg(item.img)"-->
              <!--              ></el-avatar>-->
              {{ item.name }}
            </div>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item
          prop="name"
          :label="$t('accountsTs.dialogVisible.name')"
          :rules="{
          required: true,
          message: $t('accountsTs.dialogVisible.valueNameMessage'),
          trigger: 'blur',
        }"
      >
        <el-input
            v-model="accounts.name"
            :placeholder="$t('accountsTs.dialogVisible.inputName')"
        ></el-input>
      </el-form-item>
      <el-form-item
          prop="password"
          :label="$t('accountsTs.dialogVisible.password')"
          :rules="{
          required: true,
          message: $t('accountsTs.dialogVisible.valueNameMessage'),
          trigger: 'blur',
        }"
      >
        <el-input
            v-model="accounts.password"
            :placeholder="$t('accountsTs.dialogVisible.inputPassword')"
        ></el-input>
      </el-form-item>
      <el-form-item
          label="状态"
          prop="status">
        <el-radio-group v-model="accounts.status">
          <el-radio :label="0">删除</el-radio>
          <el-radio :label="1">正常</el-radio>
          <el-radio :label="2">使用中</el-radio>
          <el-radio :label="3">无法使用</el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>
    <div style="text-align: center">
      <el-button size="small" type="primary" @click="summit">{{
          $t('form.confirm')
        }}
      </el-button>
    </div>
  </el-dialog>
  <el-button size="mini" round type="primary" @click="open">{{
      $t('accountsTs.addAccounts')
    }}
  </el-button>
  <el-table :data="pageData" style="width: 100%; margin-top: 20px" border>
    <el-table-column
        label="id"
        width="90"
        align="center"
        prop="id"
    ></el-table-column>
    <el-table-column
        :label="$t('accountsTs.paramsList.appName')"
        align="center"
        prop="appName"
        width="240px"
    ></el-table-column>
    <el-table-column
        :label="$t('accountsTs.paramsList.name')"
        align="center"
        prop="name"
        width="240px"
    ></el-table-column>
    <el-table-column
        :label="$t('accountsTs.paramsList.password')"
        header-align="center"
        prop="password"
    ></el-table-column>
    <el-table-column
        label="UDID"
        header-align="center"
        prop="udId"
    ></el-table-column>
    <el-table-column
        :label="$t('accountsTs.paramsList.status')"
        align="center"
        prop="status"
        width="240px"
    >
      <template #default="scope">
        <el-tag v-if="scope.row.status === 0" size="small" type="error"
        >{{ $t('accountsTs.status.delete') }}
        </el-tag>
        <el-tag v-if="scope.row.status === 1" size="small" type="success"
        >{{ $t('accountsTs.status.nornal') }}
        </el-tag>
        <el-tag v-if="scope.row.status === 2" size="small" type="info"
        >{{ $t('accountsTs.status.inUse') }}
        </el-tag>
        <el-tag v-if="scope.row.status === 3" size="small" type="warning"
        >{{ $t('accountsTs.status.unaleUse') }}
        </el-tag>
      </template>
    </el-table-column>
    <el-table-column :label="$t('common.operate')" width="170" align="center">
      <template #default="scope">
        <el-button
            type="primary"
            size="mini"
            @click="editaccounts(scope.row.id)"
        >{{ $t('common.edit') }}
        </el-button>
        <el-popconfirm
            style="margin-left: 10px"
            :confirm-button-text="$t('form.confirm')"
            :cancel-button-text="$t('form.cancel')"
            icon="el-icon-warning"
            icon-color="red"
            :title="$t('accountsTs.delMessage')"
            @confirm="deleteGlobal(scope.row.id)"
        >
          <template #reference>
            <el-button type="danger" size="mini"
            >{{ $t('common.delete') }}
            </el-button>
          </template>
        </el-popconfirm>
      </template>
    </el-table-column>
  </el-table>
</template>
