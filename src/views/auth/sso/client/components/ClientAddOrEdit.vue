<!--
 * @author: gaoweixuan
 * @since: 2023-11-12
-->

<!-- 客户端添加修改弹出框 -->
<script lang="ts" setup>
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import { addClient, checkClientCode, editClient, getClient } from '@/api/auth/client'
import { ClientForm } from '@/api/auth/client/type.ts'
import { useI18n } from 'vue-i18n'
import JSONBigInt from 'json-bigint'
import { useDict } from '@/hooks/dict'
import useWidth from '@/hooks/dialogWidth'

defineOptions({
  name: 'ClientAddOrEdit',
  inheritAttrs: false,
})

const { t } = useI18n()
const $emit = defineEmits(['reloadDataList'])
const visible = ref<boolean>(false)
const loading = ref<boolean>(false)
const clientDataFormRef = ref()
const clientDataForm = ref<ClientForm>({
  clientId: '',
  clientIdIssuedAt: '',
  clientName: '',
  clientSecret: '',
  clientSecretExpiresAt: '',
  confirmClientSecret: '',
  redirectUris: [],
})

const { REDIRECT_URIS } = useDict('REDIRECT_URIS')

const rules = ref({
  clientCode: [
    {
      required: true,
      message: t('common.placeholder.enter') + t('client.fields.clientCode'),
      trigger: 'blur',
    },
    {
      validator: (rule: any, value: any, callback: any) => {
        checkClientCode(value, !clientDataForm.value.id ? undefined : JSONBigInt.parse(clientDataForm.value.id)).then(
          (response: any) => {
            if (response.data) {
              callback()
              return
            }
            callback(new Error(t('client.rules.clientCodeExists')))
          },
        )
      },
      trigger: 'blur',
    },
  ],
  clientName: [
    {
      required: true,
      message: t('common.placeholder.enter') + t('client.fields.clientName'),
      trigger: 'blur',
    },
  ],
})

/**
 * 初始化
 *
 * @param id
 */
const init = async (id: number) => {
  clientDataForm.value.id = undefined
  visible.value = true
  // 重置表单数据
  if (clientDataFormRef.value) {
    clientDataFormRef.value.resetFields()
  }
  if (id) {
    await getInfo(id)
  }
}

/**
 * 获取信息
 *
 * @param id
 */
const getInfo = async (id: number) => {
  const response: any = await getClient(JSONBigInt.parse(id))
  if (response.code === '0000') {
    Object.assign(clientDataForm.value, response.data)
  }
}

/**
 * 表单提交
 */
const handleClientDataFormSubmit = () => {
  clientDataFormRef.value.validate(async (valid: boolean) => {
    if (!valid) {
      return false
    }
    loading.value = true
    const id = clientDataForm.value.id
    if (id) {
      await editClient(id, clientDataForm.value)
      ElMessage.success({
        message: `${t('common.modify') + t('common.success')}`,
        duration: 1000,
        onClose: () => {
          visible.value = false
          loading.value = false
          $emit('reloadDataList')
        },
      })
    } else {
      await addClient(clientDataForm.value)
      ElMessage.success({
        message: `${t('common.save') + t('common.success')}`,
        duration: 1000,
        onClose: () => {
          visible.value = false
          loading.value = false
          $emit('reloadDataList')
        },
      })
    }
  })
}

defineExpose({
  init,
})
</script>

<template>
  <el-dialog
    v-model="visible"
    :width="useWidth()"
    :title="!clientDataForm.id ? t('common.add') : t('common.edit')"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
  >
    <el-form
      :model="clientDataForm"
      :rules="rules"
      ref="clientDataFormRef"
      @keyup.enter="handleClientDataFormSubmit()"
      label-width="90px"
    >
      <el-divider>{{ t('client.fields.basicSettings') }}</el-divider>
      <el-form-item :label="t('client.fields.clientId')" prop="clientId">
        <el-input v-model="clientDataForm.clientId" autocomplete="off" clearable />
      </el-form-item>
      <el-form-item :label="t('client.fields.clientName')" prop="clientName">
        <el-input
          v-model="clientDataForm.clientName"
          autocomplete="off"
          clearable
          :placeholder="t('common.placeholder.enter') + t('client.fields.clientName')"
        />
      </el-form-item>
      <el-form-item :label="t('client.fields.clientSecret')" prop="clientSecret" v-if="!clientDataForm.id">
        <el-input
          v-model="clientDataForm.clientSecret"
          autocomplete="off"
          clearable
          :placeholder="t('common.placeholder.enter') + t('client.fields.clientSecret')"
        />
      </el-form-item>
      <el-form-item
        :label="t('client.fields.confirmClientSecret')"
        prop="confirmClientSecret"
        v-if="!clientDataForm.id"
      >
        <el-input
          v-model="clientDataForm.confirmClientSecret"
          autocomplete="off"
          clearable
          :placeholder="t('common.placeholder.enter') + t('client.fields.confirmClientSecret')"
        />
      </el-form-item>
      <el-form-item :label="t('client.fields.clientIdIssuedAt')" prop="clientIdIssuedAt">
        <el-date-picker
          v-model="clientDataForm.clientIdIssuedAt"
          type="datetime"
          value-format="yyyy-MM-dd HH:mm:ss"
          :placeholder="t('common.placeholder.choose') + t('client.fields.clientIdIssuedAt')"
          default-time="12:00:00"
        ></el-date-picker>
      </el-form-item>
      <el-form-item :label="t('client.fields.redirectUris')" prop="redirectUris">
        <el-select
          filterable
          allow-create
          default-first-option
          multiple
          v-model="clientDataForm.redirectUris"
          :placeholder="t('common.placeholder.enter') + t('client.fields.redirectUris')"
        >
          <el-option
            v-for="(item, index) in REDIRECT_URIS"
            :key="index"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>

      <el-form-item :label="t('client.fields.clientSecretExpiresAt')" prop="clientSecretExpiresAt">
        <el-date-picker
          v-model="clientDataForm.clientSecretExpiresAt"
          type="datetime"
          :placeholder="t('common.placeholder.choose') + t('client.fields.clientSecretExpiresAt')"
          value-format="yyyy-MM-dd HH:mm:ss"
          default-time="12:00:00"
        ></el-date-picker>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button
        @click="
          () => {
            visible = false
            loading = false
          }
        "
      >
        {{ t('common.cancel') }}
      </el-button>
      <el-button type="primary" :loading="loading" @click="handleClientDataFormSubmit()">
        {{ t('common.confirm') }}
      </el-button>
    </template>
  </el-dialog>
</template>
