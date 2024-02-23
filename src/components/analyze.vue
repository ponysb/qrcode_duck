<script setup>
    import { reactive, ref } from 'vue'
    import qrcodeParser from "qrcode-parser";
    import * as XLSX from "xlsx"

    const data = reactive({
        code_img: [],
    })
    const columns = [
        {
            title: '二维码文件名称',
            dataIndex: 'one',
            key: 'one',
            width: 120,
        },
        {
            title: '二维码内容',
            dataIndex: 'two',
            key: 'two',
            width: 320,
        },
    ];

    async function handleFileSelect(evt) {
        evt.stopPropagation();
        evt.preventDefault();

        var files = evt.dataTransfer.files;
        for (var i = 0; i < files.length; i++) {
            var file = files[i];

            // 检查文件类型
            if (!file.type.startsWith('image/')) {
                alert('只能拖入图片文件');
                return;
            }else{
                data.code_img.push({one: file.name, two: await qrcodeParser(URL.createObjectURL(file))});
            }

        }
    }


    //导出excel
    function daochu_(){
        let newData = data.code_img.map(item => ({
            二维码文件名称: item.one,
            二维码内容: item.two,
        }));

        // 创建一个新的工作簿
        let wb = XLSX.utils.book_new();

        // 将你的数据转换为工作表
        let ws = XLSX.utils.json_to_sheet(newData);

        // 将工作表添加到工作簿
        XLSX.utils.book_append_sheet(wb, ws, "Sheet1");

        // 将工作簿写入文件
        XLSX.writeFile(wb, "二维码解析.xlsx");
    }
</script>

<template>
    <div style="overflow: hidden;">
        <div style="margin-top: 10px; margin-left: 10px; margin-bottom: 10px;">
            <span>请拖入二维码图片到浏览器内,解析成功后可以 </span>
            <a-button @click="daochu_">导出excel</a-button>
        </div>
        
        <div @drop="handleFileSelect"
        @dragover.prevent>
            <div class="excel-list">
                <a-table :columns="columns" :data-source="data.code_img">
                    <template #bodyCell="{ column, text }">
                        <template v-if="column.dataIndex === 'name'">
                            <a>{{ text }}</a>
                        </template>
                    </template>
                </a-table>
            </div>
        </div>
    </div>
    
</template>

<style scoped>
.excel-list {
    width: 100%;
    height: calc(100vh - 153px);
    overflow-y: scroll;
}
</style>