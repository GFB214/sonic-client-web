<script setup>
import { onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';
import { useI18n } from 'vue-i18n';
import PublicStepUpdate from '../components/PublicStepUpdate.vue';
import Pageable from '../components/Pageable.vue';
import axios from '../http/axios';
import StepShow from '../components/StepShow.vue';

const { t: $t } = useI18n();

const route = useRoute();
const dialogVisible = ref(false);
const pageData = ref({});
const pageSize = ref(15);
const publicStepId = ref(0);
watch(dialogVisible, (newValue, oldValue) => {
  if (!newValue) {
    publicStepId.value = 0;
  }
});
const editPublicStep = async (id) => {
  publicStepId.value = id;
  await open();
};
const open = () => {
  dialogVisible.value = true;
};
const flush = (e) => {
  if (e) {
    dialogVisible.value = false;
  }
  getPublicStepList();
};
const getPublicStepList = (pageNum, pSize) => {
  axios
    .get('/controller/publicSteps/list', {
      params: {
        projectId: route.params.projectId,
        page: pageNum || 1,
        pageSize: pSize || pageSize.value,
      },
    })
    .then((resp) => {
      pageData.value = resp.data;
    });
};
// 复制该公共步骤
// const copyPublicStepId = (id) => {
//   axios.get("/controller/publicSteps/copy", {
//     params: {
//       id
//     }
//   }).then(resp => {
//     if (resp['code'] === 2000) {
//       ElMessage.success({
//         message: resp['message']
//       });
//       getPublicStepList()
//     }
//   })
// }
const deletePublicStep = (id) => {
  axios
    .delete('/controller/publicSteps', {
      params: {
        id,
      },
    })
    .then((resp) => {
      if (resp.code === 2000) {
        ElMessage.success({
          message: resp.message,
        });
        getPublicStepList();
      }
    });
};
onMounted(() => {
  getPublicStepList();
});
</script>

<template>
  <el-dialog
    v-model="dialogVisible"
    :title="$t('publicStepTS.info')"
    width="750px"
  >
    <public-step-update
      v-if="dialogVisible"
      :public-step-id="publicStepId"
      :project-id="route.params.projectId"
      @flush="flush"
    />
  </el-dialog>
  <el-button size="mini" round type="primary" @click="open">{{
    $t('publicStepTS.add')
  }}</el-button>
  <el-table :data="pageData['content']" border style="margin-top: 10px">
    <el-table-column
      width="100"
      :label="$t('publicStepTS.id')"
      prop="id"
      align="center"
    />
    <el-table-column
      :label="$t('publicStepTS.name')"
      prop="name"
      header-align="center"
    />
    <el-table-column
      :label="$t('publicStepTS.platform')"
      width="110"
      align="center"
    >
      <template #default="scope">
        {{ scope.row.platform === 1 ? $t('publicStepTS.android') : 'iOS' }}
      </template>
    </el-table-column>
    <el-table-column
      :label="$t('publicStepTS.list')"
      width="110"
      align="center"
    >
      <template #default="scope">
        <el-popover placement="left" :width="500" trigger="click">
          <el-table :data="scope.row.steps" border max-height="350">
            <el-table-column
              width="80"
              :label="$t('publicStepTS.stepId')"
              prop="id"
              align="center"
              show-overflow-tooltip
            />
            <el-table-column
              width="90"
              :label="$t('publicStepTS.useCase')"
              align="center"
            >
              <template #default="scope">
                <el-tag v-if="scope.row.caseId === 0" size="mini">{{
                  $t('common.null')
                }}</el-tag>
                <span v-else>{{ scope.row.caseId }}</span>
              </template>
            </el-table-column>
            <el-table-column
              :label="$t('publicStepTS.stepInfo')"
              header-align="center"
            >
              <template #default="scope">
                <step-show :step="scope.row" />
              </template>
            </el-table-column>
          </el-table>
          <template #reference>
            <el-button size="mini">{{
              $t('publicStepTS.viewSteps')
            }}</el-button>
          </template>
        </el-popover>
      </template>
    </el-table-column>
    <el-table-column :label="$t('common.operate')" width="250" align="center">
      <template #default="scope">
        <!--        <el-button type="primary"-->
        <!--                   size="mini"-->
        <!--                   v-on:click="copyPublicStepId(scope.row.id)"-->
        <!--        >-->
        <!--          复制-->
        <!--        </el-button>-->

        <el-button
          type="primary"
          size="mini"
          @click="editPublicStep(scope.row.id)"
        >
          {{ $t('common.edit') }}
        </el-button>

        <el-popconfirm
          style="margin-left: 10px"
          :confirm-button-text="$t('form.confirm')"
          :cancel-button-text="$t('form.cancel')"
          icon="el-icon-warning"
          icon-color="red"
          :title="$t('publicStepTS.sureDel')"
          @confirm="deletePublicStep(scope.row.id)"
        >
          <template #reference>
            <el-button type="danger" size="mini">
              {{ $t('common.delete') }}
            </el-button>
          </template>
        </el-popconfirm>
      </template>
    </el-table-column>
  </el-table>
  <pageable
    :is-page-set="true"
    :total="pageData['totalElements']"
    :current-page="pageData['number'] + 1"
    :page-size="pageData['size']"
    @change="getPublicStepList"
  ></pageable>
</template>
