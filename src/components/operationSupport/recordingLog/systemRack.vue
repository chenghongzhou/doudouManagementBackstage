<template>
    <!-- 录音下榜列表 -->
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
			style="width:100%;" 
			v-loading="listLoading" 
			element-loading-text="拼命加载中" 
			element-loading-spinner="el-icon-loading" 
			element-loading-background="rgba(0, 0, 0, 0.8)" 
			:height="tableHeight">
				<el-table-column prop="time" label="日期"></el-table-column>
				<el-table-column prop="complain"  label="被投诉次数"></el-table-column>
				<el-table-column prop="id" label="录音编码" ></el-table-column>
				<el-table-column prop="uid" label="录音发布者" ></el-table-column>
                <el-table-column label="录音内容" width="300">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<audio 
							controls="controls" 
							:src="scope.row.voice_url"></audio>
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="content"  label="描述"></el-table-column>
				<el-table-column label="操作" min-width="120">
					<template slot-scope="scope">
                        <div v-if="scope.row.status==0">
                            <el-col :span="8">
								<el-button 
								size="warning" 
								type="info">状态不对</el-button>
							</el-col>
                        </div>
                        <div v-else-if="scope.row.status==1">
                            <el-col :span="8">
								<el-button 
								size="primary" 
								type="danger" 
								@click="dealVoiceOffData(scope.$index, scope.row)">再次推荐</el-button>
							</el-col>
                        </div>
                        <div v-else>
                            <el-col :span="8">
								<el-button 
								size="small" 
								type="danger" 
								@click="deleteVoiceComplain(scope.$index, scope.row)">删除</el-button>
							</el-col>
                        </div>
					</template>
				</el-table-column>
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
                    startDate: [new Date()-14*24*60*60*1000, new Date()], 
				},
				page: 0,
				listData: [],
				totalpage:null,
				formLabelWidth: '100px',
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
				let _this = this;
				_this.listLoading = true;
				let url = '/Voice/getVoiceOffData';
				let params ={
					date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
					date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
				}
				axios.get(allget+url, {params: params})
					.then((res) => {
						if(res.data.ret){
							_this.listLoading = false;
							this.listData = res.data.data;
						}else{
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
            },
            // 再次推荐
            dealVoiceOffData() {
                let _this = this;
                const id = row.id;
                const url = '/Voice/dealVoiceOffData';
                const params = {
                    id: id,
                };
                _this.listLoading = true;				
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							baseConfig.successTipMsg(_this, '已推荐');						
							_this.getData();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
						baseConfig.warningTipMsg(_this, 'error');
					});
            },
            // 删除
            deleteVoiceComplain() {
                let _this = this;
                const id = row.id;
                const url = '/Voice/deleteVoiceComplain';
                const params = {
                    id: id,
                };
                _this.listLoading = true;				
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							baseConfig.successTipMsg(_this, '已删除');						
							_this.getData();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
						baseConfig.warningTipMsg(_this, error);
					});
            }
		},
		mounted() {
			var _this = this;
			_this.tableHeight = baseConfig.lineNumber(searchHeight);
			_this.getData();
		}
    }

</script>

<style lang="css" scoped>

</style>
