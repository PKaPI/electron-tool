<template>
  <div>
    <el-container>
      <el-header>
        <el-alert
          title="操作说明"
          description="第一步：先选择excel或者csv文件；第二步：预览选择文件；第三步：确定上传。注：文件列表为多选上传！"
          type="info"
          show-icon
          :closable="false"
        ></el-alert>
      </el-header>
      <el-container style="padding-bottom:250px">
        <el-main>
          <el-upload
            class="upload-demo"
            drag
            :show-file-list="false"
            :on-change="righthandleChange"
            action="#"
            :auto-upload="false"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">
              <em>请点击选择文件</em>
            </div>
            <div class="el-upload__tip" slot="tip">只能上传excel,csv文件</div>
          </el-upload>
          <el-button type="primary">上传<i class="el-icon-upload el-icon--right"></i></el-button>
        </el-main>
      </el-container>
      <div class="excel-p">
         <div v-show="!fileData.length">预览文件</div>
          <div v-show="fileData.length">
            <el-alert :title="fileName" type="success" :description="fileInfo" show-icon></el-alert>
            <el-table
              :data="fileData"
              stripe
              border
              v-loading="loading"
              style="width: 100%"
              max-height="300"
            >
              <el-table-column
                v-for="(item,index) in tableColumn"
                :prop="item"
                :key="index"
                :label="item"
              ></el-table-column>
            </el-table>
             <el-button type="primary">包含文件列表<i class="el-icon-upload el-icon--right"></i></el-button>
             <el-table
              :data="fileList"
              stripe
              border
              v-loading="loading"
              style="width: 100%"
              max-height="300"
            >
              <el-table-column
                prop="path"
                key="1"
                label="path"
              ></el-table-column>
              <el-table-column
                prop="status"
                key="2"
                label="文件状态"
              ></el-table-column>
            </el-table>
          </div>
      </div>
    </el-container>
  </div>
</template>
<script>

let xlsx = require("node-xlsx");
let fs = require("fs");
let moment = require("moment");
// const { remote } = require("electron");
// let Dir = remote.app.getPath("desktop");

// const { ipcRenderer } = require("electron");
// function init() {
//   //监听main process里发出的message
//   ipcRenderer.on("downstate", (event, arg) => {
//     console.log("downstate");
//   });
// }
export default {
  name: "index",
  data() {
    return {
      fileData: [], //excel显示json
      loading:false,
      fileName:'表格头',
      fileInfo:'已上传文件列表',
      tableColumn:[],
      fileList:[]
    };
  },
  methods: {
    righthandleChange: function(file, fileList) {
      this.fileReader(file, fileList);
    },
    fileReader: function(file, fileList) {
      const loading = this.$loading({ text: "文件读取中，请稍等.." });
      let fileType = file.name
        .substring(file.name.lastIndexOf(".") + 1)
        .toUpperCase();
      if (fileType == "XLSX" || fileType == "XLS" || fileType == "CSV") {
        let obj = null;
        try {
          let obj = xlsx.parse(file.raw.path);
          const data = obj[0].data;
          const title = obj[0].name;
          const column = data.splice(0,1)[0];
          var patrn=/^[C|D|E|F]:\\.+\\.+$/;
          let fileList = []
          const tableData = data.map(element=> {
                let child = {};
                element.forEach((item,index) => {
                  if (patrn.exec(item)){
                      fileList.push({
                        path:item,
                        status:fs.existsSync(item) ? '存在':'不存在'
                      })
                  }
                    child[column[index]] = item;
                });
                return child;
         })
          this.fileData=tableData.slice(0,20);
          this.fileList=fileList
          this.tableColumn=column;
          loading.close();
          this.$nextTick(function() {
            this.$notify({
              title: "提示",
              message: "上传成功",
              type: "success"
            });
          });
        } catch (error) {
          loading.close();
          this.$notify.error({
            title: "错误",
            message: "读取：【" + file.name + "】失败，错误：" + error
          });
        }
      } else {
        loading.close();
        this.$notify({
          title: "警告",
          message: file.name + "文件格式错误，只允许上传excel、csv文件！",
          type: "warning",
          duration: 0
        });
      }
    }
  }
};
</script>
<style scoped>
.el-header {
  border-bottom: 1px dotted #f2f2f2;
  margin-bottom: 15px;
}
.el-aside {
  padding: 20px;
  border-right: 1px solid #f2f2f2;
}
.el-main,
.el-aside {
  text-align: center;
}
.el-footer {
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 250px !important;
  border-top: 2px dotted #f2f2f2;
  padding: 5px 15px;
  text-align: center;
  background-color: #eaefe8;
  z-index: 6;
  box-shadow: rgba(0, 0, 0, 0.65) 0px -2px 9px 1px;
}
.el-form-item {
  margin-bottom: 0;
}
.excel-p {
  text-align: center;
  margin-top: -217px;
  color: #909090;
  display: inline-block;
  border: 1px dashed #d9d9d9;
  width: 80%;
  margin-left: 10%;
  padding:20px;
}
.my-footer {
  margin: 20px auto;
  text-align: center;
  display: flex;
}
.my-footer span {
  display: inline-block;
  width: 25px;
  height: 25px;
  background-size: cover;
}
.me {
  background: url(data:image/png;base64,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);
}
.github {
  background: url(data:image/png;base64,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);
}
</style>
