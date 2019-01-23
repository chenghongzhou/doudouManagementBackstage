<template>
	<!-- 集卡-->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 活动数据统计 -->
			<el-tab-pane 
			label="活动数据统计" 
			name="first" 
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
							@click="getTableManage">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="formOne.TabData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:row-class-name="tableRowClassName"
					:height="tableHeight">
						<el-table-column prop="date" label="日期" width="200"></el-table-column>
						<el-table-column prop="total" label="活动参与人数" min-width="100"></el-table-column>
						<el-table-column prop="successful" label="集卡人数" min-width="100"></el-table-column>
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
			<!-- 开奖数据统计 -->
			<el-tab-pane 
			label="开奖数据统计" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
			<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item label="用户ID">
							<el-input 
							v-model="formTwo.uid" 
							auto-complete="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTablePrize">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="formTwo.TabDataPrize" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:row-class-name="tableRowClassNameElse"
					:height="tableHeight">
						<el-table-column prop="uid" label="UID" width="200"></el-table-column>
						<el-table-column prop="prize" label="获得喵钻数量" min-width="80"></el-table-column>
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
			<!-- 活动执行 -->
			<el-tab-pane 
			label="活动执行" 
			name="three" 
			:style="{height:tabSearchHeight+'px'}" style="margin-top:50px" v-if="activity">
				<template>
					<ul class="activity-box">
						<li>虚拟集齐人数</li>
						<li @dblclick="clickEdit"><span v-if="spanIsShow">{{activity.user_false}}</span><input type='text' v-model="activity.user_false" class="user_false_box" v-if="inputIsShow"></li>
						<li>
							<el-button 
							type="primary" 
							@click="userFalseBtn"
							size="small">确定</el-button>
						</li>
					</ul>
					<ul class="activity-box">
						<li>奖金</li>
						<li>{{activity.prize}}</li>
						<li>
							<el-button 
							type="primary" 
							@click.native.prevent="numDialog.dialogShow=true"
							size="small">编辑</el-button>
						</li>
					</ul>
				</template>
			</el-tab-pane>
			<el-dialog 
				title="奖金发放"
				:visible.sync="numDialog.dialogShow" v-if="activity">
					<el-form :model="numDialog">
						<el-form-item label="集齐人数" :label-width="formLabelWidth">
							<el-input 
							v-model="activity.user_true" :disabled="true"></el-input>
						</el-form-item>
					</el-form>
					<el-form :model="numDialog">
						<el-form-item label="奖金数目" :label-width="formLabelWidth">
							<el-input 
							v-model="numDialog.prize"></el-input>
						</el-form-item>
					</el-form>
					<div slot="footer" class="dialog-footer">
						<el-button 
						@click.native.prevent="changeNum(0)">取 消</el-button>
						<el-button 
						type="primary" 
						@click.native.prevent="changeNum(1)">确 定</el-button>
					</div>
				</el-dialog>
		</el-tabs>
	</section>
</template>

<script>
import Event from './../../public_js/event.js';
import store from '../../vuex/store';
import axios from 'axios';
import { allget } from '../../api/api.js';
export default {
	data() {
		return {
			tableHeight: null, 
			tabSearchHeight: null,
			formOne: {
                choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				TabData: [],
				TotalPage: 1000, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			formTwo: {
				uid: '',
				prize: '',
				TabDataPrize: [],
				TotalPage: 1000, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			numDialog: {
				dialogShow: false,
				prize:''
			},
			activity:null,
			listLoading: false, 
			tabActiveName: 'second',
			formLabelWidth: '130px', 
			inputIsShow:false,
			spanIsShow:true
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
            _this.formOne.Page = val-1;
            _this.getTableManage();
		},
		twoHandleCurrentChange(val) {
			var _this = this;
            _this.formTwo.Page = val-1;
            _this.getTablePrize();
		},
		// 在列表中进行搜索条件
		searchConditionManage() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.page = _this.formOne.Page;
			return obj;
		},
		// 活动数据统计
		getTableManage() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getGreetingCardUser';
			var params = _this.searchConditionManage();
			if(params==null) { 
				_this.listLoading = false; 
			} else {
                axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							var arr = [];
							for(var i=0; i<res.data.data.length; i++) {
								for(var key in res.data.data[i]) {
									var obj = {};
									obj.date = res.data.data[i].date;
									obj.successful = res.data.data[i].successful;
									obj.total = res.data.data[i].total;
								}
								arr.push(obj);
							}
							_this.formOne.TabData = arr;
							_this.formOne.totalPage = arr.length;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 在列表中进行搜索条件
		searchConditionPrize() {
			var _this = this;
			var obj = {};
			obj.uid = _this.formTwo.uid;
			obj.Page = _this.formTwo.Page;
			return obj;
		},
		getTablePrize(){
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getGreetingCardPrize';
			var params = _this.searchConditionPrize();
			if(params==null) { 
				_this.listLoading = false; 
			} else {
                axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							var arr = [];
							for(var i=0; i<res.data.data.length; i++) {
								for(var key in res.data.data[i]) {
									var obj = {};
									obj.uid = res.data.data[i].uid;
									obj.prize = res.data.data[i].prize;
								}
								arr.push(obj);
							}
							_this.formTwo.TabDataPrize = arr;
							_this.formTwo.totalPage = arr.length;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		//活动执行
		getTableActivity(){
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getGreetingCardSetting';
			 axios.get(allget+url)
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							_this.activity = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
					});
		},
		//双击编辑
		clickEdit(){
			var _this = this;
			_this.inputIsShow = true;
			_this.spanIsShow = false;
		},
		//修改虚拟人数
		userFalseBtn(){
			var _this = this ;
			if(!_this.inputIsShow){
				baseConfig.normalTipMsg(_this, '请在左侧数字双击进行修改',2); 
				return false;
			};
			_this.listLoading = true;
			var params = {
				user : _this.activity.user_false
			};
			var url = '/NewActivity/setGreetingCardSuccessfulUser';
			 axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							_this.inputIsShow = false;
							_this.spanIsShow = true;
							baseConfig.successTipMsg(_this, res.data.msg);
							_this.getTableActivity();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
					});
		},
		//修改奖金
		changeNum(type) {
			var _this = this;
			if(type==0) {
				_this.numDialog.dialogShow = false;
				_this.numDialog.prize = '';
			} 
			else if(type==1) {
				var url = 'NewActivity/prepareGreetingCardPrize';
				var params = {
					prize: _this.numDialog.prize
				};
				axios.get(allget+url, { params: params })
				.then((res) => {
					if(res.data.ret) {
						_this.getTableActivity();
						baseConfig.successTipMsg(_this, res.data.msg);
						_this.numDialog.prize = '';
					} else {
						baseConfig.normalTipMsg(_this, res.data.msg);
					}
					_this.numDialog.dialogShow = false;
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
		tableRowClassName({row, rowIndex}) {
			if(rowIndex===0) {
				return 'warning-row';
			} 
			return '';
		},
		tableRowClassNameElse({row, rowIndex}) {
			if(rowIndex===0) {
				return 'warning-row';
			} 
			return '';
		},
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
				_this.tableHeight = baseConfig.lineNumber(tabSearchPageHeight);
				_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight);
				_this.getTablePrize();
				_this.getTableManage();
				_this.getTableActivity();
			});
		}
};
</script>

<style lang="css" scoped>
.demo-ruleForm{
	width: 500px;
}
.el-tab-pane{
	height: 800px;
}
.el-table .warning-row {
	background: oldlace;
}
.activity-box{
	list-style: none;
	height: 50px;
	width: 80%;
	/* border-bottom: 1px solid #909399; */
	margin-left: 10px;
}
.activity-box li{
	width: 45%;
	float: left;
	text-align: center;
	color: #909399;
	font-size: 14px;
	line-height: 50px;
}
.activity-box li:first-child{
	width:10%;
}
.user_false_box{
	width: 30%;
	height:30px;
}
</style>
