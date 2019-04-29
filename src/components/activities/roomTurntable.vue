<template>
	<!-- 房间转盘，房间数据、个人数据 -->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 房间数据 -->
			<el-tab-pane 
			label="房间数据" 
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
						<el-form-item label="房间ID">
							<el-input 
							v-model="formOne.room_id" 
							auto-complete="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableFind">查询</el-button>
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
						<el-table-column prop="count" label="游戏次数" min-width="80"></el-table-column>
						<el-table-column prop="gold" label="花费豆币" min-width="80"></el-table-column>
						<el-table-column prop="ticket" label="获得豆票" min-width="80"></el-table-column>
						<el-table-column prop="count_1" label="礼物1次数" min-width="80"></el-table-column>
						<el-table-column prop="count_2" label="礼物2次数" min-width="80"></el-table-column>
						<el-table-column prop="count_3" label="礼物3次数" min-width="80"></el-table-column>
						<el-table-column prop="count_4" label="礼物4次数" min-width="80"></el-table-column>
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
			<!-- 个人数据 -->
			<el-tab-pane 
			label="个人数据" 
			name="second" 
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
						<el-form-item label="用户ID">
							<el-input 
							v-model="formTwo.uid" 
							auto-complete="off"></el-input>
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
					:data="formTwo.TabData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:row-class-name="tableRowClassNameElse"
					:height="tableHeight">
						<el-table-column prop="date" label="日期" width="200"></el-table-column>
						<el-table-column prop="count" label="游戏次数" min-width="80"></el-table-column>
						<el-table-column prop="gold" label="花费豆币" min-width="80"></el-table-column>
						<el-table-column prop="ticket" label="获得豆票" min-width="80"></el-table-column>
						<el-table-column prop="count_1" label="礼物1次数" min-width="80"></el-table-column>
						<el-table-column prop="count_2" label="礼物2次数" min-width="80"></el-table-column>
						<el-table-column prop="count_3" label="礼物3次数" min-width="80"></el-table-column>
						<el-table-column prop="count_4" label="礼物4次数" min-width="80"></el-table-column>
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
				room_id: '',
				TabData: [],
				TotalPage: 1000, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			formTwo: {
				choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				uid: '',
				TabData: [], 
				TotalPage: 1000, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			listLoading: false, 
			tabActiveName: 'second',
			formLabelWidth: '130px', 
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
            _this.formOne.Page = val-1;
            _this.getTableFind();
		},
		twoHandleCurrentChange(val) {
			var _this = this;
            _this.formTwo.Page = val-1;
            _this.getTableManage();
		},
		searchConditionFind() {
			var _this = this;
			var obj = {};
			obj.start_date = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.end_date = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.room_id = _this.formOne.room_id;
			obj.page = _this.formOne.Page;
			return obj; 
		},
		// 获取家族数据统计的数据
		getTableFind() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getRoomTurnTableRoomData';
			var params = _this.searchConditionFind();
			if(params==null) {
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							var obj = {};
							obj = res.data.data.total;
							obj.date = '总计';
							res.data.data.list.unshift(obj);
							_this.formOne.TabData = res.data.data.list;
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
		searchConditionManage() {
			var _this = this;
			var obj = {};
			obj.start_date = baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0);
			obj.end_date = baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0);
			obj.uid = _this.formTwo.uid;
			obj.page = _this.formTwo.Page;
			return obj;
		},
		// 
		getTableManage() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getRoomTurnTableUserData';
			var params = _this.searchConditionManage();
			if(params==null) { 
				_this.listLoading = false; 
			} else {
                axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							var obj = res.data.data.total;
							obj.date = '总计';
							res.data.data.list.unshift(obj);
							_this.formTwo.TabData = res.data.data.list;
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
			// _this.getTableFind();
			// _this.getTableManage();
		})
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
</style>
