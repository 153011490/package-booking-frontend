<template>
 <div align='center'>
    <a-form
    :form="form"
    @submit="preedSubmit"
  >
 <a-form-item
      label="包裹单号:"
      :label-col="{ span: 11 }"
      :wrapper-col="{ span: 3 }"
    >
      <a-input
        v-decorator="[
          'id',
          {rules: [{ required: true, message: 'Please input your id!' }]}
        ]"
      />
    </a-form-item>
    <a-form-item
      label="预约时间:"
      :label-col="{ span: 11 }"
      :wrapper-col="{ span: 2 }"
    >
      <a-date-picker
        v-decorator="['date-time-picker', config]"
        show-time
        format="YYYY-MM-DD HH:mm:ss"
      />
    </a-form-item>
      <a-form-item>
      <a-button
        type="primary"
        html-type="submit"
      >
        预约
      </a-button>
      <a-button
        type="primary"
        html-type="submit"
      >
        取消
      </a-button>
    </a-form-item>
  </a-form>
 </div>
</template>
<script>
export default {
  data () {
    return {
      config: {
        rules: [{ type: 'object', required: true, message: 'Please select time!' }],
      },
    };
  },
  beforeCreate () {
    this.form = this.$form.createForm(this);
  },
  methods:{
     preedSubmit(e) {
      e.preventDefault();
      this.form.validateFields((err, fieldsValue) => {
        if (err) {
          return;
        }
        const values = {
          ...fieldsValue,
          'date-time-picker': fieldsValue['date-time-picker'].format('YYYY-MM-DD HH:mm:ss'),
        };
         if (fieldsValue['date-time-picker'].format('HHmmss')<90000||fieldsValue['date-time-picker'].format('HHmmss')>200000) {
          alert("【预约失败】取件时间：9点——20点");
          return;
        }
        this.axios
              .put(
                "http://localhost:5555/packages/ids/"+values['id']+"?time="+values['date-time-picker']
              )
              .then(response => {
                response.data==='success'?alert("预约成功"):alert("预约失败!请检查包裹单号是否有误！");
          });
      });
    }
  }
};
</script>