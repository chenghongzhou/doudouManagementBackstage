<template>
    <!-- 主播每日数据汇总 -->
	<section>
		<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
			<el-form :inline="true" style="overflow:hidden;">
				<el-form-item>
					<div class="block">
						<span class="registerTime">日期</span>
						<el-date-picker 
						v-model="formOne.startDate" 
						type="daterange" 
						range-separator=" 至 " 
						start-placeholder="开始日期" 
						end-placeholder="结束日期"></el-date-picker>
					</div>
				</el-form-item>
                <el-form-item style="float:right;">
					<el-button 
					type="primary" 
					@click="getData()">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			:data="listData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			element-loading-text="拼命加载中"
			element-loading-spinner="el-icon-loading" 
			element-loading-background="rgba(0, 0, 0, 0.8)"  
			style="width:100%;"  
			:height="tableHeight">
				<el-table-column prop="date" label="日期"></el-table-column>
				<el-table-column prop="new_anchor"  label="新增主播数"></el-table-column>
				<el-table-column prop="total_anchor" label="累积主播数"></el-table-column>
				<el-table-column prop="active_anchor" label="积极主播数" ></el-table-column>
				<el-table-column prop="day_chat_ticket" label="主播累积豆票"></el-table-column>
				<el-table-column prop="day_money" label="主播累积收入(元)" sortable></el-table-column>
				<el-table-column prop="call_chat_ticket" label="付费通话豆票"></el-table-column>
				<el-table-column prop="gift_chat_ticket" label="礼物豆票"></el-table-column>
				<el-table-column prop="eavesdrop_chat_ticket" label="被偷听豆票"></el-table-column>
			</el-table>
		</template>
    </section>
</template>

<script>
	import { allget } from '../../../api/api';
	import axios from 'axios';
    export default {
        data() {
            return {
				tableHeight: null, 
                formOne: {
                    startDate: [new Date()-7*24*60*60*1000, new Date()], 
				},
				listData: [],
				formLabelWidth: '120px',
				listLoading: false,
				page: 0,
            }
        },
		methods: {
			handleCurrentChange(val) {
				this.page = val - 1;
				this.getData();
			},
			// 获取数据
			getData() {
				var _this = this;
				_this.listLoading = true;
				let url = '/Anchor/getAnchorDayTotalData';
				let params ={
					date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
					date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
					page: this.page,
				}
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret){
							this.listData = res.data.data;
						}else{
							baseConfig.warningTipMsg(_this,res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			},
		},
		mounted() {
			var _this = this;
			_this.tableHeight = baseConfig.lineNumber(searchHeight);
			_this.getData();
		},
    }

</script>

<style lang="css" scoped>

</style>
