<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="出单日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="createTime">
              <j-date placeholder="请选择创建日期"  v-model="model.createTime" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="返点类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="rebateRatioType">
              <j-dict-select-tag type="list" v-model="model.rebateRatioType" @input="changeRebate" dictCode="rebate_ratio_type" placeholder="请选择返点类型" :disabled="isShow" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowTransfer">
            <a-form-model-item label="是否过户" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="isTransfer">
              <j-dict-select-tag type="list" v-model="model.isTransfer"  dictCode="is_transfer" placeholder="请选择是否过户" disabled />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowRenewal">
            <a-form-model-item label="续保类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="renewalType">
              <j-dict-select-tag type="list" v-model="model.renewalType" dictCode="renewal_symbol" placeholder="请选择续保类型"  disabled/>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowUsage">
            <a-form-model-item label="使用性质" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="usageType">
              <j-dict-select-tag type="list" v-model="model.usageType" dictCode="insurance_usage,usage_name,usage_type" placeholder="请选择使用性质" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowThirdPartyAbility">
            <a-form-model-item label="三责保额档" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="thirdPartyInsured">
              <a-input v-model="model.thirdPartyInsured" placeholder="请输入三责保额档"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowCarDamageInsured">
            <a-form-model-item label="车损险保额档" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="carDamageInsured">
              <a-input v-model="model.carDamageInsured" placeholder="请输入车损险保额档"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowDriverLiabilityInsured">
            <a-form-model-item label="司机责任险保额档" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="driverLiabilityInsured">
              <a-input v-model="model.driverLiabilityInsured" placeholder="请输入司机责任险保额档"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowPassengerLiability">
            <a-form-model-item label="乘客责任险保额档" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="passengerLiability">
              <a-input v-model="model.passengerLiability" placeholder="请输入乘客责任险保额档"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowBonus">
            <a-form-model-item label="奖金" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="bonus">
              <a-input-number v-model="model.bonus" placeholder="请输入奖金" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24" v-if="isShowSignFee">
            <a-form-model-item label="签单手续费" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="signFee">
              <a-input v-model="model.signFee" placeholder="请输入签单手续费" disabled></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24"  v-if="isShowRebateRatio">
            <a-form-model-item label="返点比例" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="rebateRatio">
              <a-input-number v-model="model.rebateRatio" placeholder="请输入返点比例" style="width: 100%" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>
<script>
  import { httpAction, getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'
  import JDictSelectTag from "@comp/dict/JDictSelectTag";
  export default {
    name: 'InsuranceRebateRatioForm',
    components: {
      JDictSelectTag
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        isShow: false,
        isTransferValue: '',
        isDisableIsTransfer:true,
        isShowTransfer: false,
        isShowUsage: false,
        isShowRenewal: false,
        isShowThirdPartyAbility: false,
        isShowCarDamageInsured: false,
        isShowDriverLiabilityInsured: false,
        isShowPassengerLiability: false,
        isShowBonus: false,
        isShowRebateRatio: false,
        isShowSignFee: false,
        model:{
          usageType: '',
          isTransfer: '',
          renewalType: '',
          signFee: 0,
         },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
           rebateRatioType: [
              { required: true, message: '请输入返点类型!'},
           ],
           rebateRatio: [
              { required: true, message: '请输入返点比例!'},
           ],
        },
        url: {
          add: "/rebate/insuranceRebateRatio/add",
          edit: "/rebate/insuranceRebateRatio/edit",
          queryById: "/rebate/insuranceRebateRatio/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
    },
    created () {
       //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
    },
    methods: {
      // add () {
      //   // this.edit(this.modelDefault);
      // },
      changeRebate(val){
        // console.log(val);
        if(val == 0){//商业险返点
          this.isShowUsage = true;
          this.isShowRebateRatio = true;
          /*其他*/
          this.isShowThirdPartyAbility = false;
          this.isShowCarDamageInsured = false;
          this.isShowRenewal = false;
          this.isShowTransfer = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;
          this.isShowSignFee = false;
        }
        if(val == 1) {//三者险返点
          this.isShowUsage = true;
          this.isShowThirdPartyAbility = true;
          this.isShowCarDamageInsured = true;
          this.isShowRebateRatio = true;
          this.isShowSignFee = false;
          /*其他*/
          this.isShowRenewal = false;
          this.isShowTransfer = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;
        }
        if (val == 2){//次新车返点
          this.isShowRebateRatio = true;
          this.isShowRenewal = true;
          /*其他*/
          this.isShowThirdPartyAbility = false;
          this.isShowUsage = false;
          this.isShowCarDamageInsured = false;
          this.isShowTransfer = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;
          this.isShowSignFee = false;
          this.model.renewalType = '2';
        }
        if(val == 3){//新车返点
          this.isShowRenewal = true;
          this.isShowRebateRatio = true;

          /*其他*/
          this.isShowThirdPartyAbility = false;
          this.isShowUsage = false;
          this.isShowCarDamageInsured = false;
          this.isShowTransfer = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;
          this.isShowSignFee = false;
          this.model.renewalType = "1";
        }
        if(val == 4){//竞回返点
          this.isShowRenewal = true;
          this.isShowRebateRatio = true;

          /*其他*/
          this.isShowThirdPartyAbility = false;
          this.isShowUsage = false;
          this.isShowCarDamageInsured = false;
          this.isShowTransfer = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;
          this.isShowSignFee = false;
          this.model.renewalType = "4";

        }
        if(val == 5){//过户返点比
          this.isShowTransfer = true;
          this.isShowRebateRatio = true;

          /*其他*/
          this.isShowThirdPartyAbility = false;
          this.isShowRenewal = false;
          this.isShowUsage = false;
          this.isShowCarDamageInsured = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;

          this.isShowSignFee = false;
          this.model.isTransfer = "1";
        }
        if(val == 6){//交叉返点比
          this.isShowTransfer = true;
          this.isShowRebateRatio = true;
          this.isShowRenewal = true;

          /*其他*/
          this.isShowSignFee = false;
          this.isShowThirdPartyAbility = false;
          this.isShowUsage =  false;
          this.isShowCarDamageInsured = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;
          this.model.isTransfer = '0';
          this.model.renewalType = '5';
        }
        if(val == 7){//跟单零比例
          this.isShowRebateRatio = true;

          this.isShowSignFee = true;
          /*其他*/
          this.isShowThirdPartyAbility = false;
          this.isShowTransfer = false;
          this.isShowRenewal = false;
          this.isShowUsage =  false;
          this.isShowCarDamageInsured = false;
          this.isShowDriverLiabilityInsured = false;
          this.isShowPassengerLiability = false;
          this.isShowBonus = false;
        }
        if(val == 8){//座位保比例
          this.isShowRebateRatio = true;
          this.isShowRenewal = false;
          this.isShowDriverLiabilityInsured = true;
          this.isShowPassengerLiability = true;
          this.isShowBonus = true;
          this.isShowSignFee = false;
          /*其他*/
          this.isShowThirdPartyAbility = false;
          this.isShowTransfer = false;
          this.isShowUsage =  true;
          this.isShowCarDamageInsured = false;
        }
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
        this.changeRebate(this.model.rebateRatioType);
        this.isShow = true;
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            httpAction(httpurl,this.model,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }

        })
      },
    }
  }
</script>