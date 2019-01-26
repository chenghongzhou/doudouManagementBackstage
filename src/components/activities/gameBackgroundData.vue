<template>
	<!-- 游戏数据后台 -->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 房间游戏数据 -->
			<el-tab-pane 
			label="房间游戏数据" 
			name="first" 
			:style="{height:tabHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item label="房间ID">
							<el-input 
							v-model="formOne.room_id" 
							auto-complete="off"></el-input>
						</el-form-item>
						<el-form-item>
							<div class="block">
								<span class="registerTime">日期</span>
								<el-date-picker 
								v-model="formOne.choiceDate" 
								type="datetimerange" 
								range-separator=" 至 " 
								placeholder="选择时间范围"></el-date-picker>
							</div>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableFind">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<div class="total_data" id="total_data">
					<el-row>
						<el-col :span="3">
							<div class="grid-content bg-purple">合计</div>
						</el-col>
						<el-col :span="7">
							<div class="grid-content bg-purple">购买数：{{formOne.totalDataObj.convert}}</div>
						</el-col>
						<el-col :span="7">
							<div class="grid-content bg-purple-light">抽水数：{{formOne.totalDataObj.pump}}</div>
						</el-col>
						<el-col :span="7">
							<div class="grid-content bg-purple-light">今日抽水数：{{formOne.totalDataObj.today_pump}}</div>
						</el-col>
					</el-row>
				</div>
				<template>
					<el-table 
					ref="tableOneHeight" 
					:data="formOne.tabData" 
					border fit highlight-current-row 
					style="width:100%;" 
					:height="tableOneHeight">
						<el-table-column prop="create_time" label="时间" min-width="50"></el-table-column>
						<el-table-column prop="pump" label="抽水数" min-width="50"></el-table-column>
						<el-table-column prop="game_number" label="游戏次数" min-width="50"></el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="oneHandleCurrentChange" 
						:page-size="20" 
						:total="formOne.totalpage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- 个人游戏数据 -->
			<el-tab-pane 
			label="个人游戏数据" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item label="uid">
							<el-input 
							v-model="formTwo.uid" 
							auto-complete="off"></el-input>
						</el-form-item>
						<el-form-item>
							<div class="block">
								<span class="registerTime">日期</span>
								<el-date-picker 
								v-model="formTwo.choiceDate" 
								type="datetimerange" 
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
				<div class="total_data" id="total_data">
					<el-row>
						<el-col :span="3">
							<div class="grid-content bg-purple">合计</div>
						</el-col>
						<el-col :span="7">
							<div class="grid-content bg-purple">购买金币数：{{formTwo.totalDataObj.dou_to_gold}}</div>
						</el-col>
						<el-col :span="7">
							<div class="grid-content bg-purple-light">兑换金币数：{{formTwo.totalDataObj.gold_to_dou}}</div>
						</el-col>
					</el-row>
				</div>
				<template>
					<el-table 
					ref="tableTwoHeight" 
					:data="formTwo.tabData" 
					border fit highlight-current-row 
					style="width:100%;" 
					:height="tableTwoHeight">
						<el-table-column prop="post_time" label="时间" min-width="80"></el-table-column>
						<el-table-column prop="doudou_uid" label="用户ID" min-width="80"></el-table-column>
						<el-table-column prop="count" label="数额" width="150"></el-table-column>
						<el-table-column prop="type" label="类型" min-width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.type==1">购买</p>
									<p v-else-if="scope.row.type==2">兑换</p>
								</div>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="twoHandleCurrentChange" 
						:page-size="20" 
						:total="formTwo.totalpage" 
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
import { allget, khserverget } from '../../api/api.js';
export default {
	data() {
		return {
			tabHeight: null,
			tableOneHeight: null, 
			tabSearchHeight: null,
			tableTwoHeight: null, 
			formOne: {
				room_id: '',
				choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				type: '',
				tabData: [],
				totalDataObj: {
					convert: 0,
					pump: 0,
					today_pump: 0,
				},
				totalpage: 1000, 
				page: 0, 
				star: '0',
				end: '20',
			},
			formTwo: {
				uid: '',
				choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				type: '',
				tabData: [], 
				totalDataObj: {
					gold_to_dou: 0,
					dou_to_gold: 0,
				},
				totalpage: 1000, 
				page: 0, 
				star: '0',
				end: '20',
			},
			tabActiveName: 'first',//房间游戏数据、个人游戏数据
			formLabelWidth: '130px', 
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
			_this.formOne.page = val-1;
			_this.formOne.star = (_this.formOne.page)*20;
			_this.formOne.end = _this.formOne.star+20;
		},
		twoHandleCurrentChange(val) {
			var _this = this;
			_this.formTwo.page = val-1;
			_this.formTwo.star = (_this.formTwo.page)*20;
			_this.formTwo.end = _this.formTwo.star+20;
		},
		// 获取房间游戏数据
		getTableFind() {
			var _this = this ;
			var url = '/HGame/getRoomGameData';
			// 请求的总和
			var params1 = {
				room_id: _this.formOne.room_id,
				begin_time:	baseConfig.changeDateTime(_this.formOne.choiceDate[0], 1),
				end_time: baseConfig.changeDateTime(_this.formOne.choiceDate[1], 1),
				type: 0,
			};
			axios.get(khserverget+url, { params: params1 })
				.then((res) => {
					if(res.data.code) {
						_this.formOne.totalDataObj.convert = res.data.data.convert; 
						_this.formOne.totalDataObj.pump = res.data.data.pump; 
						_this.formOne.totalDataObj.today_pump = res.data.data.today_pump; 
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
			// 请求的按日求和返回
			var params2 = {
				room_id: _this.formOne.room_id,
				begin_time:	baseConfig.changeDateTime(_this.formOne.choiceDate[0], 1),
				end_time: baseConfig.changeDateTime(_this.formOne.choiceDate[1], 1),
				type: 1,
			};

			axios.get(khserverget+url, { params: params2 })
				.then((res) => {
					if(res.data.code) {
						_this.formOne.tabData = res.data.data.data_list;
						_this.formOne.totalpage = res.data.data.data_list.length;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 个人游戏数据
		getTableManage() {
			var _this = this ;
			var url = '/HGame/getMyGameData';
			// 总和返回
			var params1 = {
				uid: _this.formTwo.uid,
				begin_time: baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 1),
				end_time: baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 1),
				type: 0,
			};
			axios.get(khserverget+url, { params: params1 })
				.then((res) => {
					if(res.data.code) {
						_this.formTwo.totalDataObj.gold_to_dou = res.data.data.gold_to_dou;
						_this.formTwo.totalDataObj.dou_to_gold = res.data.data.dou_to_gold;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
			// 流水返回
			var params2 = {
				uid: _this.formTwo.uid,
				begin_time: baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 1),
				end_time: baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 1),
				type: 1,
			};
			axios.get(khserverget+url, { params: params2 })
				.then((res) => {
					if(res.data.code) {
						_this.formTwo.tabData = res.data.data.data_list;
						_this.formTwo.totalpage = res.data.data.data_list.length;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
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
			_this.tabHeight = baseConfig.lineNumber(tabSearchHeight-38);
			_this.tableOneHeight = baseConfig.lineNumber(tabSearchPageHeight-38);
			_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight-38);
			_this.tableTwoHeight = baseConfig.lineNumber(tabSearchPageHeight-38);
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
/* 页面样式css内容 */
#total_data {
    width: 50%;
    height: 30px;
}
.el-row {
    /* margin-bottom: 20px; */
    z-index: 999;
    /* border: 1px solid red; */
    height: 0px;
}
/* .el-col {
    border-radius: 4px;
} */
.bg-purple-dark {
    background: #99a9bf;
}
.bg-purple {
    background: #d3dce6;
}
.bg-purple-light {
    background: #e5e9f2;
}
.grid-content {
    /* border-radius: 4px; */
    border: 1px solid #fff;
    min-height: 36px;
    line-height: 36px;
    text-align:center;
}
.row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
}
</style>
