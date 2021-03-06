<template>
  <el-card>
    <div slot="header">
      <span>{{title}}</span>
    </div>
    <div v-for="(item, index) in formList" :key="index">
      <el-form :model="item" :rules="rules" ref="form" label-width="120px" class="form">
        <el-form-item>
          <el-button @click="deleteForm(index)" type="danger" icon="el-icon-delete" class="fr"/>
          <el-button v-if="!item.editable" type='primary' icon="el-icon-edit-outline" @click="handleEditable(index)" class="fr"/>
        </el-form-item>
        <slot :item="item" ></slot>
        <el-form-item v-if="item.editable">
          <el-button @click="saveForm(index)" type='primary'>保存</el-button>
          <el-button @click="resetForm(index)">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <el-button @click="add" type='success' icon="el-icon-circle-plus-outline" class="fr add-button">添加</el-button>
  </el-card>
</template>

<script>
import qs from 'querystring'
import utils from '@/components/utils'
import clone from 'clone'

export default {
  data() {
    return {
      formList: []
    }
  },
  props: ['resumeId', 'subList', 'subType', 'rules', 'formBean', 'title'],
  watch: {
    subList(value) {
      if (value) {
        value.forEach(item => {
          this.formList.push(utils.fill(this.createForm(false), item))
        })
      }
    }
  },
  methods: {
    createForm(editable) {
      let form = clone(this.formBean)
      form.id = null
      form.resumeId = this.resumeId
      form.editable = editable
      return form
    },
    add() {
      this.formList.push(this.createForm(true))
    },
    resetForm(index) {
      this.$refs['form'][index].resetFields()
    },
    saveForm(index) {
      this.$refs['form'][index].validate(valid => {
        if (!valid) {
          return
        }
        let form = this.formList[index]
        this.$http
          .post('/resume2/save/' + this.subType, qs.stringify(form))
          .then(response => {
            if (response.data.success) {
              form.editable = false
              form.id = response.data.data
            } else {
              this.$message({
                message: response.data.message,
                type: 'warning'
              })
            }
          })
      })
    },
    handleEditable(index) {
      let form = this.formList[index]
      form.editable = true
    },
    deleteForm(index) {
      let form = this.formList[index]
      if (form.id) {
        this.$confirm('是否确认删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http
            .post(
              '/resume2/delete/' + this.subType,
              qs.stringify({ id: form.id, resumeId: form.resumeId })
            )
            .then(response => {
              if (response.data.success) {
                this.formList.splice(index, 1)
              } else {
                this.$message({
                  message: response.data.message,
                  type: 'warning'
                })
              }
            })
        })
      } else {
        this.formList.splice(index, 1)
      }
    }
  }
}
</script>

<style scoped>
.form {
  border-bottom: 1px dashed #ebeef5;
  margin-bottom: 20px;
}
.add-button {
  margin-bottom: 20px;
}
</style>


