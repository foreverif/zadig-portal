<template>
  <div class="create-project">
    <el-dialog :fullscreen="true" custom-class="create-project" :before-close="handleClose" :visible.sync="dialogVisible">
      <div class="project-contexts-modal">
        <header class="project-contexts-modal__header"></header>
        <div class="project-contexts-modal__content">
          <h1 class="project-contexts-modal__content-title">{{isEdit?'修改项目信息':'开始新建项目'}}</h1>
          <div class="project-contexts-modal__content-container">
            <div class="project-settings__inputs-container">
              <el-tabs style="width: 100%;" v-if="isEdit" v-model="activeName">
                <el-tab-pane label="基本信息" name="base"></el-tab-pane>
                <el-tab-pane label="高级配置" name="advance"></el-tab-pane>
              </el-tabs>
              <el-form
                :model="projectForm"
                :rules="rules"
                label-position="top"
                ref="projectForm"
                label-width="100px"
                class="demo-projectForm"
              >
                <el-form-item label="项目名称" v-show="activeName !=='advance'" prop="project_name">
                  <el-input @keyup.native="()=>projectForm.project_name=projectForm.project_name.trim()" v-model="projectForm.project_name"></el-input>
                </el-form-item>

                <el-form-item label="项目主键" v-show="activeName !=='advance'" prop="product_name">
                  <span slot="label">
                    项目主键
                    <el-tooltip effect="dark" content="项目主键是该项目资源的全局唯一标识符，用于该项目下所有资源的引用与更新，默认自动生成，同时支持手动指定，创建后不可更改" placement="top">
                      <i class="el-icon-question"></i>
                    </el-tooltip>
                    <el-button v-if="!isEdit&&!editProjectName" @click="editProjectName=true" type="text">编辑</el-button>
                    <el-button v-if="!isEdit&&editProjectName" @click="editProjectName=false" type="text">完成</el-button>
                  </span>
                  <el-input :disabled="!showProjectName" v-model="projectForm.product_name"></el-input>
                </el-form-item>
                <el-form-item label="项目管理员" v-if="!isEdit" prop="admins">
                  <el-select
                    v-model="projectForm.admins"
                    style="width: 100%;"
                    filterable
                    multiple
                    remote
                    :loading="loading"
                    placeholder="请输入用户名搜索用户"
                  >
                    <el-option
                      v-for="(user,index) in users"
                      :key="index"
                      :label="user.name ? `${user.account}(${user.name})` : user.account"
                      :value="user.uid"
                    >
                    <span v-if="user.identity_type">
                      <i class="iconfont" :class="'icon'+user.identity_type"></i>
                      <span>{{user.name ? `${user.account}(${user.name})` : user.account}}</span>
                    </span>
                    </el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label="项目权限" v-show="activeName !=='advance'" prop="public">
                  <el-radio-group v-model="projectForm.public">
                    <el-radio :label="true">公开</el-radio>
                    <el-radio :label="false">私有</el-radio>
                  </el-radio-group>
                </el-form-item>
                <el-form-item label="描述信息" v-show="activeName !=='advance'" prop="desc">
                  <el-input type="textarea" :rows="2" placeholder="请输入描述信息" v-model="projectForm.desc"></el-input>
                </el-form-item>
                <el-form-item v-if="!isEdit" v-show="activeName !=='advance'" label="项目特点" prop="desc">
                  <el-row :gutter="5">
                    <el-col :span="4">
                      <span>
                        基础设施
                        <el-tooltip placement="top">
                          <div slot="content">
                            Kubernetes：包括自建 K8s 和云厂商提供容器云服务
                            <br />主机：包括物理机和云厂商提供的云服务器
                          </div>
                          <i class="icon el-icon-question"></i>
                        </el-tooltip>
                      </span>
                    </el-col>
                    <el-col :span="10">
                      <el-radio-group size="mini" v-model="projectForm.product_feature.basic_facility">
                        <el-radio border label="kubernetes">Kubernetes</el-radio>
                        <el-radio border label="cloud_host">主机</el-radio>
                      </el-radio-group>
                    </el-col>
                  </el-row>
                  <el-row :gutter="5" v-if="projectForm.product_feature.basic_facility==='kubernetes'">
                    <el-col :span="4">
                      <span>
                        环境创建方式
                        <el-tooltip placement="top">
                          <div slot="content">
                            新建集成环境：在现有的 Kubernetes 集群新建环境
                            <br />托管现有环境：托管现有 Kubernetes 集群中的资源
                          </div>
                          <i class="icon el-icon-question"></i>
                        </el-tooltip>
                      </span>
                    </el-col>
                    <el-col :span="10">
                      <el-radio-group size="mini" v-model="projectForm.product_feature.create_env_type">
                        <el-radio border label="system">新建集成环境</el-radio>
                        <el-radio border label="external">托管现有环境</el-radio>
                      </el-radio-group>
                    </el-col>
                  </el-row>
                  <el-row
                    v-if="projectForm.product_feature.basic_facility==='kubernetes'&&projectForm.product_feature.create_env_type==='system'"
                    :gutter="5"
                  >
                    <el-col :span="4">
                      <span>
                        服务部署方式
                        <el-tooltip placement="top">
                          <div slot="content">
                            K8s YAML 部署：使用 K8s 原生的 YAML配置方式部署服务
                            <br />Helm Chart 部署：使用 Helm 工具部署服务
                          </div>
                          <i class="icon el-icon-question"></i>
                        </el-tooltip>
                      </span>
                    </el-col>
                    <el-col :span="12">
                      <el-radio-group size="mini" v-model="projectForm.product_feature.deploy_type">
                        <el-radio border label="k8s">K8s YAML 部署</el-radio>
                        <el-radio border label="helm">Helm Chart 部署</el-radio>
                      </el-radio-group>
                    </el-col>
                  </el-row>
                </el-form-item>
                <div v-if="isEdit" v-show="activeName==='advance'">
                  <el-form-item label="服务部署超时（分钟）" prop="timeout">
                    <el-input v-model.number="projectForm.timeout"></el-input>
                  </el-form-item>
                  <el-form-item label="自定义交付物名称">
                    <span slot="label">
                      自定义交付物名称
                      <el-tooltip effect="dark" placement="top">
                        <div slot="content">
                          镜像和 TAR 包规则可以通过变量和常量组装生成：
                          <br />
                          <span class="tooltip-key" v-html="'{{.TIMESTAMP}}'"></span> 时间戳
                          <br />
                          <span class="tooltip-key" v-html="'{{.TASK_ID}}'"></span> 工作流任务 ID
                          <br />
                          <span class="tooltip-key" v-html="'{{.REPO_BRANCH}}'"></span> 代码分支名称
                          <br />
                          <span class="tooltip-key" v-html="'{{.REPO_PR}}'"></span> 代码 PR ID
                          <br />
                          <span class="tooltip-key" v-html="'{{.REPO_TAG}}'"></span> 代码 TAG
                          <br />
                          <span class="tooltip-key" v-html="'{{.REPO_COMMIT_ID}}'"></span> 代码 Commit ID
                          <br />
                          <span class="tooltip-key" v-html="'{{.PROJECT}}'"></span> 项目名称
                          <br />
                          <span class="tooltip-key" v-html="'{{.SERVICE}}'"></span> 服务名称
                          <br />
                          <span class="tooltip-key" v-html="'{{.ENV_NAME}}'">${ENV_NAME}</span> 环境名称
                          <br />注意：常量字符只能是大小写字母、数字、中划线、下划线和点，即 [a-zA-Z0-9_.-]，首个字符不能是&nbsp;.&nbsp;或&nbsp;-。不能超过 127 个字符
                        </div>
                        <i class="el-icon-question"></i>
                      </el-tooltip>
                    </span>
                    <CusDeliverable
                      v-show="activeName==='advance'"
                      :customImageRule="projectForm.custom_image_rule"
                      :customTarRule="projectForm.custom_tar_rule"
                      ref="cusDeliverable"
                      v-if="isEdit"
                    />
                  </el-form-item>
                  <el-form-item v-if="isHelm">
                    <span slot="label">
                      版本交付 hook 配置
                      <el-tooltip effect="dark" content="完成配置后，交付中心-创建版本可 hook 外部系统" placement="top">
                        <i class="el-icon-question"></i>
                      </el-tooltip>
                      <el-switch v-model="projectForm.delivery_version_hook.enable" style="margin-left: 10px;"></el-switch>
                    </span>
                    <el-row :gutter="10" v-if="projectForm.delivery_version_hook.enable">
                      <el-col :span="3" style="text-align: right;">
                        * URL
                      </el-col>
                      <el-col :span="10">
                        <el-form-item prop="delivery_version_hook.hook_host">
                          <el-select v-model="projectForm.delivery_version_hook.hook_host" placeholder="选择外部系统" size="small" clearable style="width: 100%;">
                            <el-option v-for="external in externalList" :key="external.id" :label="external.server" :value="external.server"></el-option>
                          </el-select>
                        </el-form-item>
                      </el-col>
                      <el-col :span="10">
                        <el-form-item prop="delivery_version_hook.path">
                          <el-input v-model="projectForm.delivery_version_hook.path" placeholder="输入访问路径" size="small"></el-input>
                        </el-form-item>
                      </el-col>
                    </el-row>
                  </el-form-item>
                </div>
              </el-form>
            </div>
          </div>
        </div>
        <footer class="project-contexts-modal__footer">
          <el-button class="create-btn" type="primary" plain @click="submitForm('projectForm')">{{isEdit?'确认修改':'立即创建'}}</el-button>
        </footer>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import {
  usersAPI,
  createProjectAPI,
  getSingleProjectAPI,
  updateSingleProjectAPI,
  getExternalSystemsAPI
} from '@api'
import CusDeliverable from './components/cusDeliverable.vue'

const pinyin = require('pinyin')
const validateProductName = (rule, value, callback) => {
  if (typeof value === 'undefined' || value === '') {
    callback(new Error('填写项目主键'))
  } else {
    if (!/^[a-z0-9-]+$/.test(value)) {
      callback(new Error('项目主键只支持小写字母和数字，特殊字符只支持中划线'))
    } else {
      callback()
    }
  }
}
const validateDeployTimeout = (rule, value, callback) => {
  const reg = /^[0-9]+.?[0-9]*/
  if (!reg.test(value)) {
    callback(new Error('时间应为数字'))
  } else {
    if (value > 0) {
      callback()
    } else {
      callback(new Error('请输入正确的时间范围'))
    }
  }
}
export default {
  components: {
    CusDeliverable
  },
  data () {
    return {
      activeName: 'base',
      dialogVisible: true,
      users: [],
      loading: false,
      editProjectName: false,
      radio: true,
      projectForm: {
        project_name: '',
        product_name: '',
        admins: [],
        timeout: null,
        desc: '',
        enabled: true,
        public: true,
        product_feature: {
          basic_facility: 'kubernetes',
          deploy_type: 'k8s',
          create_env_type: 'system'
        }
      },
      rules: {
        project_name: [
          { required: true, message: '请输入项目名称', trigger: 'blur' }
        ],
        product_name: [
          { required: true, trigger: 'change', validator: validateProductName }
        ],
        admins: [
          {
            type: 'array',
            required: true,
            message: '请选择项目管理员',
            trigger: 'change'
          }
        ],
        enabled: [
          {
            type: 'boolean',
            required: true,
            message: '请选择项目是否启用项目',
            trigger: 'change'
          }
        ],
        timeout: [
          {
            required: true,
            trigger: 'change',
            validator: validateDeployTimeout
          }
        ],
        public: [
          { required: true, message: '项目权限不能为空', trigger: 'blur' }
        ],
        'delivery_version_hook.hook_host': {
          required: true, message: '请选择外部系统', trigger: 'blur'
        },
        'delivery_version_hook.path': {
          required: true, message: '请输入访问路径', trigger: 'blur'
        }
      },
      externalList: []
    }
  },
  methods: {
    getUsers () {
      const payload = {}
      usersAPI(payload).then(res => {
        this.users = this.$utils.deepSortOn(res.users, 'name')
      })
    },
    remoteMethod (query) {
      if (query !== '') {
        this.loading = true
        const payload = {
          name: query
        }
        usersAPI(payload).then(res => {
          this.loading = false
          this.users = this.$utils.deepSortOn(res.users, 'name')
        })
      } else {
        this.users = []
      }
    },
    handleClose () {
      if (this.isEdit) {
        this.$router.push(`/v1/projects/detail/${this.projectName}`)
      } else {
        this.$router.push('/v1/projects')
      }
    },
    createProject (payload) {
      createProjectAPI(payload).then(res => {
        this.$message({
          type: 'success',
          message: '新建项目成功'
        })
        this.$store.dispatch('getProjectList')
        if (payload.product_feature.basic_facility === 'kubernetes') {
          if (payload.product_feature.create_env_type === 'external') {
            this.$router.push(
              `/v1/projects/create/${payload.product_name}/host/config?step=1`
            )
            return
          }
          if (payload.product_feature.deploy_type === 'k8s') {
            this.$router.push(
              `/v1/projects/create/${payload.product_name}/basic/info?rightbar=step`
            )
          }
          if (payload.product_feature.deploy_type === 'helm') {
            this.$router.push(
              `/v1/projects/create/${payload.product_name}/helm/info?rightbar=step`
            )
          }
        } else if (payload.product_feature.basic_facility === 'cloud_host') {
          this.$router.push(
            `/v1/projects/create/${payload.product_name}/not_k8s/info`
          )
        }
      })
    },
    updateSingleProject (projectName, payload) {
      updateSingleProjectAPI(projectName, payload).then(res => {
        this.$message({
          type: 'success',
          message: '更新项目成功'
        })
        this.$store.dispatch('getProjectList')
        this.$router.push('/v1/projects')
      })
    },
    getProject (projectName) {
      getSingleProjectAPI(projectName).then(res => {
        this.projectForm = res
        if (res.team_id === 0) {
          this.projectForm.team_id = null
        }
        if (!res.timeout) {
          this.$set(this.projectForm, 'timeout', 10)
        }
        if (this.isHelm) {
          getExternalSystemsAPI().then(res => {
            this.externalList = res.external_system
          })
          if (!this.projectForm.delivery_version_hook) {
            this.$set(this.projectForm, 'delivery_version_hook', {
              enable: false,
              hook_host: '',
              path: ''
            })
          }
        }
      })
    },
    submitForm (formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          if (this.isEdit) {
            this.$refs.cusDeliverable.saveConfig()
            this.projectForm.custom_image_rule = this.$refs.cusDeliverable.custom_image_rule
            this.projectForm.custom_tar_rule = this.$refs.cusDeliverable.custom_tar_rule
            this.updateSingleProject(
              this.projectForm.product_name,
              this.projectForm
            )
          } else {
            this.projectForm.timeout = 10
            if (
              this.projectForm.product_feature.basic_facility === 'cloud_host'
            ) {
              this.projectForm.product_feature.deploy_type = 'k8s'
              this.projectForm.product_feature.create_env_type = 'system'
            }
            this.createProject(this.projectForm)
          }
        } else {
          return false
        }
      })
    },
    resetForm (formName) {
      this.$refs[formName].resetFields()
    }
  },
  watch: {
    'projectForm.project_name': {
      handler (val, old_val) {
        if (!this.isEdit) {
          this.projectForm.product_name = pinyin(val, {
            style: pinyin.STYLE_NORMAL
          }).join('')
        }
      }
    }
  },
  computed: {
    currentUserId () {
      return this.$store.state.login.userinfo.uid
    },
    isEdit () {
      return this.$route.path.includes('/projects/edit')
    },
    showProjectName () {
      return !this.isEdit && this.editProjectName
    },
    projectName () {
      if (this.isEdit) {
        return this.$route.params.project_name
      } else {
        return false
      }
    },
    isHelm () {
      return this.projectForm.product_feature && this.projectForm.product_feature.deploy_type === 'helm'
    }
  },
  mounted () {
    if (this.isEdit) {
      this.getProject(this.projectName)
    } else {
      this.getUsers()
      this.projectForm.admins.push(this.currentUserId)
    }
  }
}
</script>

<style lang="less" >
.tooltip-key {
  display: inline-block;
  width: 130px;
}

.create-project {
  .icon {
    cursor: pointer;
  }

  .el-dialog__headerbtn {
    font-size: 40px;
  }

  .el-dialog__body {
    padding: 5px 20px;
  }

  .create-btn {
    color: #1989fa;
    background: #fff;
    border-color: #1989fa;

    &:hover {
      color: #fff;
      background: #1989fa;
      border-color: #1989fa;
    }
  }

  .project-contexts-modal {
    height: 100%;

    .project-contexts-modal__header {
      display: flex;
      align-items: flex-end;
      justify-content: space-between;
      padding: 0 50px;
    }

    .project-contexts-modal__footer {
      display: flex;
      align-items: center;
      justify-content: center;
      height: 60px;
    }

    .project-contexts-modal__content {
      display: flex;
      flex: 1;
      flex-direction: column;
      align-items: center;
      justify-content: center;

      .project-contexts-modal__content-title {
        margin: 0;
        margin-bottom: 20px;
        color: #000;
        font-weight: bold;
        font-size: 27px;
        text-align: center;
      }

      .project-settings__inputs-container {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        justify-content: flex-start;
        width: 800px;

        .el-form {
          width: 100%;

          .el-form-item {
            margin-bottom: 5px;
          }
        }

        .small-title {
          color: #ccc;
          font-size: 12px;
        }

        .el-radio--mini {
          &.is-bordered {
            width: 135px;
            margin-right: 0;
          }
        }
      }
    }
  }
}
</style>
