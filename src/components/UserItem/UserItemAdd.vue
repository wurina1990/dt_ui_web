<template>
  <!--隐藏新增from表单-->
  <el-dialog title="新增用户" :visible.sync="saveFormValue" width="850px">
    <el-form :model="addForm" ref="addForm" :rules="rules" label-width="92px">
      <el-form-item label="账号:" prop="userName" class="username">
        <el-input clearable style="width: 250px" v-model="addForm.userName"></el-input>
      </el-form-item>
      <el-form-item label="密码:" prop="pwd" class="pwd">
        <el-input clearable style="width: 250px" v-model="addForm.pwd" type="password" maxlength="20"></el-input>
      </el-form-item>
      <el-form-item label="确认密码:" prop="confirmPwd" class="pwd">
        <el-input clearable style="width: 250px" v-model="addForm.confirmPwd" type="password"></el-input>
      </el-form-item>
      <el-checkbox v-model="addForm.checkedUpPwd" class="box">首次登陆修改密码</el-checkbox>
      <el-checkbox v-model="addForm.checkedPwd" class="box">密码满足复杂度要求</el-checkbox>
      <el-form-item label="员工:" prop="staffValue" class="staff">
        <el-select clearable value="" v-model="addForm.staffValue" style="width: 250px">
          <el-option v-for="(item,index) in staffStatusOptions" :key="index" :label="item.employeeName"
                     :value="item.sId"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="用户有效期:" prop="userExpirationDate" class="date1">
        <div class="block">
          <el-date-picker
            style="width: 250px"
            type="datetime"
            :disabled="isUserFlg"
            value-format="timestamp"
            @change="changeSearchForuserExpirationDate" v-model="addForm.userExpirationDate">
          </el-date-picker>
          <el-checkbox @change="checkedUser" v-model="addForm.uAlways" :disabled="isCheFlgUser">用户始终有效
          </el-checkbox>
        </div>
      </el-form-item>
      <el-form-item label="密码有效期:" prop="pwdValidityPeriod" class="date2">
        <el-input clearable @blur="blurSearchForAlways" style="width: 250px" :disabled="isAlwaysFlg"
                  v-model.number="addForm.pwdValidityPeriod" maxlength="4"></el-input>
        <span>天</span>
        <el-checkbox @change="checkedAlways" v-model="addForm.pwdAlways" :disabled="isCheFlgAlways">密码始终有效
        </el-checkbox>
      </el-form-item>
      <el-form-item prop="rolesId" class="transfers">
        <div class="transfer">
          <el-transfer
            filterable
            :filter-method="filterMethod"
            :button-texts="['移除', '添加']"
            :titles="['角色信息', '添加角色信息']"
            filter-placeholder="请输入角色信息"
            v-model="addForm.rolesId"
            :data="rolesData"
          >
          </el-transfer>
        </div>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="resetForm('addForm')">保存并继续</el-button>
      <el-button @click="saveFormValue = false">取 消</el-button>
      <el-button type="primary" @click="determine('addForm')">确 定</el-button>
    </div>
  </el-dialog>
</template>


<script>
  import {repFindRoles, repGetStaff, repGetUserName, repSaveUserInfo} from '../../api'
  import PubSub from 'pubsub-js'
  import message from '../../utils/Message'

  export default {
    data () {
      var userName = (rule, value, callback) => {
        console.log(value)
        var reUserName = /^[a-zA-Z][0-9a-zA-Z_]{3,9}$/
        if (!value) {
          return callback(new Error('账号不能为空~'))
        }
        if (!reUserName.test(value)) {
          return callback(new Error('长度在4-10之间，已字母开头，只能包含字符、数字和下划线~'))
        } else {
          const userData = repGetUserName(value)
          userData.then((result) => {
            if (result.data !== null) {
              callback(new Error('用户名已被注册'))
            } else {
              callback()
            }
          })
        }

      }
      var pwd = (rule, value, callback) => {
        var pwd = /^[A-Za-z0-9]{6,20}$/
        if (value === '') {
          callback(new Error('请输入密码'))
        }
        if (this.addForm.checkedPwd === true) {
          if (!pwd.test(value)) {
            callback(new Error('密码规则：6-20位字母或数字组合'))
          } else {
            callback()
          }
        } else {
          callback()
        }
      }
      var confirmPwd = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请再次输入密码'))
        } else if (value !== this.addForm.pwd) {
          callback(new Error('两次输入密码不一致!'))
        } else {
          callback()
        }
      }
      var pwdValidityPeriod = (rule, value, callback) => {
        if (this.addForm.pwdAlways !== true) {
          if (!value) {
            return callback(new Error('天数不能为空'))
          }
          if (!Number.isInteger(value)) {
            callback(new Error('请输入数字值'))
          } else {
            {
              callback()
            }
          }
        } else {
          callback()
        }
      }
      var staffValue = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('必须选择一个员工~'))
        } else {
          callback()
        }
      }
      var userExpirationDate = (rule, value, callback) => {
        if (this.addForm.uAlways !== true) {
          if (value === '' || value === null) {
            callback(new Error('必须选择一个时间~'))
          } else {
            callback()
          }
        } else {
          callback()
        }
      }
      var rolesId = (rule, value, callback) => {
        if (value.length === 0) {
          callback(new Error('必须选择一个角色~'))
        } else {
          callback()
        }
      }
      return {
        saveFormValue: false, //负责显示from
        rolesData: [],//获得所有角色
        isCheFlgAlways: false,//密码始终有效 判断标识
        isCheFlgUser: false,//用户始终有效 判断标识
        isAlwaysFlg: false,//密码有效期 判断标识
        isUserFlg: false,//用户有效期 判断标识
        staffStatusOptions: [], //获得员工信息
        addForm: {
          userName: '',//用户名
          pwd: '',//密码
          confirmPwd: '',//确认密码
          checkedPwd: false, //密码满足复杂度要求  checked
          checkedUpPwd: false, //首次登陆修改密码 修改  checked
          uAlways: false, //用户始终有效  checked
          pwdAlways: false, //密码始终有效 checked
          rolesId: [], //角色 ids
          staffValue: '', //员工对象
          pwdValidityPeriod: '', //密码有效期
          userExpirationDate: '' //用户有效期
        },
        rules: {
          userName: [
            {validator: userName, trigger: 'blur'}
          ],
          pwd: [
            {validator: pwd, trigger: 'blur'}
          ],
          confirmPwd: [
            {validator: confirmPwd, trigger: 'blur'}
          ],
          pwdValidityPeriod: [
            {validator: pwdValidityPeriod, trigger: 'blur'}
          ],
          staffValue: [
            {validator: staffValue, trigger: 'blur'}
          ],
          userExpirationDate: [
            {validator: userExpirationDate, trigger: 'blur'}
          ],
          rolesId: [
            {validator: rolesId, trigger: 'blur'}
          ]
        },
      }
    },
    async mounted () {
      //获得传来的标识 显示 隐藏form
      PubSub.subscribe('saveFormValue', (msg, saveFormValue) => {
        this.saveFormValue = saveFormValue
        //查询角色信息
        const resultRoles = repFindRoles()
        resultRoles.then((result) => {
          if (result.code === 200) {
            const generateData = _ => {
              const data = []
              const roles = result.data
              roles.forEach((role, index) => {
                data.push({
                  label: role.rName,
                  key: role.rid,
                  roles: roles[index]
                })
              })
              return data
            }
            this.rolesData = generateData()
          }
        })
        //获得员工信息
        const resultStaff = repGetStaff()
        resultStaff.then((result) => {
          if (result.code === 200) {
            this.staffStatusOptions = result.data
            console.log(this.staffStatusOptions)
          }
        })
      })
    },
    methods: {
      //通过关键字搜索
      filterMethod (query, item) {
        return item.roles.rName.indexOf(query) > -1
      },
      //获得 checkedAlways flg如果=true 就禁用input 输入框
      checkedAlways (flg) {
        this.isAlwaysFlg = flg === true
      },
      //获得 checkedUser flg如果=true 就禁用input 输入框
      checkedUser (flg) {
        this.isUserFlg = flg === true
      },
      //blurSearchForAlways 失去焦点时 判断值是否为空 如果不为空 锁定按钮
      blurSearchForAlways () {
        this.isCheFlgAlways = !!this.addForm.pwdValidityPeriod
      },
      //用户有效期 失去焦点时 判断值是否为空 如果不为空 锁定按钮
      changeSearchForuserExpirationDate () {
        this.isCheFlgUser = !!this.addForm.userExpirationDate
      },
      //点击确定
      determine (formName) {
        this.$refs[formName].validate((valid) => {
          console.log(this.addForm)
          if (valid) {
            const resultSave = repSaveUserInfo(this.addForm)
            resultSave.then((result) => {
              console.log(result)
              if (result.code === 200) {
                this.saveFormValue = false
                message.successMessage(result.msg)
              }
            })
          } else {
            console.log('error submit!!')
            return false
          }
        })
      },
      //保存并继续
      resetForm (formName) {
        this.$refs[formName].validate((valid) => {
          console.log(this.addForm)
          if (valid) {
            const resultSave = repSaveUserInfo(this.addForm)
            resultSave.then((result) => {
              console.log(result)
              if (result.code === 200) {
                alert('添加确认');
                message.successMessage(result.msg)
                this.$refs[formName].resetFields()
              }
            })
          } else {
            console.log('error submit!!')
            return false
          }
        })

      }
    }
  }
</script>


<style scope lang="scss">
  .transfer-footer {
    margin-left: 20px;
    padding: 6px 5px;
  }
//头部样式
  .el-dialog__header {
    text-align: center;
    background-color: #e8e8e8;
    border-top-left-radius: 25px;
    border-top-right-radius: 25px;
    .el-dialog__title {
      font-family: "宋体";
      font-size: 20px;
    }
  }

  //表单关闭
  .el-dialog__headerbtn {
    background-color: #F56C6C;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    border-bottom-left-radius: 10px;
    border-bottom-right-radius: 10px;
    width: 20px;
    height: 20px;
  }

  //表单边框
  .el-dialog {
    border-top-left-radius: 25px;
    border-top-right-radius: 25px;
    border-bottom-left-radius: 25px;
    border-bottom-right-radius: 25px;
  }



  //用户
  .el-form-item.username {
    .el-form-item__label {
      font-family: "宋体";
      font-size: 15px;
    }
  }

  //密码
  .el-form-item.pwd {
    float: left;
    padding-right: 55px;
    .el-form-item__label {
      color: #F56C6C;
      font-family: "宋体";
      font-size: 15px;
    }
  }

  //复选框
  .el-checkbox.box {
    margin-left: 95px;
    margin-right: 155px;
    padding-bottom: 25px;
    .el-checkbox__label {
      font-family: "宋体";
      font-size: 15px;
    }
  }

  //员工
  .el-form-item.staff {
    .el-form-item__label {
      font-family: "宋体";
      font-size: 15px;
    }
  }

  //有效期
  .el-form-item.date1 {
    .el-form-item__label {
      color: #F56C6C;
      font-family: "宋体";
    }
    .el-form-item__content {
      .block {
        .el-checkbox {
          padding-left: 56px;
          font-family: "宋体";
        }
      }
    }
  }

  //密码有效期
  .el-form-item.date2 {
    .el-form-item__label {
      color: #F56C6C;
      font-family: "宋体";
    }
    .el-form-item__content {
      font-family: "宋体";
      .el-checkbox {
        padding-left: 32px;
        font-family: "宋体";
      }
    }
  }

  //自定义添加转移
  .transfer {
    margin-left: 50px;
    .el-transfer {
      .el-transfer__buttons {
        width: 150px;
        .el-button.el-button--primary.is-disabled.el-transfer__button.is-with-texts {
          margin-left: 0;
          width: 89px;
          font-family: "宋体";
        }
        .el-button.el-button--primary.el-transfer__button.is-with-texts {
          margin-left: 0;
          width: 89px;
          font-family: "宋体";
        }
      }
    }
  }

  //自定义transfers组建内容样式
  .el-transfer-panel__body {
    .el-transfer-panel__filter.el-input.el-input--small.el-input--prefix {
      width: 168px;
    }
    .el-checkbox-group.el-transfer-panel__list.is-filterable {
      .el-checkbox.el-transfer-panel__item {
        margin-left: 20px;
        display: block;
        font-family: "宋体";
      }
    }
  }


</style>
