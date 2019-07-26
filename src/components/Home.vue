<template>
  <div>
    <div class="table-operations">
      <div align="right">
        <a-button @click="findAll">ALL</a-button>
      <a-button @click="findByNotPreed">未预约</a-button>
      <a-button @click="findByNotFetched">未取件</a-button>
      <a-button @click="findByFetched">已取件</a-button>
      <a-button type="primary" @click="showModal">+添加</a-button>
      </div>
      <a-modal title="包裹入库" :visible="visible" @ok="handleOk" :confirmLoading="confirmLoading">
        <a-form :form="form" @submit="handleSubmit">
          <a-form-item label="运单号" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'packageId',
          {rules: [{ required: true, message: 'Please input your 运单号!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item label="收件人" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'name',
          {rules: [{ required: true, message: 'Please input your 收件人!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item label="电话" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'tel',
          {rules: [{ required: true, message: 'Please input your tel!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item label="重量" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
            <a-input
              v-decorator="[
          'weight',
          {rules: [{ required: true, message: 'Please input your weight!' }]}
        ]"
            />
          </a-form-item>
          <a-form-item :wrapper-col="{ span: 12, offset: 5 }">
            <a-button type="primary" html-type="submit">添加</a-button>
          </a-form-item>
        </a-form>
      </a-modal>
    </div>
    <a-table :columns="columns" :dataSource="data">
      <span slot="action" slot-scope="text, record">
        <a-button @click="confirms(record.key)" :v-show="data[record.key]['status']">确认收件</a-button>
      </span>
    </a-table>
  </div>
</template>
<script>
const columns = [
  {
    title: "运单号",
    dataIndex: "packageId",
    key: "packageId"
  },
  {
    title: "收件人",
    dataIndex: "name",
    key: "name"
  },
  {
    title: "电话",
    dataIndex: "tel",
    key: "tel"
  },
  {
    title: "状态",
    key: "packageStatus",
    dataIndex: "packageStatus"
  },
  {
    title: "预约时间",
    key: "preTime",
    dataIndex: "preTime"
  },
  {
    title: "操作",
    key: "action",
    scopedSlots: { customRender: "action" }
  }
];

export default {
  data() {
    return {
      data: [],
      columns,
      ModalText: "Content of the modal",
      visible: false,
      confirmLoading: false,
      formLayout: "horizontal",
      form: this.$form.createForm(this)
    };
  },
  methods: {
    showModal() {
      this.visible = true;
    },
    handleOk(e) {
      this.ModalText = "The modal will be closed after two seconds";
      this.confirmLoading = true;
      setTimeout(() => {
        this.visible = false;
        this.confirmLoading = false;
      }, 1);
    },
    handleSubmit(e) {
      e.preventDefault();
      const self = this;
      this.form.validateFields((err, values) => {
        if (!err) {
          this.form.validateFields((err, fieldsValue) => {
            console.log("Received values of form: ", fieldsValue);
            let packageId = fieldsValue["packageId"];
            let name = fieldsValue["name"];
            let weight = fieldsValue["weight"];
            let tel = fieldsValue["tel"];
            this.axios
              .post(
                "http://localhost:5555/packages",
                {
                  packageId: packageId,
                  name: name,
                  phone: tel,
                  status: 0,
                  time: ""
                },
                "application/json"
              )
              .then(response => {
                if(response.status==200){
                  alert("添加成功");
                  self.findAll();
                }else{
                  alert("添加失败");
                }
              });
          });
        }
      });
    },
    handleSelectChange(value) {
      console.log(value);
      this.form.setFieldsValue({
        note: `Hi, ${value === "male" ? "man" : "lady"}!`
      });
    },
    findAll() {
      const self = this;
      self.data = [];
      this.axios.get("http://localhost:5555/packages").then(response => {
        response.data.forEach(item => {
          if (item["status"] == 0) item["status"] = "未预约";
          else if (item["status"] == 1) item["status"] = "未取件";
          else item["status"] = "已取件";
          self.data.push({
            key: self.data.length,
            packageId: item["packageId"],
            name: item["name"],
            tel: item["phone"],
            packageStatus: item["status"],
            preTime: item["time"]
          });
        });
      });
    },
    confirms(index){
      const self = this;
      let item={packageId:self.data[index]['packageId'],name:self.data[index]['name'],phone:self.data[index]['tel'],status:'2',time:self.data[index]['preTime']};
      this.axios.put("http://localhost:5555/packages",item,"application/json");
      this.findAll();
    },
    findByNotPreed() {
      this.findByStatus(0);
    },
    findByFetched() {
      this.findByStatus(2);
    },
    findByNotFetched() {
      this.findByStatus(1);
    },
    findByStatus(status) {
      const self = this;
      self.data = [];
      this.axios
        .get("http://localhost:5555/packages/status/" + status)
        .then(response => {
          response.data.forEach(item => {
            if (item["status"] == 0) item["status"] = "未预约";
            else if (item["status"] == 1) item["status"] = "未取件";
            else item["status"] = "已取件";
            self.data.push({
              key: self.data.length,
              packageId: item["packgeId"],
              name: item["name"],
              tel: item["phone"],
              packageStatus: item["status"],
              preTime: item["time"]
            });
          });
        });
    }
  },
  created() {
    const self = this;
    self.findAll();
  }
};
</script>