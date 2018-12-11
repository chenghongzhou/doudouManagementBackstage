<template>
    <!-- 代理绑定操作记录 -->
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
                <el-form-item style="margin-left:100px;">
					<span>操作类型</span>
					<el-select style="width: 150px;" v-model="searchType">
						<el-option label="全部" value=""></el-option>
						<el-option label="绑定" value="0"></el-option>
						<el-option label="解绑" value="1"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item style="float:right;">
					<el-button 
					type="primary" 
					@click="getData(0)">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			:data="listData" 
			v-loading="listLoading" 
			element-loading-text="拼命加载中" 
			element-loading-spinner="el-icon-loading" 
			element-loading-background="rgba(0, 0, 0, 0.8)"  
			border fit highlight-current-row 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="operate_time" label="操作日期"></el-table-column>
				<el-table-column prop="uid"  label="UID"></el-table-column>
				<el-table-column prop="change_uid_list" label="被操作的uid"></el-table-column>
				<el-table-column prop="type" label="类型" :formatter="judgeType"></el-table-column>
				<el-table-column prop="operate_user" label="操作人"></el-table-column>
			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
				layout="total,prev,pager,next,jumper" 
				@current-change="handleCurrentChange" 
				:page-size="20" 
				:total=1000 
				style="float:right;"></el-pagination>
			</el-col>
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
                    startDate: [new Date()-1*24*60*60*1000, new Date()], 
				},
				page: 0,
				listData: [],
				totalpage:null,
				formLabelWidth: '120px',
                listLoading: false,
                searchType: null,
            }
        },
		methods: {
			handleCurrentChange(val) {
				this.page = val-1;
				this.getData();
			},
			// 获取数据
			getData(type) {
				var _this = this;
				_this.listLoading = true;
				if(type==0){
					_this.page = 0;
				}
				let url = '/Agent/getAgentChangeRecord';
				let params = {
					page: this.page,
					date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                    date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                    type: this.searchType,
				}
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret){
							this.listData = res.data.data;
						}else{
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err)
					});
            },
            judgeType(row){
                return row.type==0 ? "绑定" : "解绑";
            }
		},
		mounted() {
			var _this = this;
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getData();
		},
    }

</script>

<style lang="css" scoped>

</style>
