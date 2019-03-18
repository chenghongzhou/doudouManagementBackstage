<template>
	<!-- 娃娃机-->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 系统赠送次数统计 -->
			<el-tab-pane 
			label="系统赠送次数统计" 
			name="first" 
			:style="{height:tabSearchHeight+'px'}">
			<el-button 
			type="primary" 
			style="margin:10px 20px;"
			@click.native.prevent="handleDownloadOne">导出</el-button>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="formOne.TabData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;"
					:height="tableHeight">
						<el-table-column prop="date" label="日期" width="200"></el-table-column>
						<el-table-column prop="buy_num" label="系统充值赠送游戏币个数" min-width="100"></el-table-column>
						<el-table-column prop="login_num" label="系统登陆赠送游戏币个数" min-width="100"></el-table-column>
                        <el-table-column prop="chat_gold" label="送出豆币总数" min-width="100"></el-table-column>
                        <el-table-column prop="chat_ticket" label="送出礼物总价值" min-width="100"></el-table-column>
                        <el-table-column prop="play_num" label="总次数" min-width="100"></el-table-column>
                        <el-table-column prop="give_num" label="用户转增张数" min-width="100"></el-table-column>
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
			label="收益数据统计" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
			<el-button 
			type="primary" 
			style="margin:10px 20px;"
			@click.native.prevent="handleDownloadTwo">导出</el-button>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="formTwo.TabDataPrize" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="date" label="日期" width="200"></el-table-column>
						<el-table-column prop="with_num" label="页面访问次数" min-width="100"></el-table-column>
						<el-table-column prop="user_num" label="活动参与人数" min-width="100"></el-table-column>
                        <el-table-column prop="buy_people" label="购买抽奖券人数" min-width="100"></el-table-column>
                        <el-table-column prop="buy_num" label="购买游戏币个数" min-width="100"></el-table-column>
                        <el-table-column prop="buy_chat_gold" label="游戏花费豆币币数" min-width="100"></el-table-column>
                        <el-table-column prop="chat_gold" label="送出豆币总数" min-width="100"></el-table-column>
                        <el-table-column prop="chat_ticket" label="送出礼物总价值" min-width="100"></el-table-column>
                        <el-table-column prop="give_num" label="用户转增张数" min-width="100"></el-table-column>
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
			<!-- 每日礼物送出数统计 -->
			<el-tab-pane 
			label="收益数据统计" 
			name="three" 
			:style="{height:tabSearchHeight+'px'}">
			<el-button 
			type="primary" 
			style="margin:10px 20px;"
			@click.native.prevent="handleDownload">导出</el-button>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="threeTwo.TabData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;"
					:height="tableHeight">
						<el-table-column prop="date" label="日期" width="200"></el-table-column>
						<el-table-column prop="count_1" label="5豆币"></el-table-column>
						<el-table-column prop="count_2" label="11豆币"></el-table-column>
                        <el-table-column prop="count_3" label="15豆币"></el-table-column>
                        <el-table-column prop="count_4" label="20豆币"></el-table-column>
                        <el-table-column prop="count_5" label="40豆币"></el-table-column>
                        <el-table-column prop="count_6" label="凤凰火x5"></el-table-column>
                        <el-table-column prop="count_7" label="包邮x3"></el-table-column>
                        <el-table-column prop="count_8" label="包邮x5"></el-table-column>
                        <el-table-column prop="count_9" label="单身狗x5"></el-table-column>
                        <el-table-column prop="count_10" label="吸血姬x3"></el-table-column>
                        <el-table-column prop="count_11" label="夜时雨x5"></el-table-column>
                        <el-table-column prop="count_12" label="太空帽x5"></el-table-column>
                        <el-table-column prop="count_13" label="寿喜锅x5"></el-table-column>
                        <el-table-column prop="count_14" label="戏精x5"></el-table-column>
                        <el-table-column prop="count_15" label="海之味x3"></el-table-column>
						<el-table-column prop="count_16" label="灵剑山x3"></el-table-column>
                        <el-table-column prop="count_17" label="灵剑山x5"></el-table-column>
                        <el-table-column prop="count_18" label="灵剑山x7"></el-table-column>
						<el-table-column prop="count_19" label="乌帽子x5"></el-table-column>
						<el-table-column prop="count_20" label="酒之烈x5"></el-table-column>
                        <el-table-column prop="count_21" label="粉红河马"></el-table-column>
                        <el-table-column prop="count_22" label="羊驼x3"></el-table-column>
						<el-table-column prop="count_23" label="羊驼x5"></el-table-column>
						<el-table-column prop="count_24" label="蚁人2x5"></el-table-column>
						<el-table-column prop="count_25" label="灰兔"></el-table-column>
						<el-table-column prop="count_26" label="小白兔"></el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="threeHandleCurrentChange" 
						:page-size="20" 
						:total="threeTwo.TotalPage" 
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
			tableOneHeight: null,
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
            threeTwo: {
				uid: '',
				prize: '',
				TabData: [],
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
			spanIsShow:true,
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
		threeHandleCurrentChange(){
			var _this = this;
            _this.threeTwo.Page = val-1;
            _this.getTableActivity();
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
		// 系统赠送
		getTableManage() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getDollMachineData';
			var params = {
                type: 1
            };
			axios.get(allget+url, { params: params })
                .then((res) => {
                    _this.listLoading = false;	
                    if(res.data.ret) {
                        _this.formOne.TabData = res.data.data;
                        _this.formOne.TotalPage = res.data.data.length;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg); 
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
		},
		// 在列表中进行搜索条件
		searchConditionPrize() {
			var _this = this;
			var obj = {};
			obj.uid = _this.formTwo.uid;
			obj.Page = _this.formTwo.Page;
			return obj;
        },
        //收益数据统计
		getTablePrize(){
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getDollMachineData';
			var params = {
                type: 2
            };
			 axios.get(allget+url, { params: params })
                .then((res) => {
                    _this.listLoading = false;	
                    if(res.data.ret) {
                        _this.formTwo.TabDataPrize = res.data.data;
                        _this.formTwo.TotalPage = res.data.data.length;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg); 
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
		},
		//数据统计
		getTableActivity(){
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getDollMachineData';
			var params = {
                type: 3
            };
			 axios.get(allget+url, { params: params })
                .then((res) => {
                    _this.listLoading = false;	
                    if(res.data.ret) {
                        var arr = [];
							for(var i=0;i<res.data.data.length;i++){
								var obj = {};
								for(var key in res.data.data[i]){
									obj.date = res.data.data[i].date;
									obj.count_1 = res.data.data[i]['5豆币x1']?res.data.data[i]['5豆币x1']:'';
									obj.count_2 = res.data.data[i]['11豆币x1']?res.data.data[i]['11豆币x1']:'';
									obj.count_3 = res.data.data[i]['15豆币x1']?res.data.data[i]['15豆币x1']:'';
									obj.count_4 = res.data.data[i]['20豆币x1']?res.data.data[i]['20豆币x1']:'';
									obj.count_5 = res.data.data[i]['40豆币x1']?res.data.data[i]['40豆币x1']:'';
									obj.count_6 = res.data.data[i]['凤凰火x5']?res.data.data[i]['凤凰火x5']:'';
									obj.count_7 = res.data.data[i]['包邮x3']?res.data.data[i]['包邮x3']:'';
									obj.count_8 = res.data.data[i]['包邮x5']?res.data.data[i]['包邮x5']:'';
									obj.count_9 = res.data.data[i]['单身狗x5']?res.data.data[i]['单身狗x5']:'';
									obj.count_10 = res.data.data[i]['吸血姬x3']?res.data.data[i]['吸血姬x3']:'';
									obj.count_11 = res.data.data[i]['夜时雨x5']?res.data.data[i]['夜时雨x5']:'';
									obj.count_12 = res.data.data[i]['太空帽x5']?res.data.data[i]['太空帽x5']:'';
									obj.count_13 = res.data.data[i]['寿喜锅x5']?res.data.data[i]['寿喜锅x5']:'';
									obj.count_14 = res.data.data[i]['戏精x5']?res.data.data[i]['戏精x5']:'';
									obj.count_15 = res.data.data[i]['海之味x3']?res.data.data[i]['海之味x3']:'';
									obj.count_16 = res.data.data[i]['灵剑山x3']?res.data.data[i]['灵剑山x3']:'';
									obj.count_17 = res.data.data[i]['灵剑山x5']?res.data.data[i]['灵剑山x5']:'';
									obj.count_18 = res.data.data[i]['灵剑山x7']?res.data.data[i]['灵剑山x7']:'';
									obj.count_19 = res.data.data[i]['乌帽子x5']?res.data.data[i]['乌帽子x5']:'';
									obj.count_20 = res.data.data[i]['酒之烈x5']?res.data.data[i]['酒之烈x5']:'';
									obj.count_21 = res.data.data[i]['粉红河马x1']?res.data.data[i]['粉红河马x1']:'';
									obj.count_22 = res.data.data[i]['羊驼x3']?res.data.data[i]['羊驼x3']:'';
									obj.count_23 = res.data.data[i]['羊驼x5']?res.data.data[i]['羊驼x5']:'';
									obj.count_24 = res.data.data[i]['蚁人2x5']?res.data.data[i]['蚁人2x5']:'';
									obj.count_25 = res.data.data[i]['灰兔x1']?res.data.data[i]['灰兔x1']:'';
									obj.count_26 = res.data.data[i]['小白兔x1']?res.data.data[i]['小白兔x1']:'';
								}

							 	arr.push(obj);
							}
							_this.threeTwo.TotalPage = arr.length;
							_this.threeTwo.TabData = arr;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg); 
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
		},
		//导出
        handleDownload() {
			require.ensure([], () => {
			const { export_json_to_excel } = require('../vendor/Export2Excel');
			const tHeader = ['时间','5豆币','11豆币','15豆币','20豆币','40豆币','凤凰火x5','包邮x3','包邮x5','单身狗x5','吸血姬x3',
								'夜时雨x5','太空帽x5','寿喜锅x5','戏精x5','海之味x3','灵剑山x3','灵剑山x5','灵剑山x7','乌帽子x5','酒之烈x5','粉红河马',
								'羊驼x3','羊驼x5','蚁人2x5','灰兔','小白兔'
							];
			const filterVal = ['date','count_1','count_2','count_3','count_4','count_5','count_6','count_7','count_8','count_9','count_10',
								'count_11','count_12','count_13','count_14','count_15','count_16','count_17','count_18','count_19','count_20','count_21',
								'count_22','count_23','count_24','count_25','count_26'
							];
			const data = this.formatJson(filterVal, this.threeTwo.TabData);
			export_json_to_excel(tHeader, data, 'excel表');
            })
		},
		handleDownloadOne() {
			require.ensure([], () => {
			const { export_json_to_excel } = require('../vendor/Export2Excel');
			const tHeader = ['时间','系统充值赠送游戏币个数','系统登陆赠送游戏币个数','送出豆币总数','送出礼物总价值','总次数','用户转增张数'];
			const filterVal = ['date','buy_num','login_num','chat_gold','chat_ticket','play_num','give_num'];
			const data = this.formatJson(filterVal, this.formOne.TabData);
			export_json_to_excel(tHeader, data, 'excel表');
            })
		},
		handleDownloadTwo() {
			require.ensure([], () => {
			const { export_json_to_excel } = require('../vendor/Export2Excel');
			const tHeader = ['时间','页面访问次数','活动参与人数','购买抽奖券人数','购买游戏币个数','游戏花费豆币币数','送出豆币总数','送出礼物总价值','用户转增张数'];
			const filterVal = ['date','with_num','user_num','buy_people','buy_num','buy_chat_gold','chat_gold','chat_ticket','give_num'];
			const data = this.formatJson(filterVal, this.formTwo.TabDataPrize);
			export_json_to_excel(tHeader, data, 'excel表');
            })
		},
		formatJson(filterVal, jsonData) {
			return jsonData.map(v => filterVal.map(j => {
			if (j === 'timestamp') {
				return parseTime(v[j])
			} else {
				return v[j]
			}
			}))
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
				_this.tabHeight = baseConfig.lineNumber(tabHeight);
				_this.tableOneHeight = baseConfig.lineNumber(tabPageHeight);
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
