<template>
  <!-- 数据同步对话框组件 参数说明
    show            ： 显示对话框，Boolean，必填
    exportTemplateUrl : 导出模板Url，String，必填
    importDataUrl     : 导入请求Url，String，必填
    extendFileName    :  校验的文件扩展名，String，必填，需要校验多个扩展名时以半角逗号“,”分隔，如xls,xlsx
    multiFiles        : 是否支持上传多个文件，Boolean，非必填，默认false
    title             : 对话框标题，String，非必填，默认显示“数据导入”
    paramName         : 参数名，String，非必填，默认file
    requestType       : 参数名，String，非必填，默认request  传入localhost下载本地文件夹文件，不调用接口
    importCallback    : 文件上传结果的回调 默认返回接口的响应对象
   -->
  <div>
    <el-dialog
      :title="title" v-dialogDrag custom-class="el-dialog-widthSmall"
      :visible.sync="dialogVisible" @close="closeDialog">
      <div>
        <b>请选择文件（{{this.extendFileName}}格式）进行上传！&emsp;</b>
        <el-button type="primary" plain size="mini" @click="exportDeviceTemplate"
                   v-if="requestType!=='localhost'&& requestType!=='isShowImport'">导出模板
        </el-button>
        <el-button type="primary" plain size="mini" @click="exportLocalTemplate"
                   v-else-if="requestType==='localhost'&& requestType!=='isShowImport'">
          导出模板
        </el-button>
      </div>
      <el-upload
        action="dev/baseInfo/import"
        style="margin:auto; margin-top: 10px;border:1px solid #DCDFE6;border-radius: 4px;"
        drag
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        :file-list="fileList"
        :auto-upload="false"
        :on-change="fileListChange">
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将数据文件拖到此处，或<em>点击上传</em></div>
        <div slot="tip" class="el-upload__tip">
          <div style="display: inline;color:#d70000;font-size:14px;"
               class="uploadFileWarning"
               id="uploadFileWarning">
          </div>
        </div>
      </el-upload>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeDialog">取 消</el-button>
        <el-button type="primary" @click="handleConfirm">提 交</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default{
    name:'dataImportDialog',
    props:{
      show : {
        type : Boolean,
        required : true,
        default : false,
      },
      exportTemplateUrl : {
        type : String,
        required : true
      },
      importDataUrl : {
        type : String,
        required : true
      },
      extendFileName : {
        type : String,
        required : true
      },
      multiFiles : {
        type : Boolean,
        required : false,
        default : false,
      },
      title : {
        type : String,
        required : false
      },
      paramName : {
        type : String,
        required : false,
        default : "file",
      },
      requestType: {
        type: String,
        required: false,
        default: 'request',
      }
    },
    data(){
      return{
        dialogVisible:false,
        radioGroup:'',
        files :[],
        fileList:[
          /*{
            name:'abc.jpg'
          }*/
        ],
      }
    },
    watch:{
      show:function(val){
        this.dialogVisible = val;
      }
    },
    mounted() {

    },
    methods:{
      //上传
      handleRemove(file, fileList) {
        this.fileList = fileList;
      },
      handlePreview(file) {

      },

      // 判断上传文件后缀
      fileListChange(file, fileList) {
        //先判断传入的extendFileName
        let extendFileNames = this.extendFileName.split(",");
        let regExpRules = [];
        for(let i=0; i<extendFileNames.length; i++){
          regExpRules.push(new RegExp("(.*).("+extendFileNames[i]+")$","gim"))
        }
        // let regExp = /(.*).(zip)$/;
        let fileNames = [];
        let files = [];
        let that=this;
        $.each(fileList, function (key, val) {
          let ret = false;
          for(let i=0; i<regExpRules.length; i++){
            ret = ret || regExpRules[i].test(val.name);
          }
          if (!ret) {
            console.log(val.name + ":"+ret)
            $(".uploadFileWarning").text("上传的文件后缀必须为"+that.extendFileName+"格式！");
            return false;
          }
          if (fileNames.indexOf(val.name) != -1) {
            $(".uploadFileWarning").text("上传的文件重复！");
            return false;
          }
          //只能上传一个文件，用最后上传的覆盖
          if(!that.multiFiles){
            files = [];fileNames = [];
          }
          files.push(val);
          fileNames.push(val.name);
          if (fileNames !== '') {
            $('#uploadMad .el-upload-dragger').css('border-color', '#409eff');
          }
          $(".uploadFileWarning").text("");

        });
        this.files = fileNames;
        this.fileList = files;
      },
      /**
       * 关闭弹窗
       */
      closeDialog() {
        this.$emit('update:show', false);
        this.fileList = [];
        this.radioGroup='';
        $(".uploadFileWarning").text("");
      },
      /**
       * 导出模板按钮
       */
      exportDeviceTemplate(){
        $("#downloadFile").attr("src",window.root+this.exportTemplateUrl);
      },//下载本地文件
      exportLocalTemplate() {
        $("#downloadFile").attr("src", this.exportTemplateUrl);
      },
      /**
       * 确认按钮
       */
      handleConfirm(){
        let filePath = this.fileList;
        if (filePath.length === 0) {
          $(".uploadFileWarning").text("未选择文件！");
          return false;
        }
        let formData = new FormData();
        this.fileList.forEach(file => {
          formData.append(this.paramName, file.raw, file.raw.name);
        });
        this.$uploadFile(this.importDataUrl, formData).then(jsonData => {
          var datas = jsonData.data;
          if (!datas.success) {
            this.$message({
              type: 'error',
              message: datas.msg
            });
            this.closeDialog();
            return;
          }
          this.$emit('importCallback', datas);
          this.closeDialog();
        });
      },
    }
  }
</script>
<style lang="less" scoped>
  .tips{
    text-align:left;
    margin-top:20px;
    margin-bottom:10px;
    color:#d70000
  }
  .center{
    text-align: center;
  }
</style>
