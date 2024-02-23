<script setup>
  import Index from './components/index.vue'
  import Barcode from './components/barcode.vue'
  import Analyze from './components/analyze.vue'
  import BarcodeInfo from './components/barcode_info.vue'
  import { reactive, ref, watch, h } from 'vue';


  const selectedKeys = ref(['13']);
  const openKeys = ref(['sub1']);
  function getItem(label, key, icon, children, type) {
    return {
      key,
      icon,
      children,
      label,
      type,
    };
  }
  const data = reactive({
    type:'index',
  })

  const items = reactive([
    
    getItem('码鸭工具', 'grp', null,
      [
        getItem('二维码批量生成', '13'),
        getItem('二维码批量解析', '14'),
        getItem('条形码批量生成', '15'),
        getItem('条形码类型说明', '16'),
      ],
      'group'),
  ]);
  const handleClick = e => {
    // console.log('click', e);
    if(e.key == '13'){
      data.type = 'index';
    }
    if(e.key == '14'){
      data.type = 'analyze';
    }
    if(e.key == '15'){
      data.type = 'barcode';
    }
    if(e.key == '16'){
      data.type = 'BarcodeInfo';
    }
  };
  watch(openKeys, val => {
    console.log('openKeys', val);
  });
</script>

<template>
  <div style="display: flex;">
    <div>
      <a-menu id="dddddd" v-model:openKeys="openKeys" v-model:selectedKeys="selectedKeys" style="width: 180px"
        mode="inline" :items="items" @click="handleClick"></a-menu>
    </div>
    <div style="width: calc( 100vw - 180px);">
      <div v-if="data.type == 'index'">
        <Index />
      </div>
      <div  v-if="data.type == 'barcode'">
        <Barcode />
      </div>
      <div  v-if="data.type == 'analyze'">
        <Analyze />
      </div>
      <div  v-if="data.type == 'BarcodeInfo'">
        <BarcodeInfo />
      </div>
      
      
      
    </div>
  </div>


</template>

<style scoped>

</style>