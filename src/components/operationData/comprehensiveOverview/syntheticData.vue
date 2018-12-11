<template>
	<!-- 综合数据 -->
	<section>
		<!-- 综合数据  在线数据  付费渗透 -->
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 综合数据 -->
			<el-tab-pane 
			label="综合数据" 
			name="one" 
			:style="{height:tabSearchHeight+'px'}">
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
							<el-button 
							type="primary" 
							@click="getTableOne">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="onePageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="time" label="日期" width="200"></el-table-column>
						<el-table-column prop="device" label="设备激活" min-width="200"></el-table-column>
						<el-table-column prop="register" label="新增注册" min-width="200"></el-table-column>
						<el-table-column prop="active" label="日活跃用户数(DAU)" min-width="200"></el-table-column>
						<el-table-column prop="ACCU" label="ACCU" min-width="200"></el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="oneHandleCurrentChange" 
						:page-size="20" 
						:total="formOne.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- 在线数据 -->
			<el-tab-pane 
			label="在线数据" 
			name="two" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item>
							<div class="block">
								<span class="registerTime">日期</span>
								<el-date-picker 
								v-model="formTwo.choiceDate" 
								type="daterange" 
								range-separator=" 至 " 
								placeholder="选择日期范围"></el-date-picker>
							</div>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableTwo">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="twoPageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="time" label="日期" width="200"></el-table-column>
						<el-table-column prop="day_online_best" label="最高在线人数(PCU)" min-width="200"></el-table-column>
						<el-table-column prop="acu" label="平均在线人数(ACU)" min-width="200"></el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="twoHandleCurrentChange" 
						:page-size="20" 
						:total="formTwo.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- 付费渗透 -->
			<el-tab-pane 
			label="付费渗透" 
			name="three" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formThree">
						<el-form-item>
							<div class="block">
								<span class="registerTime">日期</span>
								<el-date-picker 
								v-model="formThree.choiceDate" 
								type="daterange" 
								range-separator=" 至 " 
								placeholder="选择日期范围"></el-date-picker>
							</div>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableThree">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="threePageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="time" label="日期" width="200"></el-table-column>
						<el-table-column prop="consume_rate" label="日付费率" min-width="200"></el-table-column>
						<el-table-column prop="consume_ARPPU" label="付费ARPPU值" min-width="200"></el-table-column>
						<el-table-column prop="active_ARPPU" label="活跃ARPU值" min-width="200"></el-table-column>
						<el-table-column prop="total_fee" label="充值金额" min-width="200"></el-table-column>
						<el-table-column prop="consume_user" label="付费用户" min-width="200"></el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="threeHandleCurrentChange" 
						:page-size="20" 
						:total="formThree.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- 累积数据 -->
			<el-tab-pane 
			label="累积数据" 
			name="four" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formFour">
						<el-form-item>
							<div class="block">
								<span class="registerTime">日期</span>
								<el-date-picker 
								v-model="formFour.choiceDate" 
								type="daterange" 
								range-separator=" 至 " 
								placeholder="选择日期范围"></el-date-picker>
							</div>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="formFour">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="fourPageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="device" label="设备激活" width="200"></el-table-column>
						<el-table-column prop="register" label="注册用户" min-width="200"></el-table-column>
						<el-table-column prop="consume_user" label="付费用户" min-width="200"></el-table-column>
						<el-table-column prop="total_fee" label="充值金额" min-width="200"></el-table-column>
						<el-table-column prop="enchashment" label="提现" min-width="200"></el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="fourHandleCurrentChange" 
						:page-size="20" 
						:total="formFour.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
		</el-tabs>
	</section>
</template>

<script>
import Event from './../../../public_js/event.js';
import { allget } from '../../../api/api';
import store from '../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null, 
			tabSearchHeight: null,
			formOne: {
				choiceDate: [new Date()-15*24*60*60*1000, new Date()],
				position: '',
				TabData: [],
				TotalPage: null,
				Page: 1,
				star: '0',
				end: '20', 
			},
			formTwo: {
				choiceDate: [new Date()-15*24*60*60*1000, new Date()], 
				TabData: [], 
				TotalPage: null,
				Page: 1, 
				star: '0', 
				end: '20', 
			},
			formThree: {
				choiceDate: [new Date()-15*24*60*60*1000, new Date()],
				TabData: [], 
				TotalPage: null,
				Page: 1,
				star: '0',
				end: '20', 
			},
			formFour: {
				choiceDate: [new Date()-15*24*60*60*1000, new Date()],
				TabData: [], 
				TotalPage: null,
				Page: 1,
				star: '0', 
				end: '20',
			},
			listLoading: false,
			tabActiveName: 'one', // 设置为tab切换栏的选中不同的状态(one、two、three)
			formLabelWidth: '130px',
		};
	},
	computed:{
		onePageData() {
			var _this = this;
			return _this.formOne.TabData.slice(_this.formOne.star, _this.formOne.end);
		},
		twoPageData() {
			var _this = this;
			return _this.formTwo.TabData.slice(_this.formTwo.star, _this.formTwo.end);
		},
		threePageData() {
			var _this = this;
			return _this.formThree.TabData.slice(_this.formThree.star, _this.formThree.end);
		},
		fourPageData() {
			var _this = this;
			return _this.formFour.TabData.slice(_this.formFour.star, _this.formFour.end);
		},
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
			_this.formOne.Page = val;
			_this.formOne.star = (_this.formOne.Page-1)*20;
			_this.formOne.end = _this.formOne.star+20;
		},
		twoHandleCurrentChange(val) {
			var _this = this;
			_this.formTwo.Page = val;
			_this.formTwo.star = (_this.formTwo.Page-1)*20;
			_this.formTwo.end = _this.formTwo.star+20;
		},
		threeHandleCurrentChange(val) {
			var _this = this;
			_this.formThree.Page = val;
			_this.formThree.star = (_this.formThree.Page-1)*20;
			_this.formThree.end = _this.formThree.star+20;
		},
		fourHandleCurrentChange(val) {
			var _this = this;
			_this.formFour.Page = val;
			_this.formFour.star = (_this.formFour.Page-1)*20;
			_this.formFour.end = _this.formFour.star+20;
		},
		// 在记录中进行搜索条件
		searchConditionOne() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			return obj; 
		},
		searchConditionTwo() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0);
			return obj; 
		},
		searchConditionThree() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formThree.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formThree.choiceDate[1], 0);
			return obj; 
		},
		searchConditionFour() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formFour.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formFour.choiceDate[1], 0);
			return obj; 
		},
		// 获取查询到的数据
		getTableOne() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Base/comprehensiveData';
			var params = _this.searchConditionOne();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							_this.formOne.TotalPage = res.data.day_active.length;
							_this.formOne.TabData = res.data.day_active;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		getTableTwo() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Base/comprehensiveData';
			var params = _this.searchConditionTwo();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						_this.formTwo.TotalPage = res.data.time_data.length;
						_this.formTwo.TabData = res.data.time_data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
			}
		},
		getTableThree() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Base/comprehensiveData';
			var params = _this.searchConditionThree();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						_this.formThree.TotalPage = res.data.consume.length;
						_this.formThree.TabData = res.data.consume;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
			}
		},
		getTableFour() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Base/comprehensiveData';
			var params = _this.searchConditionThree();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							_this.formFour.TotalPage = 1;
							var arr = [];
							arr.push(res.data.total_data);
							_this.formFour.TabData = arr;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 顶部tab进行页面的切换
		handleClick(tab, event) {
			var _this = this;
			// console.log(tab.label);
		},
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(tabSearchPageHeight);
			_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight);			
			_this.getTableOne();
			_this.getTableTwo();
			_this.getTableThree();
			_this.getTableFour();
		})
	}
};
</script>

<style lang="css" scoped>
</style>
