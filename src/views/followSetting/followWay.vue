<style lang="less">
.followWay{
  &_main{
    	&_search {
			box-sizing: border-box;
			margin-bottom: 10px;
			.ivu-col {
				display: flex;
				>span {
					background-color: #dadada;
					text-align: center;
					line-height: 32px;
					display: block;
					width: 80px;
					flex-shrink: 0;
					border-top-left-radius: 4px;
					border-bottom-left-radius: 4px;
				}
				.ivu-input {
					border-top-left-radius: 0;
					border-bottom-left-radius: 0;
				}
				.ivu-select {
					flex-grow: 1;
					flex-shrink: 1;
				}
				.ivu-select-selection {
					border-top-left-radius: 0;
					border-bottom-left-radius: 0;
				}
			}
		}
  }
}

</style>
<template>
  <Row class="followWay">
    <!-- 搜索栏 -->
    <Col span="24" class="">
    <Row class="followWay_main_search" :gutter="15">
				<Col span="6">
				<span>
					方案名称
				</span>
        <Input type="text" v-model="waySearch.name" placeholder="请输入方案名称"></Input>
				</Col>
				<Col span="6">
				<span>
					科室类别
				</span>
			<Select :filterable="true" v-model="waySearch.departmentId" clearable @on-change="handleDeparment">
          <Option v-for="(option, index) in deparmentSelect" :value="option.value" :key="index">{{option.label}}</Option>
        </Select>
				</Col>
				<Col span="6">
				<span>
					疾病类型
				</span>
			 <Select :label="modelTre" v-model="waySearch.diseaseId" filterable remote not-found-text="" :remote-method="remoteMethod2" :loading="loading2" placeholder="请输入疾病类型" clearable>
          <Option v-for="(option, index) in options2" :value="option.value" :key="index">{{option.label}}</Option>
        </Select>
				</Col>
				<Col span="6">
				<Button style="margin-right:10px" type="primary" @click="handleSearch()">查询</Button>
        <Button type="info" v-if="!menuShow(this.AM.FollowSetting.addDep)" @click="addBtn()">增加随访方案</Button>
				</Col>
			</Row>
    </Col>
    <!-- 表格 -->
    <Col span="24" class="patientList">
    <Table border :columns="columns7" :data="pardata" :loading="createLoading"></Table>
    </Col>
    <!-- 分页 -->
    <Col span="24" class="pages">
    <Page style="margin-top:10px;float:right" :current="page" :total="pageTotal" @on-change="currentPage" show-elevator show-total></Page>
    </Col>
    <!-- 详情模态框 -->
    <Modal v-model="patientText" name="添加问题 / 编辑问题" @on-ok="ok" @on-cancel="cancel" width="650" class-name="patientInfo">
      <Form :model="formItem" :label-width="100" ref="proRuleModel" :rules="proRuleModel">
        <input type="hidden" v-model="formItem.id" placeholder="id">
        <FormItem label="问题标题" prop="name">
          <Input v-model="formItem.name" placeholder="请输入问题标题"></Input>
        </FormItem>
        <FormItem label="问题内容" prop="content">
          <Input v-model="formItem.content" placeholder="请输入随访问题内容"></Input>
        </FormItem>
        <FormItem label="采集指标">
          <RadioGroup v-model="formItem.isTarget" @on-change="targetChange">
            <Radio label="0">是</Radio>
            <Radio label="1">否</Radio>
          </RadioGroup>
        </FormItem>
        <FormItem label="关联指标" prop="targetName1" v-if="targetShow">
          <!-- <Input v-model="formItem.targetName" placeholder="根据首字母进行搜索" @on-keyup="keyupzb($event)"></Input> -->
          <Select v-model="formItem.targetName1" filterable remote :remote-method="remoteMethod1" :loading="loading1" clearable :label-in-value=true @on-change="targetRadio" not-found-text="">
            <Option v-for="(option, index) in options1" :value="option.value" :key="index">{{option.label}}</Option>
          </Select>
        </FormItem>
        <FormItem label="" prop="" label="指标类型" v-if="targetShow">
          <Tag color="blue" v-if="tagShow">{{targetTag}}</Tag>
        </FormItem>
        <FormItem label="疾病类型" prop="diseaseName">
          <Select v-model="formItem.diseaseName" filterable remote :remote-method="remoteMethod2" :loading="loading2" clearable style="width: 70%;float: left;margin-right:20px;" @on-change="selectChange" not-found-text="" :label-in-value=true placeholder="搜索疾病类型添加至疾病标签">
            <Option v-for="(option, index) in options2" :value="option.value" :key="index">{{option.label}}</Option>
          </Select>
          <Button type="primary" @click="addTag" ref="addTagbtn">添加</Button>
        </FormItem>
        <FormItem label="" prop="" label="疾病标签">
          <tag v-for="item in tagCount" color="blue" :key="item" :name="item" closable @on-close="tagClose">{{item}}</tag>
        </FormItem>
        <FormItem label="纯放音">
          <RadioGroup v-model="formItem.playWavOnly" @on-change="radioChange">
            <Radio label="1">是</Radio>
            <Radio label="0">否</Radio>
          </RadioGroup>
        </FormItem>
        <FormItem>
          <Button type="primary" @click="addModel('proRuleModel')">保存</Button>
        </FormItem>
      </Form>
    </Modal>

  </Row>
</template>

<script>
import { API } from '@/services';
import {mapGetters, mapActions} from 'vuex';

export default {
  data() {
    return {
      createLoading:true,           //默认显示为true
      waySearch: {//搜索框
        name: '',
        diseaseId: '',
        departmentId: '',
      },
        modelTre:"", //记录疾病类型值
      page:1,
      deparmentSelect: [],//科室列表
      columns7: [//表格栏
        {
          title: 'id',
          key: 'id',
          align: 'center',

        },
        {
          title: '方案名称',
          key: 'name',
          align: 'center',
        },
        {
          title: '科室类别',
          key: 'departmentName',
          align: 'center',
        },
        {
          title: '疾病类型',
          key: 'diseaseName',
          align: 'center',
        },
        {
          title: '操作',
          key: 'action',
          width: 150,
          align: 'center',
          render: (h, params) => {
            return h('div', [
              h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                style: {
                  marginRight: '5px'
                },
                'class':{
									menuHide:this.menuShow(this.AM.FollowSetting.editDep)
								},
                on: {
                  click: () => {
                    this.$router.push({ path: `/followSetting/way/way/${params.row.id}` });
                      //保存数据  当再次返回的时候进行重新赋值
                      this.saveFollowWay({
                              "page":this.page,       //页码
                      });
                  }
                }
              }, '编辑'),

              h('Button', {
                props: {
                  type: 'warning',
                  size: 'small'
                },
                style: {
                  marginRight: '5px'
                },
                'class':{
									menuHide:this.menuShow(this.AM.FollowSetting.delDep)
								},
                on: {
                  click: () => {
                    this.$Modal.confirm({
                      title: '删除',
                      content: '<p>确定要删除吗?</p>',
                      onOk: () => {
                        this.deleteRow(params.row.id)
                      },
                      onCancel: () => {
                        // this.$Message.info('Clicked cancel');
                      }
                    });
                  }
                }
              }, '删除'),
            ]);
          }
        }],
      pardata: [],//表格data
      pageTotal: 0,//数据总计
      patientText: false,//编辑模态框
      diseaseIdInp: '',//疾病id隐藏域
      formItem: {
        id: '',
        name: '',
        content: '',
        targetId: '',
        targetName1: '',
        diseaseName: '',
        diseaseId: '',
        model10: [],
        playWavOnly: '1',
        isTarget: '0'//是否采集指标
      },
      proRuleModel: {//模态框校验
        name: [
          { required: true, message: '问题标题问题标题不能为空', trigger: 'blur' },
          { type: 'string', max: 30, message: '最大长度不超过30', trigger: 'blur' }
        ],
        content: [
          { required: true, message: '问题内容不能为空', trigger: 'blur' }
        ],
        // targetName1: [
        //   { required: true, message: '关联指标不能为空', trigger: 'blur' }
        // ],
        password: [
          { required: true, message: 'Please fill in the password.', trigger: 'blur' },
          { type: 'string', min: 6, message: 'The password length cannot be less than 6 bits', trigger: 'blur' }
        ]
      },
      tagCount: [],
      tagCount2: [],
      tagCountId: [],
      editRules: {
      },
      zjmdata: [],//助记码array
      loading1: false,
      options1: [],
      listdata: [],
      loading2: false,
      options2: [],
      listdata2: [],
      diseasedata: [],
      selectLabel: '',
      selectValue: '',
      targetShow: true,//判断是否疾病标签是否展示
      targetTag: '',//指标标签
      tagShow: false,//标签是否展示,
      targetSelectId: '',//选中指标的id
    }
  },
    computed:{
        ...mapGetters([
            'authorToken'
        ]),
    },
  mounted() {
      /**
       * 获取页面 page 为Number   进入页面进行再一次赋值
       * @type {number}
       */
        this.page =Number(this.authorToken.followWayRecord.page) ;     //页码
        this.waySearch.name = this.authorToken.followWayRecord.name||'';            //方案名称
        this.waySearch.departmentId = this.authorToken.followWayRecord.departmentId||'';  //类别
        this.waySearch.diseaseId =this.authorToken.followWayRecord.diseaseId!==''?this.authorToken.followWayRecord.diseaseId:'';
          this.options2 = this.authorToken.followWayRecord.diseaseList||[];
            if(this.options2) {
                for (const item of this.options2) {
                    if (this.authorToken.followWayRecord.diseaseId == item.value) {
                        this.modelTre = item.label
                    }
                }
            }
        this.list(this.page)
        this.departmentList();
  },

  methods: {
      ...mapActions([
          'saveFollowWay'
      ]),
    /*
    *获取prolist列表数据
    */
    list(pager) {
      API.followWay.list({
        pager: pager,
        limit: '10',
        name: this.waySearch.name,
        departmentId: this.waySearch.departmentId,
        diseaseId: this.waySearch.diseaseId,
      }).then((res) => {
        if (res.code == 0) {
          this.pardata = res.data
          this.pageTotal = res.total
          this.createLoading = false;
        } else {
          console.log(res.code)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    /*
    *获取科室列表
    */
    departmentList() {
      API.followWay.departmentList({
        'pager': 1,
        'limit': '10000',
        'name': this.waySearch.name,
      }).then((res) => {
        if (res.code == 0) {
          class Deparment {
            constructor(item) {
              this.value = item.value;
              this.label = item.label;
            }
          }
          res.data.forEach((item) => {
            this.deparmentSelect.push(new Deparment({
              value: item.id,
              label: item.name
            }))
          })
          this.deparmentSelect.splice(0,0,{
            value:"",
            label:"全部"
          })
        } else {
          console.log(res.code)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    /*
    *获取选中的部门id
    */
    handleDeparment(value) {
      console.log(value)
      console.log(this.waySearch.departmentId)
    },
    /*
    *获取分页列表数据
    */
    currentPage: function(page) {
      this.page=page;
      API.followWay.list({
        'pager': page,
        'limit': '10',
        'name': this.waySearch.name,
        'diseaseId': this.waySearch.diseaseId,
        'departmentId': this.waySearch.departmentId
      }).then((res) => {
        if (res.code == 0) {
          this.pardata = res.data
          this.pageTotal = res.total
        } else {
          console.log(res.message)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    /*
    *获取选中的指标value
    */
    targetRadio(value) {
      console.log(value.label)
      API.followWay.list({
        pager: 1,
        limit: '10',
        name: value.label,
      }).then((res) => {
        if (res.code == 0) {
          console.log(res)
          let otypeName
          if (res.data[0].otype == '01') {
            otypeName = '症状'
          } else if (res.data[0].otype == '02') {
            otypeName = '体征'
          } else if (res.data[0].otype == '03') {
            otypeName = '生活方式指导'
          } else if (res.data[0].otype == '04') {
            otypeName = '辅助检查'
          } else if (res.data[0].otype == '05') {
            otypeName = '用药反馈'
          } else if (res.data[0].otype == '06') {
            otypeName = '转诊情况'
          } else if (res.data[0].otype == '07') {
            otypeName = '通用'
          }
          if (this.formItem.targetName1 != '') {
            this.tagShow = true
            this.targetTag = otypeName
            this.targetSelectId = res.data[0].id
          } else {
            this.tagShow = false
            this.targetTag = ''
          }
          // console.log('this.targetTag='+this.targetTag)
        } else {
          console.log(res.message)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    /*
    *查询功能
    */
    handleSearch() {
      this.page=1;
      API.followWay.list({
        'pager': 1,
        'limit': '10',
        'name': this.waySearch.name,
        'diseaseId': this.waySearch.diseaseId,
        'departmentId': this.waySearch.departmentId
      }).then((res) => {
        if (res.code == 0) {
          this.pardata = res.data
          this.pageTotal = res.total

            //保存数据  当再次返回的时候进行重新赋值
            this.saveFollowWay({
                "name":this.waySearch.name,       //页码
                'diseaseId': this.waySearch.diseaseId,
                'departmentId': this.waySearch.departmentId,
                "diseaseList":this.options2
            });

        } else {
          console.log(res.message)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    /*
    *添加指标
    */
    addBtn(name) {
      this.$router.push({ path: `/followSetting/way/way/new` });
    },
    /*
    *获取选中的疾病标签列value
    */
    selectChange(value) {
      console.log(value.label)
      this.selectLabel = value.label
      this.selectValue = value.value
    },
    /*
    *确定添加
    */
    addModel(name) {
      let jbnam = this.tagCount.join(',')
      let addPram = {
        "id": this.formItem.id,
        "name": this.formItem.name,
        "isTarget": this.formItem.isTarget,
        "content": this.formItem.content,
        "targetId": this.targetSelectId,
        "diseaseId": this.tagCountId.join(','),
        "playWavOnly": this.formItem.playWavOnly,
        "status": 0,
      }
      this.$refs[name].validate((valid) => {
        if (valid) {
          this.$Message.success('Success!');
          API.followWay.addList(addPram).then((res) => {
            if (res.code == 0) {
              console.log(res.message)
              this.formItem.name = ''
              this.formItem.select2 = ''
              this.formItem.select = ''
              this.formItem.radio = 'string'
              this.formItem.textarea = ''
              this.patientText = false;
              this.tagCount = []//清空疾病标签
              this.list(1)
              console.log(res)
            } else {
              alert('res.message=' + res.message)
            }
          }).catch((error) => {
            console.log(error)
          })
        } else {
          this.$Message.error('Fail!');
        }
      })
    },
    /*
    *删除
    */
    deleteRow(id) {
      API.followWay.deleteList({
        id: id
      }).then((res) => {
        if (res.code == 0) {
          console.log(res.message)
          this.list(1)
          this.$Message.success({
            content: '删除成功',
            top: 500
          });
        } else {
          alert(res.message)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    /*
    *添加标签
    */
    addTag() {
      let flag = 0;
      this.tagCount.forEach((item) => {
        if (this.selectLabel == item || this.selectLabel == '') {
          flag++;
          alert('您添加的为空或者重复添加')
        }
      })
      if (flag > 0) {
        this.selectLabel = ''
        return false;
      }
      this.tagCount.push(this.selectLabel)
      this.tagCount2.push(this.selectValue)
      this.tagCountId.push(this.selectValue)
      this.selectLabel = ''
      this.selectValue = ''
      this.formItem.diseaseName = ''
    },
    /*
    *删除标签
    */
    tagClose(event, name) {
      console.log(event)
      console.log(name)
      const index = this.tagCount.indexOf(name);
      this.tagCount.splice(index, 1);
      this.tagCountId.splice(index, 1);
    },
    /*
    *监听是否纯放音的单选
    */
    radioChange(value) {
      console.log('是否纯放音=' + value)
    },
    /*
    *监听是否采集指标
    */
    targetChange(value) {
      console.log('是否采集指标=' + value)
      if (value == '0') {
        this.targetShow = true
      } else {
        this.targetShow = false
      }
    },
    //详情模态框
    show(index) {
      this.$Modal.info({
        name: 'User Info',
        content: `Name：${this.pardata[index].name}<br>Age：${this.pardata[index].age}<br>Address：${this.pardata[index].address}`
      })
    },
    remove(index) {
      this.pardata.splice(index, 1);
    },
    //详情关闭确认点击事件
    ok() {
      this.$Message.info('您打开了弹框');
    },
    cancel() {
      this.$Message.info('您关闭了弹框');
      this.tagCount = []//清空疾病标签
    },
    //编辑模态框提交按钮
    handleEdit(name) {
      this.$refs[name].validate((valid) => {
        if (valid) {
          this.$Message.success('Success!');
        } else {
          this.$Message.error('Fail!');
        }
      })
    },
    /*
    *知名名称，根据助记码搜索指标类型
    */
    keyupzb: function(ev) {
      console.log(this.formItem.targetName)
    },
    /*
    *指标类型--远程搜索
    */
    remoteMethod1(query) {
      API.follSetting.list({
        'pager': '1',
        'limit': '1000',
        'zjm': query
      }).then((res) => {
        if (res.code == 0) {
          // console.log(res.data)
          class Point {
            constructor(item) {
              this.value = item.value;
              this.label = item.label;
            }
          }
          let parr = []
          let more = res.data
          more.forEach((item) => {
            parr.push(new Point({
              value: item.id,
              label: item.name
            }))
          })

          this.options1 = parr
          if (query !== '') {
            this.options1 = parr

          } else {
            this.options1 = [];
          }
        } else {
          console.log(res)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    /*
    *疾病类型--远程搜索
    */
    remoteMethod2(query) {
      if(query.trim()==""){
        return false;
      }
      API.followProblems.disease({
        'zjm': query
      }).then((res) => {

        console.log(res)
        if (res.code == 0) {
          class Point {
            constructor(item) {
              this.value = item.value;
              this.label = item.label;
            }
          }
          let parr2 = []
          let more2 = res.data
          more2.forEach((item) => {
            parr2.push(new Point({
              value: item.id,
              label: item.value
            }))
          })

          this.options2 = parr2
          if (query !== '') {
            this.options2 = parr2

          } else {
            this.options2 = [];
            this.waySearch.diseaseName = ''
          }


        } else {
          console.log(res)
        }
      }).catch((error) => {
        console.log(error)
      })
    },
  }
}
</script>

