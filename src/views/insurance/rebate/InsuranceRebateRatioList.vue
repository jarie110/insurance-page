<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('返点比例')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-button @click="handleBatchSetting" type="primary" icon="form">批量设置</a-button>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal"
                     @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->


    <div>

      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>


      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt=""
               style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit/*Rebate*/(record)">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <insurance-rebate-ratio-modal ref="modalForm" @ok="modalFormOk"></insurance-rebate-ratio-modal>

    <a-modal :width="800" switchFullscreen v-model="isShowBatchSettings" title="*****提示：在设置其他返点类型前，请保存当前设置*****" on-ok="handleOk" >


      <a-dropdown>
        <a class="ant-dropdown-link" @click="getAllRebate">
          请选择返点类型
          <a-icon type="down"/>
        </a>
        <a-menu slot="overlay">
          <a-menu-item v-for="item in rebates">
            <a href="javascript:;" @click="showrebate(item)">{{item.title}}</a>
          </a-menu-item>
        </a-menu>
      </a-dropdown>

      <a-row :gutter="24" v-if="showIndex == '0'">
        <a-form-item label="商业基础险">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="使用性质" v-for="item in usages">
              <a-input :value="item.text" disabled></a-input>
              返点比例：<a-input placeholder="请输入返点比例" v-model="rebateRatio"></a-input>
              <a-col :span="24">
                <a-form-item label="录入日期">
                  <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
                  <span class="query-group-split-cust"></span>
                  <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
                </a-form-item>
              </a-col>
            </a-form-item>
          </a-col>
        </a-form-item>
      </a-row>

      <a-row :gutter="24" v-if="showIndex == '1'">
        <a-form-item label="三者责任险（根据不同使用性质）">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="使用性质" v-for="item in usages">
              <a-input :value="item.text" disabled></a-input>
              <a-col :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="三者保额档">
                  <a-input placeholder="请输入三者保额档" v-model="thirdPartyInsured"></a-input>
                </a-form-item>
              </a-col>
              <a-col :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="车损保额档">
                  <a-input placeholder="车损保额档" v-model="carDamageInsured"></a-input>
                </a-form-item>
              </a-col>
              车损险不为0返点比例：<a-input placeholder="请输入返点比例" v-model="rebateRatio"></a-input>


              <a-col :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="三者保额档">
                  <a-input placeholder="请输入三者保额档" v-model="thirdPartyInsured"></a-input>
                </a-form-item>
              </a-col>
              <a-col :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="车损保额档">
                  <a-input placeholder="等于0" disabled></a-input>
                </a-form-item>
              </a-col>
              车损险为0返点比例：<a-input placeholder="请输入返点比例" v-model="rebateRatioZero"></a-input>
              <a-col :span="24">
                <a-form-item label="录入日期">
                  <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
                  <span class="query-group-split-cust"></span>
                  <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
                </a-form-item>
              </a-col>
            </a-form-item>
          </a-col>

        </a-form-item>
      </a-row>

      <!--新车返点比-->
      <a-row :gutter="24" v-if="showIndex == '3'">
        <a-form-item label="新车返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>

          <a-col :span="24">
            <a-form-item label="录入日期">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
            </a-form-item>
          </a-col>
        </a-form-item>
      </a-row>

      <!--次新车返点比-->
      <a-row :gutter="24" v-if="showIndex == '2'">
        <a-form-item label="次新车返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="录入日期">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
            </a-form-item>
          </a-col>
        </a-form-item>
      </a-row>

      <!--竞回返点比-->
      <a-row :gutter="24" v-if="showIndex == '4'">
        <a-form-item label="竟回返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="录入日期">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
            </a-form-item>
          </a-col>
        </a-form-item>
      </a-row>

      <!--过户返点比例-->
      <a-row :gutter="24" v-if="showIndex == '5'">
        <a-form-item label="过户返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="录入日期">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
            </a-form-item>
          </a-col>
        </a-form-item>
      </a-row>

      <!--交叉返点比-->
      <a-row :gutter="24" v-if="showIndex == '6'">
        <a-form-item label="交叉返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="是否过户">
              <a-input  placeholder="未过户" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="续保标志">
              <a-input placeholder="市公司续保" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="录入日期">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
            </a-form-item>
          </a-col>
        </a-form-item>
      </a-row>

<!--跟单0返点-->
      <a-row :gutter="24" v-if="showIndex == '7'">
        <a-form-item label="跟单0返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="录入日期">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
            </a-form-item>
          </a-col>
        </a-form-item>
      </a-row>

      <a-row :gutter="24" v-if="showIndex == '8'">
        <a-form-item label="座位保">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="使用性质" v-for="item in usages">
              <a-input :value="item.text" disabled></a-input>
              返点比例：<a-input placeholder="请输入返点比例" v-model="rebateRatio"></a-input>
              <a-col :span="24">
                <a-form-item label="录入日期">
                  <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="createTime_begin"></j-date>
                  <span class="query-group-split-cust"></span>
                  <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="createTime_end"></j-date>
                </a-form-item>
              </a-col>
              <a-col  :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="司机责任险保额">
                  <a-input v-model="batchSettingsData" placeholder="司机责任险保额"></a-input>
                </a-form-item>
              </a-col>
              <a-col :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="乘客责任险保额">
                  <a-input v-model="batchSettingsData" placeholder="乘客责任险保额"></a-input>
                </a-form-item>
              </a-col>
              <a-col  :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="奖金">
                  <a-input v-model="batchSettingsData" placeholder="座位保奖金"></a-input>
                </a-form-item>
              </a-col>
            </a-form-item>
          </a-col>

        </a-form-item>
      </a-row>



      <template slot="footer">
        <a-button key="back" @click="handleCancel">
         取消
        </a-button>
        <a-button key="submit" type="primary" :loading="loading" @click="handleOk()">
         提交
        </a-button>
      </template>
    </a-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import {mixinDevice} from '@/utils/mixin'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import InsuranceRebateRatioModal from './modules/InsuranceRebateRatioModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import {getAction, postAction} from '@/api/manage'
  import {JFormContainer} from '@/components/jeecg/JFormContainer.vue'
  import { ajaxGetDictItems } from '@/api/api'
  export default {
    name: 'InsuranceRebateRatioList',
    mixins: [JeecgListMixin, mixinDevice],
    components: {
      InsuranceRebateRatioModal
    },
    data() {
      return {
        rebateType: '',
        rebateRatioZero: '',
        thirdPartyInsured: '',//三者保额档
        carDamageInsured: '' ,//车损险保额档
        rebateRatio: '',//返点比例
        createTime_begin: '',//开始时间
        createTime_end: '',//结束时间
        showIndex: '',
        usages: [],//
        rebates: [],
        // usageType: '',
        isShowBatchSettings: false,
        description: '返点比例管理页面',
        obj: [],
        //商业基础险类型数据
        BasicInsuranceRebate: {
          rebateType: this.rebateType,
          usageType: '',
          rebateRatio: '',
          createTimeBegin: this.createTime_begin,
          createTimeEnd: this.createTime_end
        },

        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '出单日期',
            align: "center",
            dataIndex: 'createTime'
          },

          {
            title: '返点类型',
            align: "center",
            dataIndex: 'rebateRatioType_dictText'
          },
          // {
          //   title: '是否过户',
          //   align: "center",
          //   dataIndex: 'isTransfer_dictText'
          // },
          // {
          //   title: '续保类型',
          //   align: "center",
          //   dataIndex: 'renewalType_dictText'
          // },
          {
            title: '使用性质',
            align: "center",
            dataIndex: 'usageType_dictText'
          },
          {
            title: '三责保额档',
            align: "center",
            dataIndex: 'thirdPartyInsured'
          },
          {
            title: '车损险保额档',
            align: "center",
            dataIndex: 'carDamageInsured'
          },
          {
            title: '司机责任险保额档',
            align: "center",
            dataIndex: 'driverLiabilityInsured'
          },
          {
            title: '乘客责任险保额档',
            align: "center",
            dataIndex: 'passengerLiability'
          },
          {
            title: '返点比例',
            align: "center",
            dataIndex: 'rebateRatio'
          },
          {
            title: '奖金',
            align: "center",
            dataIndex: 'bonus'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            fixed: "right",
            width: 147,
            scopedSlots: {customRender: 'action'}
          }
        ],
        url: {
          list: "/rebate/insuranceRebateRatio/list",
          delete: "/rebate/insuranceRebateRatio/delete",
          deleteBatch: "/rebate/insuranceRebateRatio/deleteBatch",
          exportXlsUrl: "/rebate/insuranceRebateRatio/exportXls",
          importExcelUrl: "rebate/insuranceRebateRatio/importExcel",

        },
        dictOptions: {},
        superFieldList: [],
      }
    },
    created() {
      this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      onDateChange: function (value, dateString) {
        console.log(dateString[0],dateString[1]);
        this.createTime_begin=dateString[0];
        this.createTime_end=dateString[1];
      },
      showrebate(item){
        console.log(item);
        this.showIndex = item.value
      //  查询所有车辆使用性质
        var str='insurance_usage,usage_name,usage_type';
        ajaxGetDictItems(str, null).then((res) => {
          if (res.success) {
            this.usages = res.result;
            console.log(this.usages);
            // this.options = res.result;
          }
        })
      },
      handleOk() {
        if(this.rebateType == 0){
        //  商业基础险类型
          this.usages.forEach(usageType =>{
            this.obj.push()
          })

        }

      },
      handleCancel() {
        this.isShowBatchSettings = false;
      },
      initDictConfig() {
      },
      //显示所有返点类型
      getAllRebate(e) {
        // getAction(this.url, param).then(res => {
        //   if (res.success && res.result) {
        //     for (let i of res.result) {
        //       i.value = i.key
        //       if (i.leaf == false) {
        //         i.isLeaf = false
        //       } else if (i.leaf == true) {
        //         i.isLeaf = true
        //       }
        //     }
        //     this.treeData = [...res.result]
        //   } else {
        //     console.log("树一级节点查询结果-else", res)
        //   }
        var str='rebate_ratio_type';
        ajaxGetDictItems(str, null).then((res) => {
          console.log(res,557);
          if (res.success) {
            // })
            console.log(res);
            this.rebates = res.result;
            this.rebateType = res.result.value;
            // this.options = res.result;
          }
        })
    },
    //批量设置
    handleBatchSetting() {
      this.isShowBatchSettings = true;
    },
    getSuperFieldList() {
      let fieldList = [];
      fieldList.push({type: 'date', value: 'createTime', text: '创建日期', dictCode: ''})
      fieldList.push({type: 'int', value: 'rebateRatioType', text: '返点类型', dictCode: 'rebate_ratio_type'})
      // fieldList.push({type: 'string', value: 'isTransfer', text: '是否过户', dictCode: 'is_transfer'})
      // fieldList.push({type: 'string', value: 'renewalType', text: '续保类型', dictCode: 'renewal_symbol'})
      fieldList.push({
        type: 'string',
        value: 'usageType',
        text: '使用性质',
        dictCode: 'insurance_usage,usage_name,usage_type'
      })
      fieldList.push({type: 'string', value: 'thirdPartyInsured', text: '三责保额档', dictCode: ''})
      fieldList.push({type: 'string', value: 'carDamageInsured', text: '车损险保额档', dictCode: ''})
      fieldList.push({type: 'string', value: 'driverLiabilityInsured', text: '司机责任险保额档', dictCode: ''})
      fieldList.push({type: 'string', value: 'passengerLiability', text: '乘客责任险保额档', dictCode: ''})
      fieldList.push({type: 'double', value: 'rebateRatio', text: '返点比例', dictCode: ''})
      fieldList.push({type: 'double', value: 'bonus', text: '奖金', dictCode: ''})
      this.superFieldList = fieldList
    }
  }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>