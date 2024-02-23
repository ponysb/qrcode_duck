<script setup>
    import { onMounted,reactive } from 'vue'
    import JsBarcode from 'jsbarcode'
    import JSZip from 'jszip'
    import * as XLSX from "xlsx"

    const data = reactive({
        dan_code:'',
        barcode: [],
        barcode_:{
            format: "CODE39",//选择要使用的条形码类型
            width: 2,//设置条之间的宽度
            height: 100,//高度
            displayValue: true,//是否在条形码下方显示文字
            textMargin: 5,//设置条形码和文本之间的间距
            fontSize: 22,//设置文本的大小
            background: "#fff",//设置条形码的背景
            lineColor: "#000",//设置条和文本的颜色。
            margin: 15,//设置条形码周围的空白边距
            fontOptions:"bold",
            font:"Microsoft YaHei"
        }
    })
    const columns = [
        {
            title: '条形码文件标题',
            dataIndex: 'one',
            key: 'one',
            width: 120,
        },
        {
            title: '条形码',
            dataIndex: 'two',
            key: 'two',
            width: 320,
        },
    ];

    onMounted(() => {
        generate()
    })

    function generate(){
        // 生成条形码
        data.barcode.forEach((item, index) => {
            JsBarcode('#barcode_'+item.two, item.two,data.barcode_)
            // 修改文本的样式
            const textElement = document.querySelector('#barcode_'+item.two);
            textElement.style.fontSize = data.barcode_.fontSize + 'px';
            textElement.style.fontWeight = data.barcode_.fontOptions;
        })
    }

    async function daochu(){
        const zip = new JSZip();
        data.barcode.forEach((item, index) => {
            const svgElement = document.querySelector('#barcode_' + item.two);
            const serializer = new XMLSerializer();
            const svgString = serializer.serializeToString(svgElement);
            zip.file(`${item.one}.svg`, svgString);
        });

        const content = await zip.generateAsync({ type: 'blob' });
        const url = URL.createObjectURL(content);
        const link = document.createElement('a');
        link.href = url;
        link.download = 'barcodes.zip';
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

                    data.barcode.push({
                        one: data1[index][0],
                        two: data1[index][1]
                    });
                }
            };

            reader.readAsBinaryString(file);
        }
    };


    function add_code(){
        data.barcode.push({
            one: '条形码'+(data.barcode.length+1),
            two: data.dan_code
        })
    }

</script>

<template>
    <div>
        <div class="top">
            <div style="margin-top: 13px; margin-left: 20px; display: flex;">
                <a-select
                    ref="select"
                    v-model:value="data.barcode_.format"
                    style="width: 150px; margin-right: 20px;"
                >
                    <a-select-option value="CODE39">CODE39</a-select-option>
                    <a-select-option value="CODE128">CODE128</a-select-option>
                    <a-select-option value="EAN13">EAN13</a-select-option>
                    <a-select-option value="EAN8">EAN8</a-select-option>
                    <a-select-option value="UPC">UPC</a-select-option>
                    <a-select-option value="ITF14">ITF14</a-select-option>
                    <a-select-option value="MSI">MSI</a-select-option>
                    <a-select-option value="MSI10">MSI10</a-select-option>
                    <a-select-option value="MSI11">MSI11</a-select-option>
                    <a-select-option value="MSI1010">MSI1010</a-select-option>
                    <a-select-option value="MSI1110">MSI1110</a-select-option>
                    <a-select-option value="pharmacode">pharmacode</a-select-option>
                    <a-select-option value="codabar">codabar</a-select-option>
                </a-select>
                
                <a-input-group style="width: 280px;" compact>
                    <a-input placeholder="请输入编码" v-model:value="data.dan_code" style="width: 180px" />
                    <a-button @click="add_code" type="primary">插入编码</a-button>
                </a-input-group>

                <input type="file" id="file" accept=".xlsx" style="display: none;" @change="handleFileChange" />
                    <label for="file" class="custom-file-label">导入excel</label>

                <a-button style="margin-left: 20px; margin-right: 20px;" @click="generate">生成</a-button>
                <a-button @click="daochu">下载</a-button>
            </div>

            <div style="margin-top: 20px; margin-left: 20px;">
                <a-input addon-before="条间距(px)" style="width: 150px; margin-right: 20px;" 
                v-model:value="data.barcode_.width" placeholder="像素px" />

                <a-input addon-before="条高度(px)" style="width: 150px; margin-right: 20px;" 
                v-model:value="data.barcode_.height" placeholder="像素px" />

                <span>背景色：</span>
                <input style="margin-right: 20px;" v-model="data.barcode_.background" type="color">

                <span>条码和文字颜色：</span>
                <input style="margin-right: 20px;" v-model="data.barcode_.lineColor" type="color">

                <span style="line-height: 28px;">编码显示：</span>
                <a-switch style="margin-right: 20px;" v-model:checked="data.barcode_.displayValue" checked-children="开"
                   un-checked-children="关" />
               
               <a-input addon-before="白边(px)" style="width: 130px; margin-right: 20px;" 
                   v-model:value="data.barcode_.margin" placeholder="像素px" />
   
               <a-input addon-before="与文字距离(px)" style="width: 160px; margin-right: 20px;" 
                v-model:value="data.barcode_.textMargin" placeholder="像素px" />
   
               <a-input addon-before="文字大小(px)" style="width: 150px; margin-right: 20px;" 
                v-model:value="data.barcode_.fontSize" placeholder="像素px" />
            </div>

        </div>

        <div style="display: flex;">
            <div class="excel-list">
                <a-table :columns="columns" :data-source="data.barcode">
                    <template #bodyCell="{ column, text }">
                        <template v-if="column.dataIndex === 'name'">
                            <a>{{ text }}</a>
                        </template>
                    </template>
                </a-table>
            </div>
            <div style="width: 50%; height: calc(100vh - 153px);overflow-y: scroll;">
                <svg v-for="(item,index) in data.barcode" :id="'barcode_'+item.two"></svg>
            </div>
        </div>
        
    </div>
</template>

<style scoped>
.custom-file-label {
    display: inline-block;
    width: 80px;
    height: 31px;
    text-align: center;
    line-height: 31px;
    margin-left: 8px;
    background: #0d6df3;
    border-radius: 5px;
    color: #fff;
    font-size: 14px;
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

.top{
    width: 97%;
    height: 113px;
    margin: 0 auto;
    margin-top: 20px;
    margin-bottom: 20px;
    border-radius: 5px;
    border: 1px solid #d9d9d9;
}
.excel-list {
    margin-left: 16px;
    width: 50%;
    height: calc(100vh - 153px);
    overflow-y: scroll;
}
</style>

