<template>
  <div>
    <el-form :model="form"
             :rules="rules"
             ref="createGroupForm"
             label-width="100px">
      <el-form-item label="群ID">
        <el-input v-model="form.groupID"></el-input>
      </el-form-item>
      <el-form-item label="群名称"
                    prop="name">
        <el-input v-model="form.name"></el-input>
      </el-form-item>
      <el-form-item label="群类型">
        <el-select v-model="form.type">
          <el-option label="Work"
                     :value="TIM.TYPES.GRP_WORK"></el-option>
          <el-option label="Public"
                     :value="TIM.TYPES.GRP_PUBLIC"></el-option>
          <el-option label="Meeting"
                     :value="TIM.TYPES.GRP_MEETING"></el-option>
          <el-option label="AVChatRoom"
                     :value="TIM.TYPES.GRP_AVCHATROOM"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="群头像地址">
        <el-input v-model="form.avatar"></el-input>
      </el-form-item>
      <el-form-item label="群简介">
        <el-input type="textarea"
                  v-model="form.introduction"
                  :maxlength="240"></el-input>
      </el-form-item>
      <el-form-item label="群公告">
        <el-input type="textarea"
                  v-model="form.notification"
                  :maxlength="300"></el-input>
      </el-form-item>
      <el-form-item label="加群方式">
        <el-radio-group v-model="form.joinOption"
                        :disabled="joinOptionDisabled">
          <el-radio label="FreeAccess">自由加群</el-radio>
          <el-radio label="NeedPermission">需要验证</el-radio>
          <el-radio label="DisableApply">禁止加群</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="群成员列表">
        <el-select v-model="form.memberList"
                   default-first-option
                   multiple
                   filterable
                   remote
                   :disabled="form.type === TIM.TYPES.GRP_AVCHATROOM"
                   :remote-method="handleSearchUser"
                   :loading="loading"
                   placeholder="请输入群成员 userID">
          <el-option v-for="item in options"
                     :key="item"
                     :label="item"
                     :value="item"></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <div slot="footer">
      <el-button type="primary"
                 @click="onSubmit('createGroupForm')">立即创建</el-button>
      <el-button @click="closeCreateGroupModel">取消</el-button>
    </div>
  </div>
</template>

<script>
import {
  Form,
  FormItem,
  Input,
  Select,
  Option,
  Radio,
  RadioGroup,
} from 'element-ui'
export default {
  components: {
    ElForm: Form,
    ElFormItem: FormItem,
    ElInput: Input,
    ElSelect: Select,
    ElOption: Option,
    ElRadioGroup: RadioGroup,
    ElRadio: Radio,
  },
  data() {
    return {
      form: {
        groupID: '',
        name: '',
        type: this.TIM.TYPES.GRP_WORK,
        avatar: '',
        introduction: '',
        notification: '',
        joinOption: 'FreeAccess',
        memberList: [],
      },
      options: [],
      loading: false,
      rules: {
        name: [{ required: true, message: '请输入群名称', trigger: 'blur' }],
      },
    }
  },
  computed: {
    joinOptionDisabled() {
      return [
        this.TIM.TYPES.GRP_WORK,
        this.TIM.TYPES.GRP_MEETING,
        this.TIM.TYPES.GRP_AVCHATROOM,
      ].includes(this.form.type)
    },
  },
  methods: {
    onSubmit(ref) {
      this.$refs[ref].validate((valid) => {
        if (!valid) {
          return false
        }
        this.createGroup()
      })
    },
    closeCreateGroupModel() {
      this.$store.commit('updateCreateGroupModelVisible', false)
    },
    createGroup() {
      this.tim
        .createGroup(this.getOptions())
        .then((imResponse) => {
          this.$store.commit('showMessage', {
            message: `群组：【${imResponse.data.group.name}】创建成功`,
            type: 'success',
          })
          this.closeCreateGroupModel()
        })
        .catch((error) => {
          this.$store.commit('showMessage', {
            type: 'error',
            message: error.message,
          })
        })
    },
    getOptions() {
      let options = {
        ...this.form,
        memberList: this.form.memberList.map((userID) => ({ userID })),
      }
      if (
        [this.TIM.TYPES.GRP_WORK, this.TIM.TYPES.GRP_AVCHATROOM].includes(
          this.form.type
        )
      ) {
        delete options.joinOption
      }
      return options
    },
    handleSearchUser(userID) {
      if (userID !== '') {
        this.loading = true
        this.tim
          .getUserProfile({ userIDList: [userID] })
          .then(({ data }) => {
            this.options = data.map((item) => item.userID)
            this.loading = false
          })
          .catch((error) => {
            this.$store.commit('showMessage', {
              type: 'error',
              message: error.message,
            })
          })
      }
    },
  },
}
</script>

<style lang="stylus" scoped></style>
