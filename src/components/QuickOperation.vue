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
  caseId: null
});

const formData = ref({
  groups: [],
  cases: []
});

const getTestCaseList = () => {
  tableLoading.value = true;
  let platform = null
  formData.value.groups.forEach(function (data, index) {
    if (project.value.groupId === data['id']) {
      platform = data['platform']
    }
  })

  axios
      .get('/controller/testCases/listAll', {
        params: {
          projectId: project.value.id,
          platform: platform
        },
      })
      .then((resp) => {
        formData.value.cases = resp.data;
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
const emit = defineEmits(['flush']);
const updateSteps = (steps) => {
  project.value.steps = steps
}
const summit = () => {
  projectUpdateForm.value.validate((valid) => {
    if (valid) {
      // console.log('params:', JSON.stringify(project.value))
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
const getCaseInfo = () => {
  axios
      .get('/controller/testCases', {
        params: {
          id: project.value.caseId
        },
      })
      .then((resp) => {
        if (resp.code === 2000) {
          testCase.value = resp.data;
        }
      });
};
onMounted(() => {
  if (props.isUpdate) {
    console.log(store.state.project)
    // project.value = JSON.parse(JSON.stringify(store.state.project));
    getTestSuitesList()
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
    <el-form-item :label="$t('form.group')">
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
      v-show="project.caseId"
  >
    <el-col :span="6">
      <el-card>
        <template ><strong>{{ $t('stepListViewTS.caseInfo') }}</strong></template
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
            :project-id="project.id"
            @updateSteps="updateSteps"
        />
      </el-card>
    </el-col>
  </el-row>
  <div style="text-align: center">
    <el-button size="medium" type="primary" icon="el-icon-edit" @click="summit">{{ $t('form.send') }}</el-button>
  </div>
</template>
