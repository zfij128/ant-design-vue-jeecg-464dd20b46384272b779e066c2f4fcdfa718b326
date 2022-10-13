<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="应用包名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="appPackageName">
              <a-input v-model="model.appPackageName" placeholder="请输入应用包名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="应用名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="appName">
              <a-input v-model="model.appName" placeholder="请输入应用名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="应用描述" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="appDesc">
              <a-textarea v-model="model.appDesc" rows="4" placeholder="请输入应用描述" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="大小" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="appSize">
              <a-input v-model="model.appSize" placeholder="请输入大小"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="交互方法" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="interMethod">
              <a-input v-model="model.interMethod" placeholder="请输入交互方法"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="视频链接" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="videoUrl">
              <a-input v-model="model.videoUrl" placeholder="请输入视频链接"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="标注链接" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="sign">
              <a-input v-model="model.sign" placeholder="请输入标注链接"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="应用下载链接" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="downLoadurl">
              <j-upload v-model="model.downLoadurl"   ></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="应用类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="appType">
              <j-dict-select-tag type="list" v-model="model.appType" dictCode="apktype" placeholder="请选择应用类型" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'YsApkInfoForm',
    components: {
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        model:{
         },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
        },
        url: {
          add: "/ysapkinfo/ysApkInfo/add",
          edit: "/ysapkinfo/ysApkInfo/edit",
          queryById: "/ysapkinfo/ysApkInfo/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
    },
    created () {
       //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
    },
    methods: {
      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            httpAction(httpurl,this.model,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
         
        })
      },
    }
  }
</script>