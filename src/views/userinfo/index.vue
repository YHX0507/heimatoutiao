<template>
  <el-card v-loading="loading">
    <breadcrumb slot="header">
      <template slot="title">用户信息</template>
    </breadcrumb>
    <el-form ref="myForm" :model="dataForm" :rules="dataRules" style="margin-left:100px;" label-width="100px">
        <el-form-item prop="name" label="用户名：" style="width:40%">
            <el-input v-model="dataForm.name"></el-input>
        </el-form-item>
        <el-form-item label="简介：" style="width:40%">
            <el-input v-model="dataForm.intro"></el-input>
        </el-form-item>
        <el-form-item prop="email" label="邮箱：" style="width:40%">
            <el-input v-model="dataForm.email"></el-input>
        </el-form-item>
        <el-form-item prop="mobile" label="手机号：" style="width:40%">
            <el-input disabled v-model="dataForm.mobile"></el-input>
        </el-form-item>
        <el-form-item>
            <el-button @click="saveUserInfo" type="primary">保存</el-button>
        </el-form-item>
    </el-form>
    <el-upload class="uesrImg" action="" :http-request="uploadUserImg" :show-file-list="false">
        <img :src="dataForm.photo" alt="">
    </el-upload>
  </el-card>
</template>

<script>
import { getUserInfo, saveUserInfo } from '../../actions/articles'
import eventBus from '../../utils/eventBus'
export default {
  data () {
    return {
      dataForm: {
        name: '',
        intro: '',
        photo: '',
        email: '',
        mobile: ''
      },
      dataRules: {
        name: [{ required: true, message: '请输入用户名' }, { min: 1, max: 7, message: '请控制在1-7字符之间' }],
        email: [{ required: true, message: '请输入邮箱' }, { pattern: /^([a-zA-Z]|[0-9])(\w|-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/, message: '邮箱格式不正确' }]
      },
      loading: false
    }
  },
  methods: {
    // 上传用户图片
    async uploadUserImg (params) {
      this.loading = true
      let fd = new FormData()
      fd.append('photo', params.file)
      let result = await this.$axios({
        url: '/user/photo',
        method: 'patch',
        data: fd
      })
      this.loading = false
      this.dataForm.photo = result.data.photo
      eventBus.$emit('eventBus')
    },
    // 保存用户信息
    async saveUserInfo () {
      await this.$refs.myForm.validate()
      await saveUserInfo(this.dataForm)
      this.$message({
        type: 'success',
        message: '保存信息成功'
      })
      eventBus.$emit('eventBus')
    },
    // 获取用户信息
    async getUserInfo () {
      let result = await getUserInfo()
      this.dataForm = result.data
    }
  },
  created () {
    this.getUserInfo()
  }
}
</script>

<style lang="less" scoped>
.uesrImg{
    position: absolute;
    top: 200px;
    right: 200px;
    img{
        width: 200px;
        height: 200px;
        border-radius: 50%;
    }
}
</style>
