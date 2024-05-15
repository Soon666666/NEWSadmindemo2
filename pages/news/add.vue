<template>
  <view class="uni-container">
    <uni-forms ref="form" :model="formData" validateTrigger="bind">
      <uni-forms-item name="category_id" label="分类">
        <uni-data-picker v-model="formData.category_id" collection="category" field="name as text, _id as value"></uni-data-picker>
      </uni-forms-item>
      <uni-forms-item name="title" label="标题" required>
        <uni-easyinput placeholder="文章标题" v-model="formData.title" trim="both"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="content" label="文章内容" required>
        <uni-easyinput type="textarea" autoHeight maxlength="10100" placeholder="文章内容" v-model="formData.content"></uni-easyinput>
      </uni-forms-item>
      <uni-forms-item name="excerpt" label="文章摘录">
        <uni-easyinput type="textarea" autoHeight maxlength="10100" placeholder="文章摘录" v-model="formData.excerpt"></uni-easyinput>
      </uni-forms-item>
     <uni-forms-item name="cover" label="封面图" required>
        <uni-file-picker file-mediatype="image" file-extname="jpg,png" return-type="object" v-model="formData.cover"></uni-file-picker>
      </uni-forms-item>
      <uni-forms-item name="publish_date" label="发表时间">
        <uni-datetime-picker return-type="timestamp" v-model="formData.publish_date"></uni-datetime-picker>
      </uni-forms-item>
      <uni-forms-item name="last_modify_date" label="最后修改时间">
        <uni-datetime-picker return-type="timestamp" v-model="formData.last_modify_date"></uni-datetime-picker>
      </uni-forms-item>
      <view class="uni-button-group">
        <button type="primary" class="uni-button" style="width: 100px;" @click="submit">提交</button>
        <navigator open-type="navigateBack" style="margin-left: 15px;">
          <button class="uni-button" style="width: 100px;">返回</button>
        </navigator>
      </view>
    </uni-forms>
  </view>
</template>

<script>
  import { validator } from '../../js_sdk/validator/news.js';

  const db = uniCloud.database();
  const dbCmd = db.command;
  const dbCollectionName = 'news';

  function getValidator(fields) {
    let result = {}
    for (let key in validator) {
      if (fields.includes(key)) {
        result[key] = validator[key]
      }
    }
    return result
  }

  

  export default {
    data() {
      let formData = {
        "category_id": "",
        "title": "",
        "content": "",
        "excerpt": "",
        "cover": null,
        "publish_date": null,
        "last_modify_date": null
      }
      return {
        formData,
        formOptions: {},
        rules: {
          ...getValidator(Object.keys(formData))
        }
      }
    },
    onReady() {
      this.$refs.form.setRules(this.rules)
    },
    methods: {
		
      
      /**
       * 验证表单并提交
       */
      submit() {
        uni.showLoading({
          mask: true
        })
        this.$refs.form.validate().then((res) => {
          return this.submitForm(res)
        }).catch(() => {
        }).finally(() => {
          uni.hideLoading()
        })
      },

      /**
       * 提交表单
       */
      submitForm(value) {
        // 使用 clientDB 提交数据
        return db.collection(dbCollectionName).add(value).then((res) => {
          uni.showToast({
            title: '新增成功'
          })
          this.getOpenerEventChannel().emit('refreshData')
          setTimeout(() => uni.navigateBack(), 500)
        }).catch((err) => {
          uni.showModal({
            content: err.message || '请求服务失败',
            showCancel: false
          })
        })
      }
	  
	  
    }
  }
</script>
