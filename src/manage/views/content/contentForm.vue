<template>
  <div class="dr-contentForm">
    <el-form :model="formState.formData"
             :rules="formState.formData.type=='2'?flashRules:(formState.formData.type=='3'?twiterRules:rules)"
             ref="ruleForm" label-width="120px" class="demo-ruleForm">
      <el-form-item :label="$t('contents.categories')" prop="categories">
        <el-cascader size="small" expand-trigger="hover" :options="contentCategoryList.docs"
                     v-model="formState.formData.categories" @change="handleChangeCategory" :props="categoryProps">
        </el-cascader>
      </el-form-item>
      <el-form-item :label="$t('contents.type')" prop="type">
        <el-radio class="radio" v-model="formState.formData.type" label="1">{{$t("contents.type_normal")}}</el-radio>
      </el-form-item>
      <div v-if="formState.formData.type == 1">
        <el-form-item :label="$t('contents.title')" prop="title">
          <el-input size="small" v-model="formState.formData.title"></el-input>
        </el-form-item>
        <el-form-item :label="$t('contents.stitle')" prop="stitle">
          <el-input size="small" v-model="formState.formData.stitle"></el-input>
        </el-form-item>
        <el-form-item :label="$t('contents.from')" prop="from">
          <el-radio class="radio" v-model="formState.formData.from" label="1">{{$t("contents.from_1")}}</el-radio>
          <el-radio class="radio" v-model="formState.formData.from" label="2">{{$t("contents.from_2")}}</el-radio>
          <el-radio class="radio" v-model="formState.formData.from" disabled label="3">{{$t("contents.from_3")}}
          </el-radio>
        </el-form-item>
        <el-form-item :label="$t('contents.enable')" prop="state">
          <el-switch :on-text="$t('main.radioOn')" :off-text="$t('main.radioOff')"
                     v-model="formState.formData.state"></el-switch>
        </el-form-item>
        <el-form-item :label="$t('contents.tagOrKey')" prop="tags">
          <el-select size="small" v-model="formState.formData.tags" multiple filterable allow-create
                     :placeholder="$t('validate.selectNull', {label: this.$t('contents.tags')})">
            <el-option v-for="item in contentTagList.docs" :key="item._id" :label="item.name" :value="item._id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item :label="$t('contents.sImg')" prop="sImg">
          <el-upload class="avatar-uploader" action="/system/upload?type=images" :show-file-list="false"
                     :on-success="handleAvatarSuccess" :before-upload="beforeAvatarUpload">
            <img v-if="formState.formData.sImg" :src="formState.formData.sImg" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
      </div>
      <el-form-item :label="$t('contents.discription')" prop="discription">
        <el-input size="small" type="textarea" v-model="formState.formData.discription"></el-input>
      </el-form-item>
      <el-form-item :label="$t('contents.comments')" prop="comments">
        <Ueditor @ready="editorReady" ref="ueditor"></Ueditor>
      </el-form-item>
      <el-form-item :label="$t('contents.version')" prop="versions">
        <el-table
                :data="formState.formData.versions"
                style="width: 100%">
          <el-table-column
                  prop="version"
                  label="版本号"
                  width="180">
          </el-table-column>
          <el-table-column
                  prop="lang"
                  label="语言"
                  width="180">
          </el-table-column>
          <el-table-column
                  prop="updateTime"
                  label="更新时间">
          </el-table-column>
          <el-table-column
                  prop="fileSize"
                  label="文件大小">
          </el-table-column>
          <el-table-column
                  prop="downloadUrl"
                  label="下载地址">
          </el-table-column>
        </el-table>
        <el-form :model="addVerData" label-width="80px">
          <el-form-item label="版本号">
            <el-input v-model="addVerData.version"></el-input>
          </el-form-item>
          <el-form-item label="语言">
            <el-select v-model="addVerData.lang" placeholder="lang">
              <el-option
                      v-for="item in langs"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="更新时间">
            <el-date-picker
                    v-model="addVerData.updateTime"
                    type="date"
                    placeholder="选择日期">
            </el-date-picker>
          </el-form-item>
          <el-form-item label="文件大小">
            <el-input v-model="addVerData.fileSize"></el-input>
          </el-form-item>
          <el-form-item label="下载地址">
            <el-input v-model="addVerData.downloadUrl"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="addVersion">添加</el-button>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item class="dr-submitContent">
        <el-button size="medium" type="primary" @click="submitForm('ruleForm')">{{formState.edit ?
          $t("main.form_btnText_update") : $t("main.form_btnText_save")}}
        </el-button>
        <el-button size="medium" @click="backToList">{{$t("main.back")}}</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<style lang="scss">
  .dr-contentForm {
    margin: 15px 0;
    width: 80%;
    padding-bottom: 50px;

    .post-rate {
      .el-rate {
        margin-top: 10px;
      }
    }

    .dr-submitContent {
      position: fixed;
      z-index: 9999999;
      padding: 10px 30px;
      text-align: right;
      right: 0;
      bottom: 0;
      background: none;
      margin-bottom: 0;
    }

    .avatar-uploader .el-upload {
      border: 1px dashed #d9d9d9;
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hidden;
    }

    .avatar-uploader .el-upload:hover {
      border-color: #409eff;
    }

    .avatar-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 200px;
      height: 150px;
      line-height: 150px;
      text-align: center;
    }

    .avatar {
      width: 200px;
      height: 158px;
      display: block;
    }
  }
</style>

<script>
  import services from "../../store/services.js"
  import Ueditor from "../common/Ueditor.vue"

  import _ from "lodash"
  import {mapGetters, mapActions} from "vuex"

  export default {
    props: {
      groups: Array
    },
    data() {
      return {
        addVerData: {
          version: "",
          lang: "",
          updateTime: "",
          fileSize: "",
          downloadUrl: ""

        },
        langs: [{
          value: "zh",
          label: "中文"
        }, {
          value: "en",
          label: "英文"
        }],
        content: "",
        isflash: false,
        config: {
          initialFrameWidth: null,
          initialFrameHeight: 320
        },
        imageUrl: "",
        flashPostConfig: {
          panneState: true
        },
        categoryProps: {
          value: "_id",
          label: "name",
          children: "children"
        },
        currentType: "1",
        rules: {
          title: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.title")
              }),
              trigger: "blur"
            },
            {
              min: 2,
              max: 50,
              message: this.$t("validate.rangelength", {min: 2, max: 50}),
              trigger: "blur"
            }
          ],
          stitle: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.stitle")
              }),
              trigger: "blur"
            },
            {
              min: 2,
              max: 50,
              message: this.$t("validate.rangelength", {min: 2, max: 50}),
              trigger: "blur"
            }
          ],
          categories: [
            {
              validator: (rule, value, callback) => {
                if (_.isEmpty(value)) {
                  callback(
                      new Error(
                          this.$t("validate.selectNull", {
                            label: this.$t("contents.categories")
                          })
                      )
                  )
                } else {
                  callback()
                }
              },
              trigger: "blur"
            }
          ],
          tags: [
            {
              validator: (rule, value, callback) => {
                if (_.isEmpty(value)) {
                  callback(
                      new Error(
                          this.$t("validate.selectNull", {
                            label: this.$t("contents.tags")
                          })
                      )
                  )
                } else {
                  callback()
                }
              },
              trigger: "change"
            }
          ],
          discription: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.discription")
              }),
              trigger: "blur"
            },
            {
              min: 5,
              max: 300,
              message: this.$t("validate.rangelength", {min: 5, max: 100}),
              trigger: "blur"
            }
          ],
          comments: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.comments")
              }),
              trigger: "blur"
            },
            {
              min: 5,
              message: this.$t("validate.rangelength", {min: 5, max: 10000}),
              trigger: "blur"
            }
          ]
        },
        flashRules: {
          categories: [
            {
              validator: (rule, value, callback) => {
                if (_.isEmpty(value)) {
                  callback(
                      new Error(
                          this.$t("validate.selectNull", {
                            label: this.$t("contents.categories")
                          })
                      )
                  )
                } else {
                  callback()
                }
              },
              trigger: "blur"
            }
          ],
          discription: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.flashComments")
              }),
              trigger: "blur"
            },
            {
              min: 5,
              max: 300,
              message: this.$t("validate.rangelength", {min: 5, max: 300}),
              trigger: "blur"
            }
          ],
          comments: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.comments")
              }),
              trigger: "blur"
            },
            {
              min: 5,
              message: this.$t("validate.rangelength", {min: 5, max: 10000}),
              trigger: "blur"
            }
          ]
        },
        twiterRules: {
          twiterAuthor: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.twiterAuthor")
              }),
              trigger: "blur"
            },
            {
              min: 2,
              max: 100,
              message: this.$t("validate.rangelength", {min: 2, max: 100}),
              trigger: "blur"
            }
          ],
          categories: [
            {
              validator: (rule, value, callback) => {
                if (_.isEmpty(value)) {
                  callback(
                      new Error(
                          this.$t("validate.selectNull", {
                            label: this.$t("contents.categories")
                          })
                      )
                  )
                } else {
                  callback()
                }
              },
              trigger: "blur"
            }
          ],
          translate: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.translate")
              }),
              trigger: "blur"
            },
            {
              min: 5,
              max: 300,
              message: this.$t("validate.rangelength", {min: 5, max: 300}),
              trigger: "blur"
            }
          ],
          comments: [
            {
              required: true,
              message: this.$t("validate.inputNull", {
                label: this.$t("contents.comments")
              }),
              trigger: "blur"
            },
            {
              min: 5,
              message: this.$t("validate.rangelength", {min: 5, max: 10000}),
              trigger: "blur"
            }
          ]
        }
      }
    },
    components: {
      Ueditor
    },
    methods: {
      addVersion() {
        Date.prototype.Format = function (fmt) { //author: meizz
          var o = {
            "M+": this.getMonth() + 1, //月份
            "d+": this.getDate(), //日
            "h+": this.getHours(), //小时
            "m+": this.getMinutes(), //分
            "s+": this.getSeconds(), //秒
            "q+": Math.floor((this.getMonth() + 3) / 3), //季度
            "S": this.getMilliseconds() //毫秒
          }
          if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length))
          for (var k in o)
            if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)))
          return fmt
        }
        console.log(this.addVerData)
        if (this.addVerData.updateTime != "") {
          this.formState.formData.versions.unshift({
            "version": this.addVerData.version,
            "lang": this.addVerData.lang,
            "updateTime": new Date(this.addVerData.updateTime.valueOf()).Format("yyyy-MM-dd"),
            "fileSize": this.addVerData.fileSize,
            "downloadUrl": this.addVerData.downloadUrl
          })
        }
        this.addVerData.version = ""
        this.addVerData.lang = ""
        this.addVerData.updateTime = ""
        this.addVerData.fileSize = ""
        this.addVerData.downloadUrl = ""
      },
      checkFlashPost(currentType) {
        this.$store.dispatch("showContentForm", {
          edit: this.formState.edit,
          formData: Object.assign({}, this.formState.formData, {
            type: currentType ? "2" : "1"
          })
        })
      },
      inputEditor(value) {
        this.$store.dispatch("showContentForm", {
          edit: this.formState.edit,
          formData: Object.assign({}, this.formState.formData, {
            markDownComments: value
          })
        })
      },
      changeEditor(value) {
        console.log(value)
      },
      getLocalContents() {
        let localContent = localStorage.getItem("addContent") || ""
        if (localContent) {
          return JSON.parse(localContent)
        } else {
          return {}
        }
      },
      editorReady(instance) {
        if (this.formState.edit) {
          instance.setContent(this.formState.formData.comments)
        } else {
          instance.setContent("")
        }
        instance.addListener("contentChange", () => {
          this.content = instance.getContent()
          this.$store.dispatch("showContentForm", {
            edit: this.formState.edit,
            formData: Object.assign({}, this.formState.formData, {
              comments: this.content
            })
          })
        })
      },

      handleAvatarSuccess(res, file) {
        let imageUrl = res
        this.$store.dispatch("showContentForm", {
          edit: this.formState.edit,
          formData: Object.assign({}, this.formState.formData, {
            sImg: imageUrl
          })
        })
      },
      beforeAvatarUpload(file) {
        const isJPG = file.type === "image/jpeg"
        const isPNG = file.type === "image/png"
        const isGIF = file.type === "image/gif"
        const isLt2M = file.size / 1024 / 1024 < 2
        if (!isJPG && !isPNG && !isGIF) {
          this.$message.error(this.$t("validate.limitUploadImgType"))
        }
        if (!isLt2M) {
          this.$message.error(
              this.$t("validate.limitUploadImgSize", {size: 2})
          )
        }
        return (isJPG || isPNG || isGIF) && isLt2M
      },
      handleChangeCategory(value) {
        console.log(value)
      },
      backToList() {
        this.$router.push("/content")
      },
      submitForm(formName, type = "") {
        this.$refs[formName].validate(valid => {
          if (valid) {
            let params = this.formState.formData
            // 更新
            if (this.formState.edit) {
              services.updateContent(params).then(result => {
                if (result.data.status === 200) {
                  this.$router.push("/content")
                  this.$message({
                    message: this.$t("main.updateSuccess"),
                    type: "success"
                  })
                } else {
                  this.$message.error(result.data.message)
                }
              })
            } else {
              // 新增
              services.addContent(params).then(result => {
                if (result.data.status === 200) {
                  this.$router.push("/content")
                  this.$message({
                    message: this.$t("main.addSuccess"),
                    type: "success"
                  })
                } else {
                  this.$message.error(result.data.message)
                }
              })
            }
          } else {
            console.log("error submit!!")
            return false
          }
        })
      }
    },
    computed: {
      ...mapGetters(["contentCategoryList", "contentTagList"]),
      formState() {
        return this.$store.getters.contentFormState
      }
    },
    mounted() {
      // 针对手动页面刷新
      let _this = this
      if (this.$route.params.id) {
        services.getOneContent(this.$route.params).then(result => {
          if (result.data.status === 200) {
            if (result.data.data.doc) {
              let contentObj = result.data.data.doc,
                  categoryIdArr = []
              contentObj.categories.map((item, index) => {
                categoryIdArr.push(item._id)
              })
              contentObj.categories = categoryIdArr
              this.$store.dispatch("showContentForm", {
                edit: true,
                formData: contentObj
              })
            } else {
              this.$message({
                message: this.$t("validate.error_params"),
                type: "warning",
                onClose: () => {
                  this.$router.push("/content")
                }
              })
            }
          } else {
            this.$message.error(result.data.message)
          }
        })
      } else {
        let localContent = this.getLocalContents()
        if (!_.isEmpty(localContent)) {
          this.$confirm(
              this.$t("main.askForReInputContent"),
              this.$t("main.scr_modal_title"),
              {
                confirmButtonText: this.$t("main.confirmBtnText"),
                cancelButtonText: this.$t("main.cancelBtnText"),
                type: "warning"
              }
          )
              .then(() => {
                let currentComments = localContent.comments || ""
                _this.$refs.ueditor.setContent(currentComments)
                // 清除缓存
                localStorage.removeItem(this.$route.path.split("/")[1])
                this.$store.dispatch("showContentForm", {
                  edit: false,
                  formData: localContent
                })
              })
              .catch(() => {
                localStorage.removeItem(this.$route.path.split("/")[1])
                this.$message({
                  type: "info",
                  message: this.$t("main.scr_modal_del_error_info")
                })
              })
        }
      }
      this.$store.dispatch("getContentCategoryList")
      this.$store.dispatch("getContentTagList", {
        pageSize: 200
      })
    }
  }
</script>
