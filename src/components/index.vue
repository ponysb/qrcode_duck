<script setup>
    import { reactive, ref } from 'vue'
    import * as XLSX from "xlsx"
    import html2canvas from 'html2canvas';
    import JSZip from 'jszip'
    import vueQr from 'vue-qr/src/packages/vue-qr.vue'
    const qrcodeCanvasRef = ref(null);

    const data = reactive({
        xlsx_data: [],
        qr: [],
        qr_type: {
            type: 'canvas',
            icon: '',
            size: 260,
            bordered: false,
            margin:20,
            errorLevel: '1',
            binarize:false,    //二值化
            bgSrc:'',
            gifBgSrc:''

        },

        dan_url:''
    })

    const columns = [
        {
            title: '二维码文件标题',
            dataIndex: 'one',
            key: 'one',
            width: 120,
        },
        {
            title: '二维码链接',
            dataIndex: 'two',
            key: 'two',
            width: 320,
        },
    ];




    //二维码生成
    function qrcode_click() {
        for (let index1 = 0; index1 < data.xlsx_data.length; index1++) {
            data.qr.push({ src: data.xlsx_data[index1].two, title: data.xlsx_data[index1].one })
        }
    }

    //下载png二维码
    async function download_qrcode() {
        var zip = new JSZip();
        // 获取所有图像元素
        const qrCodes = document.querySelectorAll('.qr-code-');
        // var imageElements = await qrcodeCanvasRef.value
        await Promise.all(Array.from(qrCodes).map(async (qrCode, index) => {
            const canvas = await html2canvas(qrCode,{ scale: 1 });
            const imgData = canvas.toDataURL('image/png').split(',')[1];
            zip.file(`${data.qr[index].title}.png`, imgData, { base64: true });
        }));

        const content = await zip.generateAsync({ type: 'blob' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(content);
        link.download = 'qrcodes.zip';
        link.click();
    }


    //excel解析
    const handleFileChange = (event) => {
        const file = event.target.files[0];

        if (file) {
            const reader = new FileReader();

            reader.onload = (e) => {
                const data2 = e.target.result;
                const workbook = XLSX.read(data2, { type: 'binary' });

                // Assuming only one sheet in the Excel file
                const sheetName = workbook.SheetNames[0];
                const sheet = workbook.Sheets[sheetName];

                // Convert sheet data to an array of arrays
                const data1 = XLSX.utils.sheet_to_json(sheet, { header: 1 });

                for (let index = 0; index < data1.length; index++) {
                    // Check if the row is empty
                    if (data1[index].every(cell => !cell)) {
                        continue; // Skip this iteration if the row is empty
                    }

                    data.xlsx_data.push({
                        one: data1[index][0],
                        two: data1[index][1]
                    });
                }
            };

            reader.readAsBinaryString(file);
        }
    };


    //选择logo
    function handleLogoChange(e) {
        data.qr_type.icon = URL.createObjectURL(e.target.files[0])
    }

    //选择背景图
    function handleBgChange(e) {
        data.qr_type.bgSrc = URL.createObjectURL(e.target.files[0])
    }

    // //选择背景gif图
    // function handleGifChange(e) {
    //     data.qr_type.gifBgSrc = URL.createObjectURL(e.target.files[0])
    // }

    function jiaocheng() {
        window.open('https://wwu.lanzouo.com/ie7wX1p6u3ij');
    }

    //插入链接
    function add_url(){
        data.xlsx_data.push({
            one: '链接 '+ (data.xlsx_data.length + 1),
            two: data.dan_url
        })
    }
</script>

<template>
    <div style="overflow: hidden;">
        <div class="toubu">
            <div style="display: flex; justify-content:space-between;">
                <div class="shangchuan">
                    <a-input-group style="width: 380px;" compact>
                        <a-input placeholder="请输入要生成二维码的链接..." v-model:value="data.dan_url" style="width: 280px" />
                        <a-button @click="add_url" type="primary">插入链接</a-button>
                    </a-input-group>

                    <input type="file" id="file" accept=".xlsx" style="display: none;" @change="handleFileChange" />
                    <label for="file" class="custom-file-label">导入excel</label>

                    <a-button style="margin-right: 12px;" @click="qrcode_click">生成二维码</a-button>
                    <a-button @click="download_qrcode">二维码打包下载</a-button>

                </div>
                <div @click="jiaocheng" style="margin-top: 20px;margin-right: 10px; cursor: pointer;">
                    <a-tag color="default">
                        导入模板下载
                    </a-tag>
                </div>
            </div>

            <div class="shezhi">
                <input type="file" id="filelogo" accept=".png,.jpg,.jpeg,.webp" style="display: none;"
                    @change="handleLogoChange" />
                <label for="filelogo" class="custom-file-label">上传二维码logo</label>

                <input type="file" id="filebg" accept=".png,.jpg,.jpeg,.webp" style="display: none;"
                    @change="handleBgChange" />
                <label for="filebg" class="custom-file-label">上传二维码背景图</label>

                <!-- <input type="file" id="filegif" accept=".gif" style="display: none;"
                    @change="handleGifChange" />
                <label for="filegif" class="custom-file-label">上传GIF背景图</label> -->

                <a-input addon-before="二维码边框(像素)" style="width: 200px; margin-right: 20px;" v-model:value="data.qr_type.margin"
                    placeholder="像素px" />

                <a-input addon-before="二维码尺寸(像素)" style="width: 200px; margin-right: 20px;" v-model:value="data.qr_type.size"
                    placeholder="像素px" />

                <span style="margin-right: 12px;">二维码密度:</span>

                <a-radio-group style="margin-right: 20px;" v-model:value="data.qr_type.errorLevel">
                    <a-radio-button value="0">L</a-radio-button>
                    <a-radio-button value="1">M</a-radio-button>
                    <a-radio-button value="2">Q</a-radio-button>
                    <a-radio-button value="3">H</a-radio-button>
                </a-radio-group>

                <a-checkbox v-model="data.qr_type.binarize">二值化处理</a-checkbox>
            </div>
        </div>


        <div class="qr-box">
            <div class="excel-list">
                <a-table :columns="columns" :data-source="data.xlsx_data">
                    <template #bodyCell="{ column, text }">
                        <template v-if="column.dataIndex === 'name'">
                            <a>{{ text }}</a>
                        </template>
                    </template>
                </a-table>
            </div>

            <div class="qr-code">
                <div style="margin-top: 16px; border: 1px solid #f1f1f1;" v-for="(item,index) in data.qr">
                    <vue-qr
                        class="qr-code-"
                        ref="qrcodeCanvasRef"
                        :margin="data.qr_type.margin"
                        :logoSrc="data.qr_type.icon"
                        :correctLevel="Number(data.qr_type.errorLevel)" 
                        :text="item.src"
                        :binarize="data.qr_type.binarize"
                        :size="data.qr_type.size"
                        :bgSrc="data.qr_type.bgSrc"
                        :gifBgSrc="data.qr_type.gifBgSrc"
                    ></vue-qr>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
    .custom-file-label {
        display: inline-block;
        padding: 8px 10px 8px 10px;
        background: #0d6df3;
        border-radius: 5px;
        color: #fff;
        font-size: 14px;
        margin-right: 10px;
        cursor: pointer;
        transition: all 0.2s;
        /* 过度 */
        -moz-transition: all 0.2s;
        /* Firefox */
        -webkit-transition: all 0.2s;
        /* Safari 和 Chrome */
    }

    .custom-file-label:hover {
        background: #3a82e8;
    }

    .shezhi {
        margin-left: 20px;
        height: 50px;
        display: flex;
        align-items: center;
    }

    .shangchuan {
        height: 50px;
        margin-left: 20px;
        margin-top: 6px;
        display: flex;
        align-items: center;
    }

    .toubu {
        width: 97%;
        height: 113px;
        margin: 0 auto;
        margin-top: 20px;
        margin-bottom: 20px;
        border-radius: 5px;
        border: 1px solid #d9d9d9;
    }

    .qr-code {
        margin-right: 16px;
        width: 50%;
        height: calc(100vh - 153px);
        display: flex;
        justify-content: space-between;
        align-content: flex-start;
        flex-wrap: wrap;
        overflow-y: scroll;
    }

    .qr-box {
        display: flex;
    }

    .excel-list {
        margin-left: 16px;
        width: 50%;
        height: calc(100vh - 153px);
        overflow-y: scroll;
    }
</style>