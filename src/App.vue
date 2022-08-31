<template>
  <ConfigProvider :locale="getAntdLocale">
    <AppProvider>
      <RouterView />
      <Modal v-model:visible="visible" :wrapStyle="{ overflow: 'hidden' }" :footer="null">
        <Progress :percent="percent" />
      </Modal>
    </AppProvider>
  </ConfigProvider>
</template>

<script lang="ts" setup>
  import { ref } from 'vue';
  import { ConfigProvider, Modal, Progress } from 'ant-design-vue';
  import { AppProvider } from '/@/components/Application';
  import { useTitle } from '/@/hooks/web/useTitle';
  import { useLocale } from '/@/locales/useLocale';

  // Auto subscribe main process message
  import { ipcRenderer } from 'electron';

  import 'dayjs/locale/zh-cn';
  // support Multi-language
  const { getAntdLocale } = useLocale();
  const visible = ref(true);
  const message = ref('');
  const percent = ref(0);
  ipcRenderer.on('message', (_: any, arg) => {
    // for (var i = 0; i < arg.length; i++) {
    if ('update-available' == arg.cmd) {
      //显示升级对话框
      visible.value = true;
    } else if ('download-progress' == arg.cmd) {
      //更新升级进度
      /**
       *
       * message{bytesPerSecond: 47673
        delta: 48960
        percent: 0.11438799862426002
        total: 42801693
        transferred: 48960
        }
       */
      console.log(arg.message.percent);
      percent.value = Math.round(parseFloat(arg.message.percent));
    } else if ('error' == arg.cmd) {
      visible.value = false;
      //更新失败
      message.value = '更新失败';
    }
  });
  // Listening to page changes and dynamically changing site titles
  useTitle();
</script>
