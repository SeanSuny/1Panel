<template>
    <div>
        <el-drawer v-model="drawerVisible" :destroy-on-close="true" :close-on-click-modal="false" size="50%">
            <template #header>
                <DrawerHeader :header="title + $t('setting.backupAccount')" :back="handleClose" />
            </template>
            <el-form @submit.prevent ref="formRef" v-loading="loading" label-position="top" :model="webdavData.rowData">
                <el-row type="flex" justify="center">
                    <el-col :span="22">
                        <el-form-item :label="$t('commons.table.type')" prop="type" :rules="Rules.requiredSelect">
                            <el-tag>{{ $t('setting.' + webdavData.rowData!.type) }}</el-tag>
                        </el-form-item>
                        <el-form-item
                            :label="$t('setting.address')"
                            prop="varsJson.addressItem"
                            :rules="Rules.requiredInput"
                        >
                            <el-input v-model="webdavData.rowData!.varsJson['addressItem']">
                                <template #prepend>
                                    <el-select v-model.trim="addressProto" style="width: 100px">
                                        <el-option label="http" value="http" />
                                        <el-option label="https" value="https" />
                                    </el-select>
                                </template>
                            </el-input>
                        </el-form-item>
                        <el-form-item :label="$t('commons.table.port')" prop="varsJson.port" :rules="[Rules.port]">
                            <el-input-number
                                :min="0"
                                :max="65535"
                                v-model.number="webdavData.rowData!.varsJson['port']"
                            />
                        </el-form-item>
                        <el-form-item
                            :label="$t('commons.login.username')"
                            prop="accessKey"
                            :rules="[Rules.requiredInput]"
                        >
                            <el-input v-model.trim="webdavData.rowData!.accessKey" />
                        </el-form-item>
                        <el-form-item
                            :label="$t('commons.login.password')"
                            prop="credential"
                            :rules="[Rules.requiredInput]"
                        >
                            <el-input
                                type="password"
                                clearable
                                show-password
                                v-model.trim="webdavData.rowData!.credential"
                            />
                        </el-form-item>
                        <el-form-item :label="$t('setting.path')" prop="bucket" :rules="[Rules.requiredInput]">
                            <el-input v-model.trim="webdavData.rowData!.bucket" />
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button :disabled="loading" @click="handleClose">
                        {{ $t('commons.button.cancel') }}
                    </el-button>
                    <el-button :disabled="loading" type="primary" @click="onSubmit(formRef)">
                        {{ $t('commons.button.confirm') }}
                    </el-button>
                </span>
            </template>
        </el-drawer>
    </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import { Rules } from '@/global/form-rules';
import i18n from '@/lang';
import { ElForm } from 'element-plus';
import { Backup } from '@/api/interface/backup';
import DrawerHeader from '@/components/drawer-header/index.vue';
import { addBackup, editBackup } from '@/api/modules/setting';
import { MsgSuccess } from '@/utils/message';
import { spliceHttp, splitHttp } from '@/utils/util';

const loading = ref(false);
type FormInstance = InstanceType<typeof ElForm>;
const formRef = ref<FormInstance>();

const addressProto = ref('http');
const emit = defineEmits(['search']);

interface DialogProps {
    title: string;
    rowData?: Backup.BackupInfo;
}
const title = ref<string>('');
const drawerVisible = ref(false);
const webdavData = ref<DialogProps>({
    title: '',
});
const acceptParams = (params: DialogProps): void => {
    webdavData.value = params;
    if (webdavData.value.title === 'edit') {
        let httpItem = splitHttp(webdavData.value.rowData!.varsJson['address']);
        webdavData.value.rowData!.varsJson['addressItem'] = httpItem.url;
        addressProto.value = httpItem.proto;
    }
    title.value = i18n.global.t('commons.button.' + webdavData.value.title);
    drawerVisible.value = true;
};

const handleClose = () => {
    emit('search');
    drawerVisible.value = false;
};

const onSubmit = async (formEl: FormInstance | undefined) => {
    if (!formEl) return;
    formEl.validate(async (valid) => {
        if (!valid) return;
        if (!webdavData.value.rowData) return;
        webdavData.value.rowData!.varsJson['address'] = spliceHttp(
            addressProto.value,
            webdavData.value.rowData!.varsJson['addressItem'],
        );
        webdavData.value.rowData.vars = JSON.stringify(webdavData.value.rowData!.varsJson);
        loading.value = true;
        if (webdavData.value.title === 'create') {
            await addBackup(webdavData.value.rowData)
                .then(() => {
                    loading.value = false;
                    MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
                    emit('search');
                    drawerVisible.value = false;
                })
                .catch(() => {
                    loading.value = false;
                });
            return;
        }
        await editBackup(webdavData.value.rowData)
            .then(() => {
                loading.value = false;
                MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
                emit('search');
                drawerVisible.value = false;
            })
            .catch(() => {
                loading.value = false;
            });
    });
};

defineExpose({
    acceptParams,
});
</script>
