<template>
    <!-- 每日新增录音统计 -->
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
						end-placeholder="结束日期">
						</el-date-picker>
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
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="date" label="时间"></el-table-column>
				<el-table-column prop="click_people"  label="录播人数"></el-table-column>
				<el-table-column prop="new_people" label="成功发布录音人数" ></el-table-column>
				<el-table-column prop="new_num" label="新增录音数"></el-table-column>
				<el-table-column prop="new_money_num" label="新增付费录音数"></el-table-column>
				<el-table-column prop="new_free_num"  label="新增免费录音数"></el-table-column>
				<el-table-column prop="total_free_num" label="平台免费录音总数"></el-table-column>
				<el-table-column prop="total_money_num" label="平台付费录音总数"></el-table-column>
				<el-table-column prop="total_listen_money" label="听录音花费豆币数"></el-table-column>
				<el-table-column prop="total_award_money" label="打赏录音豆币数"></el-table-column>
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
                    startDate: [new Date()-14*24*60*60*1000, new Date()],
				},
				value6: '',
				value10: 'text',
				page: 0,
				listData: [],
				totalpage:null,
				formLabelWidth: '120px',
				listLoading: false,
            }
        },
		methods: {
			handleCurrentChange(val) {
				this.page = val-1;
				this.getData();
			},
			// 获取数据
			getData() {
				var _this = this;
				this.listLoading = true;
				let url = '/Voice/getVoiceDayData';
				let params = {
					date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
					date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
				}
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret){
							this.listData = res.data.data;
						}else{
							baseConfig.warningTipMsg(this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			},
		},
		mounted() {
			var _this = this;
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getData();
		}
    }

</script>

<style lang="css" scoped>

</style>
