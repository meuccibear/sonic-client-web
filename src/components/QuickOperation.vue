<script setup>
import {onMounted, ref} from 'vue';
import {ElMessage} from 'element-plus';
import {useRoute, useRouter} from 'vue-router';
import {useStore} from 'vuex';
import {useI18n} from 'vue-i18n';
import axios from '../http/axios';
import StepList from '../components/StepListView.vue';


const {t: $t} = useI18n();
const img = import.meta.globEager('./../assets/img/*');
const props = defineProps({
  isUpdate: Boolean,
});
const store = useStore();
const route = useRoute();
const router = useRouter();
const tableLoading = ref(false);
const pageData = ref({});
const loading = ref(false);

const project = ref({
  id: route.params.projectId,
  groupId: null,
  // groupIds: [],
  caseId: null,
  steps: [],
  script: null,
  // deviceIds: []
});

const formData = ref({
  groups: [],
  cases: []
});

const getTestCaseList = (data) => {
  // console.log(JSON.stringify(project.value.groupId))
  // console.log(JSON.stringify(data))
  tableLoading.value = true;
  let platform = null
  formData.value.groups.forEach(function (data, index) {
    if (project.value.groupId === data['id']) {
      platform = data['platform']
    }
  })

  axios.get('/controller/testCases/listAll', {
    params: {
      projectId: project.value.id,
      platform: platform
    },
  }).then((resp) => {
    formData.value.cases = resp.data;
    formData.value.cases.push({id: -10, name: "执行脚本"})
    tableLoading.value = false;
  });
};
const getTestSuitesList = () => {
  tableLoading.value = true;
  axios
      .get('/controller/testSuites/listAll', {
        params: {
          projectId: project.value.id
        },
      })
      .then((resp) => {
        formData.value.groups = resp.data;
        tableLoading.value = false;
      });
};

const projectUpdateForm = ref(null);
const substr = (data, key) => {
  // console.log('data:', data)
  let index = data.indexOf(key)
  // console.log(index, index + key.length)
  if (index > -1) {
    let result = data.substring(index + key.length)
    // console.log('result:', result)
    return result
  }
  return undefined
}
const summit = () => {
  projectUpdateForm.value.validate((valid) => {
    if (valid) {
      // console.log('params:', JSON.stringify(formData.value.groupId))
      // project.value.deviceIds = []
      // project.value.groupIds = []
      // formData.value.groupId.forEach(function (data, index) {
      //   if (data.length > 1) {
      //     project.value.deviceIds.push(substr(data[data.length - 1], "device_") - 0)
      //   }else{
      //     project.value.groupIds.push(substr(data[data.length - 1], "group_") - 0)
      //   }
      // })
      // let set = new Set(project.value.deviceIds);
      // project.value.deviceIds = Array.from(set);
      // console.log('arr', project.value.deviceIds)

      loading.value = true;
      axios.post('/controller/testSuites/run', project.value).then((resp) => {
        loading.value = false;
        if (resp.code === 2000) {
          ElMessage.success({
            message: resp.message + $t('testSuitesTS.testStart'),
          });
        }
      });
    }
  });
};
const testCase = ref({});
const getImg = (name) => {
  let result;
  if (name === 'meizu') {
    name = 'Meizu';
  }
  if (name === 'LENOVO') {
    name = 'Lenovo';
  }
  try {
    result = img[`./../assets/img/${name}.jpg`].default;
  } catch {
    result = img['./../assets/img/unName.jpg'].default;
  }
  return result;
};
const getStepsList = (caseId) => {
  axios
      .get('/controller/steps/listAll', {
        params: {
          caseId: caseId,
        },
      })
      .then((resp) => {
        project.value.steps = resp.data
        console.log('data', project.value.steps)
      });
};
const getCaseInfo = () => {
  console.log('caseId::', project.value.caseId)
  if (project.value.caseId != -10) {
    axios.get('/controller/testCases', {
      params: {
        id: project.value.caseId
      },
    }).then((resp) => {
      if (resp.code === 2000) {
        testCase.value = resp.data;
        getStepsList(testCase.value.id)
      }
    });
  }
};
onMounted(() => {
  if (props.isUpdate) {
    // console.log(store.state.project)
    // project.value = JSON.parse(JSON.stringify(store.state.project));
    getTestSuitesList()
    // getTestCaseList()
  }
});
</script>

<template>
  <el-form
      ref="projectUpdateForm"
      :model="project"
      label-width="100px"
      class="project-table-expand"
  >

    <el-form-item :label="$t('form.script')">
      <el-input
          v-model="project.script"
          :autosize="{ minRows: 2, maxRows: 4 }"
          type="textarea"
      />
    </el-form-item>
    <el-form-item :label="$t('form.group')">
<!--      <el-cascader-->
<!--          style="width: 78vw"-->
<!--          :placeholder="$t('form.groupPlaceholder')"-->
<!--          v-model="formData.groupId"-->
<!--          :options="formData.groups"-->
<!--          emitPath="false"-->
<!--          clearable-->
<!--          :props="{ multiple: true }"-->
<!--          filterable-->
<!--      />-->
      <el-select
          v-model="project.groupId"
          style="width: 100%"
          :placeholder="$t('form.groupPlaceholder')"
          @change="getTestCaseList"
      >
        <el-option
            v-for="item in formData.groups"
            :key="item.id"
            :value="item.id"
            :label="item.name"
        >
        </el-option>
      </el-select>
    </el-form-item>
    <el-form-item :label="$t('form.case')">
      <el-select
          v-model="project.caseId"
          style="width: 100%"
          :placeholder="$t('form.casePlaceholder')"
          @change="getCaseInfo"
      >
        <el-option
            v-for="item in formData.cases"
            :key="item.name"
            :value="item.id"
            :label="item.name"
            :disabled="item['disabled']"
        >
        </el-option>
      </el-select>
    </el-form-item>
  </el-form>
  <el-row
      :gutter="20"
      v-show="project.caseId ? project.caseId != -10 : false"
  >
    <el-col :span="6">
      <el-card>
        <template><strong>{{ $t('stepListViewTS.caseInfo') }}</strong></template
        >
        <el-form
            v-if="testCase['id']"
            label-position="left"
            class="demo-table-expand"
            label-width="100px"
            style="margin-left: 10px; word-break: break-all"
        >
          <el-form-item :label="$t('projectIndexTS.page.caseId')">
            <span>{{ testCase['id'] }}</span>
          </el-form-item>
          <el-form-item :label="$t('projectIndexTS.page.caseName')">
            <span>{{ testCase.name }}</span>
          </el-form-item>
          <el-form-item :label="$t('stepListViewTS.platformToBe')">
            <div style="display: flex; align-items: center">
              <el-avatar
                  style="margin-right: 10px"
                  :size="27"
                  :src="getImg(testCase['platform'] === 1 ? 'ANDROID' : 'IOS')"
                  shape="square"
              ></el-avatar>
              {{
                testCase['platform'] === 1 ? $t('publicStepTS.android') : 'iOS'
              }}
            </div>
          </el-form-item>
          <el-form-item :label="$t('stepListViewTS.module')">
            <span>{{
                testCase['modulesDTO'] !== null ? testCase['modulesDTO'].name : ''
              }}</span>
          </el-form-item>
          <el-form-item :label="$t('stepListViewTS.versionName')">
            <span>{{ testCase['version'] }}</span>
          </el-form-item>
          <el-form-item :label="$t('stepListViewTS.designer')">
            <span>{{ testCase['designer'] }}</span>
          </el-form-item>
          <el-form-item :label="$t('stepListViewTS.last')">
            <span>{{ testCase['editTime'] }}</span>
          </el-form-item>
          <el-form-item :label="$t('stepListViewTS.testMessage')">
            <span>{{ testCase['des'] }}</span>
          </el-form-item>
        </el-form>
      </el-card>
    </el-col>
    <el-col :span="18">
      <el-card shadow="hover">
        <step-list
            ref="stepListView"
            v-if="testCase['id']"
            :is-show-run="false"
            :platform="testCase['platform']"
            :is-driver-finish="false"
            :case-id="testCase['id']"
            :steps="project.steps"
            :project-id="project.id"
            @getStepsList="getStepsList"
        />
      </el-card>
    </el-col>
  </el-row>
  <div style="text-align: center">
    <el-button size="medium" type="primary" icon="el-icon-edit" @click="summit">{{ $t('form.send') }}</el-button>
  </div>
</template>
