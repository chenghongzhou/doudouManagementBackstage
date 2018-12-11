<template>
	<!-- 任务进度查询 -->
	<section>
		<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
			<el-form :inline="true" style="overflow:hidden;" :model="formOne">
				<el-form-item>
					<div class="block">
						<span class="registerTime">日期</span>
						<el-date-picker 
						v-model="formOne.choiceDate" 
						type="daterange" 
						range-separator=" 至 " 
						placeholder="选择日期范围"></el-date-picker>
					</div>
				</el-form-item>
				<el-form-item>
                    <span>UID：</span>
					<el-input 
					placeholder="UID" 
					style="width: 120px;" 
					clearable 
					v-model="formOne.uid" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item>
                    <span>任务类型</span>
                    <el-select style="width: 120px;" v-model="formOne.type">
                        <el-option label="全部" value=""></el-option>
                        <el-option label="每日任务" value="2"></el-option>
                        <el-option label="成就任务" value="3"></el-option>
                    </el-select>
                </el-form-item>
				<el-form-item>
                    <span>动作类型</span>
                    <el-select style="width:200px;" v-model="action_type">
						<el-option 
						v-for="item in formOne.action_type" 
						:key="item.value" 
						:label="item.name" 
						:value="item.action_type"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item style="float:right;">
					<el-button 
					type="primary" 
					@click="getTableData(0)">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			ref="tableHeight" 
			:data="tabData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="uid" label="UID" ></el-table-column>
				<el-table-column prop="nickname" label="昵称" ></el-table-column>
				<el-table-column prop="type" label="任务类型" >
					<template slot-scope="scope">
						<p v-if="scope.row.type == 2">每日任务</p>
						<p v-else-if="scope.row.type == 3">成就任务</p>
					</template>
				</el-table-column>
				<el-table-column prop="name" label="任务名称" ></el-table-column>
				<el-table-column prop="desc" label="描述" min-width="200" ></el-table-column>
				<el-table-column prop="finish_num" label="完成进度" ></el-table-column>
				<el-table-column prop="task_num" label="任务量" ></el-table-column>
				<el-table-column prop="date" label="时间" ></el-table-column>
				<el-table-column prop="reward_type" label="奖励类型" >
					<template slot-scope="scope">
						<p v-if="scope.row.reward_type == 0">豆币</p>
						<p v-else-if="scope.row.reward_type == 1">好评分</p>
						<p v-else-if="scope.row.reward_type == 2">料片</p>
						<p v-else-if="scope.row.reward_type == 3">无奖励</p>
						<p v-else-if="scope.row.reward_type == 4">经验值</p>
					</template>
				</el-table-column>
				<el-table-column prop="reward_num" label="奖励数量" ></el-table-column>
			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
				layout="total,prev,pager,next,jumper" 
				@current-change="handleCurrentChange" 
				:page-size="20" 
				:total="totalpage" 
				style="float:right;"></el-pagination>
			</el-col>
		</template>
	</section>
</template>

<script>
import Event from './../../../public_js/event.js';
import { allget } from './../../../api/api';
import store from './../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null,
			formOne: {
				choiceDate: [new Date()-30*24*60*60*1000, new Date()], 
				uid: '',
				type: '',
				action_type: [{action_type:"",name:"全部"}], 
			},
			listLoading: false, 
			tabData: [], 
			totalpage: 1000,  
			page: 0, 
			formLabelWidth: '130px', 
			action_type: '',
		};
	},
	methods: {
		handleCurrentChange(val) {
            this.page = val - 1;
            this.getTableData();
		},
		// 搜索条件
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.uid = _this.formOne.uid;
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.page = _this.page;
			obj.action_type = _this.action_type;
			return obj; 
		},
		
		// 获取数据列表
		getTableData(type) {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewLevel/getTaskRecordList';
            var params = _this.searchCondition();
			if(params==null) {
				_this.listLoading = false;
			} else {
				if(type == 0){
					params.page = 0;
				}
				axios.get(allget+url, { params: params })		
					.then(res => {
						_this.listLoading = false;
						if(res.data.ret) {
							_this.tabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		
		// 获取动作类型
		getActionType() {
			var _this = this;
			var url = '/NewLevel/getTaskActionTypeList';
			var params = {};
			axios.get(allget+url, {params: params})
				.then((res) => {
					if(res.data.ret){
						for(var i = 0; i < res.data.data.length; i++){
							_this.formOne.action_type.push(res.data.data[i]);
						}
					}else{
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
		}
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getTableData();
			_this.getActionType();
		})
	}
};
</script>

<style lang="css" scoped>
</style>
