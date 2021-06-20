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

                <a-range-picker @change="onChange" />

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
                  <a-input placeholder="请输入三者保额档" v-model="thirdPartyInsuredZero"></a-input>
                </a-form-item>
              </a-col>
              <a-col :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="车损保额档">
                  <a-input placeholder="请输入0" v-model="carDamageInsuredZero" disabled></a-input>
                </a-form-item>
              </a-col>
              车损险为0返点比例：<a-input placeholder="请输入返点比例" v-model="rebateRatioZero"></a-input>


              <a-col :span="24">
                <a-range-picker @change="onChange" />
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
            <a-range-picker @change="onChange" />
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
            <a-range-picker @change="onChange" />
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
            <a-range-picker @change="onChange" />
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
            <a-form-item label="过户">
              <a-input v-model="isTransfer" placeholder="已过户" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-range-picker @change="onChange" />
          </a-col>
        </a-form-item>
      </a-row>

      <!--交叉返点比-->
      <a-row :gutter="24" v-if="showIndex == '6'">
        <a-form-item label="交叉返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="是否过户">
              <a-input  placeholder="未过户" v-model="isTransfer" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="续保标志">
              <a-input placeholder="市公司续保" v-model="renewalName" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-range-picker @change="onChange" />
          </a-col>
        </a-form-item>
      </a-row>

<!--跟单0返点-->
      <a-row :gutter="24" v-if="showIndex == '7'">
        <a-form-item label="跟单0返点">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="手续费">
              <a-input v-model="signFee" placeholder="手续费为0"></a-input>
            </a-form-item>
            <a-form-item label="返点比例">
              <a-input v-model="rebateRatio" placeholder="请输入返点比例"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-range-picker @change="onChange" />
          </a-col>
        </a-form-item>
      </a-row>

      <a-row :gutter="24" v-if="showIndex == '8'">
        <a-form-item label="座位保">
          <a-col :xl="8" :lg="7" :md="8" :sm="24">
            <a-form-item label="使用性质" v-for="item in usages">
              <a-input :value="item.text" disabled></a-input>
              返点比例：<a-input placeholder="请输入返点比例" v-model="rebateRatio"></a-input>
              <a-col  :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="司机责任险保额">
                  <a-input v-model="driverLiabilityInsured" placeholder="司机责任险保额"></a-input>
                </a-form-item>
              </a-col>
              <a-col :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="乘客责任险保额">
                  <a-input v-model="passengerLiabilityInsured" placeholder="乘客责任险保额"></a-input>
                </a-form-item>
              </a-col>
              <a-col  :xl="12" :lg="12" :md="12" :sm="24">
                <a-form-item label="奖金">
                  <a-input v-model="bonus" placeholder="座位保奖金"></a-input>
                </a-form-item>
              </a-col>
              <a-col :span="24">
                <a-range-picker @change="onChange" />
              </a-col>
            </a-form-item>
          </a-col>

        </a-form-item>
      </a-row>



      <template slot="footer">
        <a-button key="back" @click="handleCancel">
         取消
        </a-button>
        <a-button key="submit" type="primary" :loading="loading" @click="submitOk">
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


        rebateType: '',

        thirdPartyInsuredZero: '',//三者险中车损险为0时的三者保额档
        thirdPartyInsured: '',//三者保额档

        carDamageInsured: '',//车损险保额档
        carDamageInsuredZero: 0,

        renewalName: '市公司续保',
        rebateRatio: '',//返点比例
        rebateRatioZero: '',//三者险中车损险为0
        isTransfer: 1,
        signFee: 0,
        bonus: 0,
        driverLiabilityInsured: '',
        passengerLiabilityInsured: '',
        createTime_begin: '',//开始时间
        createTime_end: '',//结束时间
        showIndex: '',
        usages: [],//
        rebates: [],
        isShowBatchSettings: false,
        description: '返点比例管理页面',
        obj: [],
        //商业基础险类型数据
        BasicInsuranceRebate: {
          rebateType: '',
          usageType: '',
          rebateRatio: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },

        //三者险数据类型(车损险不为0)
        thirdPartyAbility: {
          rebateType: '',
          usageType: '',
          rebateRatio: '',
          carDamageInsured: '',
          thirdPartyInsured: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //三者险数据类型(车损险为0)
        thirdPartyAbilityZero: {
          rebateType: '',
          rebateRatioZero: '',
          usageType: '',
          thirdPartyInsuredZero: '',
          carDamageInsuredZero: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //次新车返点数据
        lastYearCar: {
          rebateType: '',
          rebateRatio: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //新车返点数据
        newCar: {
          rebateType: '',
          rebateRatio: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //  竞回返点数据
        contestRebate: {
          rebateType: '',
          rebateRatio: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //  过户返点数据
        transferRebate: {
          rebateType: '',
          rebateRatio: '',
          isTransfer: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //  交叉返点比数据
        overlappingRebate: {
          rebateType: '',
          rebateRatio: '',
          isTransfer: '',
          renewalName: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //跟单0比例数据
        followUpRebate: {
          rebateType: '',
          rebateRatio: '',
          signFee: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //座位保数据 （司机险 >= 20000 ）
        seatRebate: {
          rebateType: '',
          usageType: '',
          rebateRatio: '',
          driverLiabilityInsured: '',
          passengerLiabilityInsured: '',
          bonus: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },

        //座位保数据（司机险 < 20000）
        seatRebateLT: {
          rebateType: '',
          usageType: '',
          rebateRatio: '',
          driverLiabilityInsured: '',
          passengerLiabilityInsured: '',
          bonus: '',
          createTimeBegin: "",
          createTimeEnd: ""
        },
        //其他返点类型
        OtherRebate: {
          rebateType: '',
          rebateRatio: '',
          createTimeBegin: "",
          createTimeEnd: ""
        }
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
      onChange(date, dateString) {
        this.createTime_begin = dateString[0];
        this.createTime_end = dateString[1];
        console.log(date, dateString);
      },
      showrebate(item) {//字典查询使用性质
        console.log(item);
        this.showIndex = item.value
        //  查询所有车辆使用性质
        var str = 'insurance_usage,usage_name,usage_type';
        ajaxGetDictItems(str, null).then((res) => {
          if (res.success) {
            this.usages = res.result;
            // this.usages.forEach(v => {
            //   v.rebateRatio = ''
            //   v.createTime_begin = this.createTime_begin
            //   v.createTime_end = this.createTime_end
            // })
            console.log(this.usages);
            // this.options = res.result;
          }
        })
      },
      //提交
      submitOk() {
        if (this.rebateType == 0) {
          //  商业基础险类型
          this.usages.forEach(usageType => {
            this.BasicInsuranceRebate.usageType = usageType;
            this.BasicInsuranceRebate.rebateType = this.rebateType,
              this.BasicInsuranceRebate.rebateRatio = this.rebateRatio,

              this.obj.push(this.BasicInsuranceRebate);
          })
        }

        if (this.rebateType == 1) {
          //  三者险类型
          this.usages.forEach(usageType => {
            //车损险为0
            if (this.carDamageInsured == 0) {
              this.thirdPartyAbilityZero.usageType = usageType;
              this.thirdPartyAbilityZero.thirdPartyInsuredZero = thirdPartyInsuredZero;
              this.thirdPartyAbilityZero.carDamageInsuredZero = this.carDamageInsuredZero;
              this.thirdPartyAbilityZero.rebateRatioZero = this.rebateRatioZero;
              this.thirdPartyAbilityZero.rebateType = this.rebateType;
              this.thirdPartyAbilityZero.createTimeBegin = this.createTime_begin;
              this.thirdPartyAbilityZero.createTimeEnd = this.createTime_end;
              this.obj.push(this.thirdPartyAbilityZero);
            } else {
              this.thirdPartyAbilityZero.usageType = usageType;
              this.thirdPartyAbilityZero.thirdPartyInsured = thirdPartyInsured;
              this.thirdPartyAbilityZero.carDamageInsured = this.carDamageInsured;
              this.thirdPartyAbilityZero.rebateRatio = this.rebateRatio;
              this.thirdPartyAbilityZero.rebateType = this.rebateType;
              this.thirdPartyAbilityZero.createTimeBegin = this.createTime_begin;
              this.thirdPartyAbilityZero.createTimeEnd = this.createTime_end;
              this.obj.push(this.thirdPartyAbility);
            }
          })
        }
        //次新车返点
        if (this.rebateType == 2) {
          this.lastYearCar.rebateType = this.rebateType;
          this.lastYearCar.rebateRatio = this.rebateRatio;
          this.lastYearCar.createTimeBegin = this.createTime_begin;
          this.lastYearCar.createTimeEnd = this.createTime_end;
          this.obj.push(this.lastYearCar);
        }
        // 新车返点
        if (this.rebateType == 3) {
          this.newCar.rebateType = this.rebateType;
          this.newCar.rebateRatio = this.rebateRatio;
          this.newCar.createTimeBegin = this.createTime_begin;
          this.newCar.createTimeEnd = this.createTime_end;
          this.obj.push(this.newCar);
        }
        //竞回返点比
        if (this.rebateType == 4) {
          this.contestRebate.rebateType = this.rebateType;
          this.contestRebate.rebateRatio = this.rebateRatio;
          this.contestRebate.createTimeBegin = this.createTime_begin;
          this.contestRebate.createTimeEnd = this.createTime_end;
          this.obj.push(this.contestRebate);
        }
        //过户返点比
        if (this.rebateType == 5) {
          this.transferRebate.rebateType = this.rebateType;
          this.transferRebate.rebateRatio = this.rebateRatio;
          this.transferRebate.isTransfer = this.isTransfer;
          this.transferRebate.createTimeBegin = this.createTime_begin;
          this.transferRebate.createTimeEnd = this.createTime_end;
          this.obj.push(this.transferRebate);
        }

        //  交叉返点比
        if (this.rebateType == 6) {
          this.overlappingRebate.rebateType = this.rebateType;
          this.overlappingRebate.rebateRatio = this.rebateRatio;
          this.overlappingRebate.isTransfer = 0;
          this.overlappingRebate.renewalName = this.renewalName;

          this.overlappingRebate.createTimeBegin = this.createTime_begin;
          this.overlappingRebate.createTimeEnd = this.createTime_end;
          this.obj.push(this.transferRebate);
        }
        //跟单0返点
        if (this.rebateType == 7) {
          this.followUpRebate.rebateType = this.rebateType;
          this.followUpRebate.rebateRatio = this.rebateRatio;
          this.followUpRebate.signFee = this.signFee;
          this.followUpRebate.createTimeBegin = this.createTime_begin;
          this.followUpRebate.createTimeEnd = this.createTime_end;
          this.obj.push(this.followUpRebate);
        }
        //座位保返点比
        if (this.rebateType == 8) {
          if (this.driverLiabilityInsured <= 20000) {
            this.seatRebateLT.driverLiabilityInsured = -4;
            this.seatRebateLT.rebateType = this.rebateType;
            this.seatRebateLT.driverLiabilityInsured = this.driverLiabilityInsured;
            this.seatRebateLT.passengerLiabilityInsured = this.passengerLiabilityInsured;
            this.seatRebateLT.bonus = 0,
              this.seatRebateLT.createTimeBegin = this.createTime_begin;
            this.seatRebateLT.createTimeEnd = this.createTime_end;
            this.obj.push(this.seatRebateLT);
          } else {
            this.seatRebate.rebateType = this.rebateType;
            this.seatRebate.driverLiabilityInsured = this.driverLiabilityInsured;
            this.seatRebate.passengerLiabilityInsured = this.passengerLiabilityInsured;
            this.seatRebateLT.bonus = this.bonus;
            this.seatRebate.createTimeBegin = this.createTime_begin;
            this.seatRebate.createTimeEnd = this.createTime_end;
            this.seatRebate.rebateRatio = this.rebateRatio;
            this.obj.push(this.seatRebate);
          }
        }
        //其他返点比
        if (this.rebateType == 9) {
          this.OtherRebate.rebateType = this.rebateType;
          this.OtherRebate.rebateRatio = this.rebateRatio;
          this.OtherRebate.createTimeBegin = this.createTime_begin;
          this.OtherRebate.createTimeEnd = this.createTime_end;
          this.obj.push(this.OtherRebate);
        }
        //发起请求
        postAction("/rebate/insuranceRebateRatio/insertBatch", this.obj).then((res) => {

          if(res.success){
            this.$message.success(res.message)
          }else{
            this.$message.warning(res.message)
          }
        })
        console.log(this.obj);
      },
      handleCancel() {
        this.isShowBatchSettings = false;
      },
      initDictConfig() {
      },
      //显示所有返点类型
      getAllRebate() {//字典查询返点类型
        var str = 'rebate_ratio_type';
        ajaxGetDictItems(str, null).then((res) => {
          console.log(res, 557);
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