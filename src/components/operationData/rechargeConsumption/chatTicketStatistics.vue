<template>
	<!-- 豆票数据统计 -->
	<section>
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<el-form 
			:inline="true" 
			style="overflow:hidden;" 
			:model="formOne">
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
					<span>展示类型</span>
					<el-select v-model="formOne.show_type">
						<el-option label="按天数" value="0"></el-option>
						<el-option label="按月份" value="1"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item>
					<el-button 
					type="primary" 
					@click="chartPieShowOne">产出占比图</el-button>					
					<el-button 
					type="primary" 
					@click="chartPieShowTwo">消耗占比图</el-button>					
					<el-button 
					type="primary" 
					@click="getTableData">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
            <el-table 
			ref="tableHeight" 
			:data="expendData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
                <el-table-column prop="date" label="日期" width="100"></el-table-column>
                <el-table-column label="豆票产出">
                    <el-table-column prop="one" label="通话获得" width="80"></el-table-column>
                    <el-table-column prop="two" label="礼物获得" width="80"></el-table-column>
                    <el-table-column prop="three" label="后台赠送" width="80"></el-table-column>
                    <el-table-column prop="four" label="系统赠送" width="80"></el-table-column>
                    <el-table-column prop="five" label="活动获得" min-width="80"></el-table-column>
                    <el-table-column prop="six" label="被偷听获得" min-width="80"></el-table-column>
                    <el-table-column prop="seven" label="被解锁用户信息获得" min-width="80"></el-table-column>
                    <el-table-column prop="eight" label="私密通话获得" min-width="80"></el-table-column>
                    <el-table-column prop="nine" label="切换关门通话获得" width="80"></el-table-column>
                    <el-table-column prop="one_zero" label="打赏录音获得" width="80"></el-table-column>
                    <el-table-column prop="one_one" label="偷听录音获得" width="80"></el-table-column>
                    <el-table-column prop="one_two" label="微信分享获得" width="80"></el-table-column>
                    <el-table-column prop="one_three" label="随机通话加速获得" min-width="80"></el-table-column>
                    <el-table-column prop="one_four" label="代理获得" min-width="80"></el-table-column>
                    <el-table-column prop="one_five" label="发消息消费（豆票）获得" min-width="80"></el-table-column>
                    <el-table-column prop="one_six" label="代理活动获得" min-width="80"></el-table-column>
                    <el-table-column prop="one_seven" label="家族内礼物获得" min-width="80"></el-table-column>
                    <el-table-column prop="one_eight" label="家族内抢礼物红包获得" min-width="80"></el-table-column>
                    <el-table-column prop="one_nine" label="一对一转盘获得" min-width="80"></el-table-column>
                    <el-table-column prop="two_zero" label="直播间内礼物获得" min-width="80"></el-table-column>
                    <el-table-column prop="two_one" label="直播间内抢礼物红包获得" min-width="80"></el-table-column>
                    <el-table-column prop="two_two" label="平台奖励获得" min-width="80"></el-table-column>
                    <el-table-column prop="two_three" label="好友赠送道具获得" min-width="80"></el-table-column>
                    <el-table-column prop="two_four" label="系统赠送道具获得" width="80"></el-table-column>                    
                </el-table-column>
                <el-table-column label="豆票消耗">
                    <el-table-column prop="two_five" label="提现" min-width="80"></el-table-column>
                    <el-table-column prop="two_six" label="兑换" min-width="80"></el-table-column>
                    <el-table-column prop="two_seven" label="系统扣除" min-width="80"></el-table-column>
                </el-table-column>
            </el-table>
		</template>
		<!-- 产出占比图 -->
		<el-dialog 
		title="饼状图" 
		:width="dialogChartPieOne.dialogWidth" 
		:visible.sync="dialogChartPieOne.dialogVisible" 
		@open="showOne()" 
		size="large">
            <div class="chartPieOne" style="width:100%;height:600px;"></div>
            <p style="color:red;font-size:20px;font-family:'微软雅黑';">总量数值为：{{dialogChartPieOne.total_num}}</p>
		</el-dialog>
		<!-- 消耗占比图 -->
		<el-dialog 
		title="饼状图" 
		:width="dialogChartPieTwo.dialogWidth" 
		:visible.sync="dialogChartPieTwo.dialogVisible" 
		@open="showTwo()" 
		size="large">
            <div class="chartPieTwo" style="width:100%;height:600px;"></div>
            <p style="color:red;font-size:20px;font-family:'微软雅黑';">总量数值为：{{dialogChartPieTwo.total_num}}</p>
		</el-dialog>
	</section>
</template>

<script>
import echarts from 'echarts';
import Event from './../../../public_js/event.js';
import { allget } from '../../../api/api';
import store from '../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null, 
			formOne: {
				choiceDate: [new Date()-7*24*60*60*1000, new Date()], // 对应选择的日期,给默认时间180之前到现在
                type: '2',//1为豆币，2为豆票，当前是豆币的页面
                channel: '',//渠道号
                show_type: '0'//显示方式0->日，1->月
			},
			listLoading: false, //动画加载时显示的动画
			tabData: [], //列表的所有数据，移除删除的功能用全部的数据进行移除
			expendData: [], //支付的具体详情
			formLabelWidth: '120px',
			chartPieDataOne: [],
			chartPieDataTwo: [],
			dialogChartPieOne: {
				dialogVisible: false,
				chartPie: null,
				chartData: null,
				dialogWidth: '',
				total_num: '11111',
			},
			dialogChartPieTwo: {
				dialogVisible: false, 
				chartPie: null,
				chartData: null,
				dialogWidth: '',
				total_num: '11111',
			},
		};
	},
	methods: {
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
            obj.type = _this.formOne.type;
            obj.show_type = _this.formOne.show_type;
            if(_this.formOne.channel!='') {
                obj.channel = _this.formOne.channel;
            }
			return obj; 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewMoney/getChatGoldChatTicket';
			var params = _this.searchCondition();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						// table数据格式的处理开始
						// ********************
						var arrName = [
							'one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine',
							'one_zero', 'one_one', 'one_two', 'one_three', 'one_four', 'one_five', 'one_six', 'one_seven', 'one_eight', 'one_nine',
							'two_zero', 'two_one', 'two_two', 'two_three', 'two_four', 'two_five', 'two_six', 'two_seven',
						];
						var arrParam = [];//判断条件的数组
						var arrDate = [];//日期的数组
						if(params.show_type=='0') {//按日进行显示
							arrDate = _this.getDate(baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0), baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0));
						} else if(params.show_type=='1') {//按月进行显示
							arrDate = _this.getDateMonth(baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0), baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0));
						}
						// 组装成专门的日期、加上数据的结构
						for(var l=0; l<arrDate.length; l++) {
							arrDate[l] = {
								date: arrDate[l],
								arr: [
                                        // 豆票获取
                                        { type: '6', goldTicket: 2, name: '通话获得', num: '0', },
                                        { type: '7', goldTicket: 2, name: '礼物获得', num: '0', },
                                        { type: '11', goldTicket: 12, name: '后台赠送', num: '0', },
                                        { type: '12', goldTicket: 12, name: '系统赠送', num: '0', },
                                        { type: '13', goldTicket: 12, name: '活动获得', num: '0', },
                                        { type: '14', goldTicket: 2, name: '被偷听获得', num: '0', },
                                        { type: '17', goldTicket: 2, name: '被解锁用户信息获得', num: '0', },
                                        { type: '20', goldTicket: 2, name: '私密通话获得', num: '0', },
                                        { type: '23', goldTicket: 2, name: '切换关门通话获得', num: '0', },
                                        { type: '26', goldTicket: 2, name: '打赏录音获得', num: '0', },
                                        { type: '28', goldTicket: 2, name: '偷听录音获得', num: '0', },
                                        { type: '29', goldTicket: 2, name: '微信分享获得', num: '0', },
                                        { type: '30', goldTicket: 2, name: '随机通话加速获得', num: '0', },
                                        { type: '31', goldTicket: 2, name: '代理获得', num: '0', },
                                        { type: '35', goldTicket: 2, name: '发消息消费（豆票）获得', num: '0', },
                                        { type: '36', goldTicket: 2, name: '代理活动获得', num: '0', },
                                        { type: '39', goldTicket: 2, name: '家族内礼物获得', num: '0', },
                                        { type: '42', goldTicket: 2, name: '家族内抢礼物红包获得', num: '0', },
                                        { type: '47', goldTicket: 2, name: '一对一转盘获得', num: '0', },
                                        { type: '49', goldTicket: 2, name: '直播间内礼物获得', num: '0', },
                                        { type: '51', goldTicket: 2, name: '直播间内抢礼物红包获得', num: '0', },
                                        { type: '53', goldTicket: 2, name: '平台奖励获得', num: '0', },
                                        { type: '56', goldTicket: 2, name: '好友赠送道具获得', num: '0', },
                                        { type: '57', goldTicket: 2, name: '系统赠送道具获得', num: '0', },                                                                                
                                        // 豆票消耗
                                        { type: '9', goldTicket: 2, name: '提现', num: '0', },                                        
                                        { type: '8', goldTicket: 2, name: '兑换', num: '0', },
                                        { type: '62', goldTicket: 12, name: '系统扣除', num: '0', },
									],
							};
						}
						// 将相应的数据进行组装展示所需要的数据
                        for(var i=0; i<res.data.data.chat_ticket_in.length; i++) {
							for(var m=0; m<arrDate.length; m++) {
								if(res.data.data.chat_ticket_in[i].date==arrDate[m].date) {
									for(var n=0; n<arrDate[m].arr.length; n++) {
										if(res.data.data.chat_ticket_in[i].trade_type==arrDate[m].arr[n].type) {
                                            if(arrDate[m].arr[n].type=='9') {
                                                // 提现返回不进行处理
                                            } else {
                                                arrDate[m].arr[n].num = res.data.data.chat_ticket_in[i].total;
                                            }
											break;
										}
									}
								}
							}
                        }
                        for(var i=0; i<res.data.data.chat_ticket_out.length; i++) {
							for(var m=0; m<arrDate.length; m++) {
								if(res.data.data.chat_ticket_out[i].date==arrDate[m].date) {
									for(var n=0; n<arrDate[m].arr.length; n++) {
										if(res.data.data.chat_ticket_out[i].trade_type==arrDate[m].arr[n].type) {
											arrDate[m].arr[n].num = res.data.data.chat_ticket_out[i].total;
											break;
										}
									}
								}
							}
                        }
						var finalData = [];
						for(var w=0; w<arrDate.length; w++) {
							finalData.push({
								date: arrDate[w].date,
							});
							for(var v=0; v<arrDate[w].arr.length; v++) {
								finalData[w][arrName[v]] = arrDate[w].arr[v].num;
							}
						}
						_this.expendData = finalData;
						// ********************
						// table数据格式的处理结束
						var pie_chat_ticket_in_total_list = [];
						var pie_chat_ticket_in_total_name = [];
						var pie_chat_ticket_out_total_list = [];
						var pie_chat_ticket_out_total_name = [];
						var tatal_list = [
                            // 豆票产出
							{ type: '6', goldTicket: 2, name: '通话获得', },
                            { type: '7', goldTicket: 2, name: '礼物获得', },
                            { type: '11', goldTicket: 12, name: '后台赠送', },
                            { type: '12', goldTicket: 12, name: '系统赠送', },
                            { type: '13', goldTicket: 12, name: '活动获得', },
                            { type: '14', goldTicket: 2, name: '被偷听获得', },
                            { type: '17', goldTicket: 2, name: '被解锁用户信息获得', },
                            { type: '20', goldTicket: 2, name: '私密通话获得', },
                            { type: '23', goldTicket: 2, name: '切换关门通话获得', },
                            { type: '26', goldTicket: 2, name: '打赏录音获得', },
                            { type: '28', goldTicket: 2, name: '偷听录音获得', },
                            { type: '29', goldTicket: 2, name: '微信分享获得', },
                            { type: '30', goldTicket: 2, name: '随机通话加速获得', },
                            { type: '31', goldTicket: 2, name: '代理获得', },
                            { type: '35', goldTicket: 2, name: '发消息消费（豆票）获得', },
                            { type: '36', goldTicket: 2, name: '代理活动获得', },
                            { type: '39', goldTicket: 2, name: '家族内礼物获得', },
                            { type: '42', goldTicket: 2, name: '家族内抢礼物红包获得', },
                            { type: '47', goldTicket: 2, name: '一对一转盘获得', },
                            { type: '49', goldTicket: 2, name: '直播间内礼物获得', },
                            { type: '51', goldTicket: 2, name: '直播间内抢礼物红包获得', },
                            { type: '53', goldTicket: 2, name: '平台奖励获得', },
                            { type: '56', goldTicket: 2, name: '好友赠送道具获得', },
                            { type: '57', goldTicket: 2, name: '系统赠送道具获得', },                                                        
                            // 豆票消耗
                            { type: '9', goldTicket: 2, name: '提现', },                            
                            { type: '8', goldTicket: 2, name: '兑换', },
                            { type: '62', goldTicket: 12, name: '系统扣除', },
                        ];
                        var nineNum = '0';
						for(var u=0; u<res.data.data.chat_ticket_in_total_list.length; u++) {
							for(var v=0; v<tatal_list.length; v++) {
                                // 在豆票产出中，提现返回的不另外计入
                                if(res.data.data.chat_ticket_in_total_list[u].trade_type==tatal_list[v].type) {
                                    if(tatal_list[v].type=='9') {
                                        nineNum = res.data.data.chat_ticket_in_total_list[u].total;
                                    } else {
                                        pie_chat_ticket_in_total_name.push(tatal_list[v].name);
                                        pie_chat_ticket_in_total_list.push({
                                            value: res.data.data.chat_ticket_in_total_list[u].total,
                                            name: tatal_list[v].name,
                                        });
                                    }
                                }                                    
							}
						}
						for(var u=0; u<res.data.data.chat_ticket_out_total_list.length; u++) {
							for(var v=0; v<tatal_list.length; v++) {
								if(res.data.data.chat_ticket_out_total_list[u].trade_type==tatal_list[v].type) {
									pie_chat_ticket_out_total_name.push(tatal_list[v].name);
									pie_chat_ticket_out_total_list.push({
										value: res.data.data.chat_ticket_out_total_list[u].total,
										name: tatal_list[v].name,
									});
								}
							}
						}
                        // 豆票产出的饼状图所需要的数据
						_this.chartPieDataOne = {
							title: baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0)+'至'+baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0)+'豆票产出占比饼状图',
							sub_title: '1、悬浮查看；2、左边可点击选取特定几项对比。',
							content: '具体豆票数及其比例',
							name: pie_chat_ticket_in_total_name,
							data: pie_chat_ticket_in_total_list,
							total: res.data.data.chat_ticket_in_total-0-nineNum,
						};
						// 豆票消耗的饼状图所需要的数据
						_this.chartPieDataTwo = {
							title: baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0)+'至'+baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0)+'豆票消耗占比饼状图',
							sub_title: '1、悬浮查看；2、左边可点击选取特定几项对比。',
							content: '具体豆票数及其比例',
							name: pie_chat_ticket_out_total_name,
							data: pie_chat_ticket_out_total_list,
							total: res.data.data.chat_ticket_out_total,							
						};
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
		// 饼状图展示
		chartPieShowOne() {
			var _this = this;
			_this.dialogChartPieOne.dialogVisible=true;
		},
		chartPieShowTwo() {
			var _this = this;
			_this.dialogChartPieTwo.dialogVisible=true;
		},
		showOne() {
			var _this = this;
			_this.$nextTick(function() {
				_this.dialogChartPieOne.chartPie = echarts.init(document.getElementsByClassName('chartPieOne')[0]);
				var arrData = _this.chartPieDataOne;
                _this.dialogChartPieOne.total_num = arrData.total;
				_this.dialogChartPieOne.chartPie.setOption({
					title: {
						text: arrData.title,
						subtext: arrData.sub_title,
						x: 'left',
					},
					tooltip: {
						trigger: 'item',
						formatter: "{a} <br/>{b} : {c} ({d}%)",
					},
					legend: {
						orient: 'vertical',
						left: 'right',
						data: arrData.name,
					},
					series: [{
						name: arrData.content,
						type: 'pie',
						radius: '70%',
						center: ['50%', '60%'],
						data: arrData.data,
						itemStyle: {
							normal:{
								label:{
									show: true,
									formatter: '{b} : {c} \n ({d}%)',
								},
								labelLine:{
									show: true,
								}
							},
							emphasis: {
								shadowBlur: 10,
								shadowOffsetX: 0,
								shadowColor: 'rgba(0, 0, 0, 0.5)'
							},
						},
					}],
				});
			});	
		},
		showTwo() {
			var _this = this;
			_this.$nextTick(function() {
				_this.dialogChartPieTwo.chartPie = echarts.init(document.getElementsByClassName('chartPieTwo')[0]);
				var arrData = _this.chartPieDataTwo;
				_this.dialogChartPieTwo.total_num = arrData.total;
				_this.dialogChartPieTwo.chartPie.setOption({
					title: {
						text: arrData.title,
						subtext: arrData.sub_title,
						x: 'left',
					},
					tooltip: {
						trigger: 'item',
						formatter: "{a} <br/>{b} : {c} ({d}%)",
					},
					legend: {
						orient: 'vertical',
						left: 'right',
						data: arrData.name,
					},
					series: [{
						name: arrData.content,
						type: 'pie',
						radius: '70%',
						center: ['50%', '60%'],
						data: arrData.data,
						itemStyle: {
							normal:{
								label:{
									show: true,
									formatter: '{b} : {c} \n ({d}%)',
								},
								labelLine:{
									show: true,
								}
							},
							emphasis: {
								shadowBlur: 10,
								shadowOffsetX: 0,
								shadowColor: 'rgba(0, 0, 0, 0.5)'
							},
						},
					}],
				});
			});	
		},
		// 获得两个相邻日期的中间日期
		getDate(day1, day2) {
			var getDate = function(str) {
				var tempDate = new Date();  
				var list = str.split("-");  
				tempDate.setFullYear(list[0]);  
				tempDate.setMonth(list[1] - 1);  
				tempDate.setDate(list[2]);  
				return tempDate;  
			}  
			var date1 = getDate(day1);  
			var date2 = getDate(day2);  
			if (date1 > date2) {  
				var tempDate = date1;  
				date1 = date2;  
				date2 = tempDate;  
			}  
			date1.setDate(date1.getDate() + 1);  
			var dateArr = [];  
			var i = 0;  
			while (!(date1.getFullYear() == date2.getFullYear()  
					&& date1.getMonth() == date2.getMonth() && date1.getDate() == date2  
					.getDate())) {  
				var dayStr = date1.getDate().toString();  
				if(dayStr.length==1){  
					dayStr = "0"+dayStr;  
				}  
				var monthStr = (date1.getMonth()-0+1).toString();
				if(monthStr.length==1){  
					monthStr = "0"+monthStr;  
				}  
				dateArr[i] = date1.getFullYear() + "-" + monthStr + "-" + dayStr;  
				i++;  
				date1.setDate(date1.getDate() + 1);  
			}  
			dateArr.splice(0,0,day1)
			dateArr.push(day2);
			return dateArr;  
		},
		// 获得两个日期的之间的所有的月份
		getDateMonth(day1, day2) {
			var getDate = function(str) {
				var tempDate = new Date();  
				var list = str.split("-");  
				tempDate.setFullYear(list[0]);  
				tempDate.setMonth(list[1] - 1);  
				tempDate.setDate(list[2]);  
				return tempDate;  
			}  
			var date1 = getDate(day1);  
			var date2 = getDate(day2);  
			if (date1 > date2) {  
				var tempDate = date1;  
				date1 = date2;  
				date2 = tempDate;  
			}  
			date1.setDate(date1.getDate() + 1);  
			var dateArr = [];  
			var i = 0;  
			while (!(date1.getFullYear() == date2.getFullYear()  
					&& date1.getMonth() == date2.getMonth() && date1.getDate() == date2  
					.getDate())) {  
				var dayStr = date1.getDate().toString();  
				if(dayStr.length==1){  
					dayStr = "0"+dayStr;  
				}  
				var monthStr = (date1.getMonth()-0+1).toString();
				if(monthStr.length==1){  
					monthStr = "0"+monthStr;  
				}  
				dateArr[i] = date1.getFullYear() + "-" + monthStr;  
				i++;  
				date1.setDate(date1.getDate() + 1);  
			}  
			dateArr.splice(0,0,day1.substring(0, 7));
			dateArr.push(day2.substring(0, 7));
			// 在进行去重,es6
			dateArr = [...new Set(dateArr)];
			return dateArr;  
		},
	},
	mounted() {
		var _this = this;
		_this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(searchHeight);
			_this.getTableData();
		})
		_this.dialogChartPieOne.dialogWidth = lookWidth*0.8+'px';
		_this.dialogChartPieTwo.dialogWidth = lookWidth*0.8+'px';
	}
};
</script>

<style lang="css">
.el-table thead.is-group tr:nth-of-type(1) th:nth-of-type(1){
    background: yellow !important;

}
.el-table thead.is-group tr:nth-of-type(1) th:nth-of-type(1) div{
    background: yellow !important;

}
.el-table thead.is-group tr:nth-of-type(1) th:nth-of-type(2){
    background: #a2e9b1 !important;
}
.el-table thead.is-group tr:nth-of-type(1) th:nth-of-type(2) div{
    background: #a2e9b1 !important;    
}

.el-table thead.is-group tr:nth-of-type(1) th:nth-of-type(3){
    background: #bef0ff !important;
}
.el-table thead.is-group tr:nth-of-type(1) th:nth-of-type(3) div{
    background: #bef0ff !important;    
}
</style>
