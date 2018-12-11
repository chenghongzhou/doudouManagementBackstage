<template>
    <el-tabs 
	v-model="activeName" 
	type="border-card" 
	@tab-click="handleClick">
   		<el-tab-pane 
		label="礼物" 
		name="first" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<el-form 
				:inline="true" 
				:model="formOne">
					<el-form-item label="礼物">
						<el-select v-model="formOne.type" placeholder="全部">
							<el-option label="全部" value="0"></el-option>
							<el-option label="活动礼物1" value="603"></el-option> 
							<el-option label="活动礼物2" value="604"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item>
						<el-button 
						type="primary" 
						@click="chartLineShow">导出</el-button>
						<el-button 
						type="primary" 
						@click="getOneData">查询</el-button>
					</el-form-item>
				</el-form>	
			</el-col>		
   			<el-table 
			:data="formOne.tabData" 
			style="width:100%" 
			:height="tabSearchPageHeight">
			  	<el-table-column prop="uid" label="用户ID" min-width="100"></el-table-column>
		  		<el-table-column prop="name" label="礼物" min-width="100"></el-table-column>
		  		<el-table-column prop="gift_num" label="个数" min-width="100"></el-table-column>
		  		<el-table-column prop="price" label="消耗豆币" min-width="100"></el-table-column>
		  		<el-table-column prop="time" label="时间" min-width="200"></el-table-column>
   			</el-table>  						
		    <el-col :span="24" class="toolbar">
				<el-pagination 
					layout="total,prev,pager,next,jumper" 
					@current-change="oneHandleCurrentChange" 
					:page-size="20" 
					:total="formOne.totalPage" 
					style="float:right;">
				</el-pagination>
			</el-col>
			<!-- 折线图 -->
			<el-dialog 
			title="折线图" 
			:width="dialogWidth"  
			:visible.sync="dialogVisible" 
			@open="show">
                <div class="chartLine" style="width:100%;height:600px;"></div>
            </el-dialog>
   		</el-tab-pane>
    	<el-tab-pane 
		label="页面按钮数据" 
		name="second" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<el-form 
				:inline="true" 
				:model="formTwo">
					<el-form-item label="页面按钮">
						<el-select v-model="formTwo.type" placeholder="全部">
							<el-option label="全部" value="0"></el-option>
							<el-option label="落地页" value="100"></el-option> 
							<el-option label="落地页-去围观" value="1"></el-option>					    
							<el-option label="落地页-撩一撩" value="2"></el-option>
							<el-option label="落地页-送礼物" value="3"></el-option> 
							<el-option label="落地页-去充值" value="4"></el-option>				    
							<el-option label="展示页" value="200"></el-option>
							<el-option label="展示页-语音" value="5"></el-option> 
							<el-option label="展示页-去围观" value="6"></el-option>					    
							<el-option label="展示页-撩一撩" value="7"></el-option> 
							<el-option label="展示页-送礼物" value="8"></el-option>
							<el-option label="展示页-去充值" value="9"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item>
						<el-button 
						type="primary" 
						@click="getTwoData">查询</el-button>
					</el-form-item>
				</el-form>
			</el-col>
   			<el-table 
			:data="formTwo.tabData" 
			style="width:100%" 
			:height="tabSearchPageHeight">
			  	<el-table-column prop="page" label="页面" min-width="100"></el-table-column>
		  		<el-table-column prop="uv" label="uv" min-width="100"></el-table-column>
		  		<el-table-column prop="pv" label="pv" min-width="100"></el-table-column>
		  		<el-table-column prop="date" label="停留时间" min-width="100"></el-table-column>
   			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
					layout="total,prev,pager,next,jumper" 
					@current-change="twoHandleCurrentChange" 
					:page-size="20" 
					:total="formTwo.totalPage" 
					style="float:right;">
				</el-pagination>
			</el-col>
    	</el-tab-pane>
    </el-tabs>
</template>

<script>
	import Event from './../../public_js/event.js';
	import { wechatget } from '../../api/api';
	import store from '../../vuex/store';
	import axios from 'axios';
	import echarts from 'echarts';
	export default{
		data(){
			return{
				dialogVisible:false,
				tabHeight: null,
				activeName:'first',
				tabSearchPageHeight:null,
           		dialogWidth: null,	
				formOne:{
					tabData:[],
					type:'',
					page:0,
					totalPage:0,
				},
				formTwo:{
					tabData:[],
					type:'',
					page: 0,
					totalPage:0,
				},
	            chartLineData: {
	                title: {
	                        text: '礼物统计'
	                    },
	                tooltip : {
	                    trigger: 'axis'
	                },
	                toolbox: {
	                    show : true,
	                    feature : {
	                        mark : {show: true},
	                        dataView : {show: true, readOnly: false},
	                        magicType: {show: true, type: ['line', 'bar']},
	                        restore : {show: true},
	                        saveAsImage : {show: true}
	                    }
	                },
	                calculable : true,
	                legend: {
	                    data:['消耗豆币数量']
	                },
	                xAxis: [
	                    {
	                        type: 'category',
	                        data: []
	                    }
	                ],
	                yAxis: [
	                    {
	                        type : 'value',
	                        name : '消耗豆币',
	                        axisLabel : {
	                            formatter: '{value} '
	                        }
	                    },
	                    {
	                        type : 'value',
	                        name : '消耗率',
	                        axisLabel : {
	                            formatter: '{value} %'
	                        }
	                    },
	                ],
	                series: [
	                    {
	                        name:'消耗豆币数量',
	                        type:'bar',
	                        data:[]
	                    },
	                    {
	                        name:'',
	                        type:'bar',
	                        data:[]
	                    },
	                    {
	                        name:'',
	                        type:'bar',
	                        data:[]
	                    },
	                    {
	                        name:'',
	                        type:'bar',
	                        data:[]
	                    }
	                ]
	            }
			}
		},
		mounted(){
			this.$nextTick(function() { 
		        this.dialogWidth = lookWidth*0.8+'px'; //设置进行dialog的宽度进行设置为屏幕的80%
				this.tabHeight = baseConfig.lineNumber(tabHeight);
				this.tabPageHeight = baseConfig.lineNumber(tabPageHeight);
				this.tabSearchPageHeight = baseConfig.lineNumber(tabSearchPageHeight);
				this.getOneData();
				this.getTwoData();
//				console.log(this.tabHeight,this.tabPageHeight,this.tabSearchPageHeight)
			})			
		},
		methods:{
			// 搜索条件
			searchConditionGift(){
				var _this = this;
				var obj = {};
				obj.activity ="online_celebrity";
				obj.gift_id = _this.formOne.type;
				obj.page=_this.formOne.page;
				return obj; 
			},
			searchConditionData(){
				var _this = this;
				var obj = {};
				obj.activity ="online_celebrity";
				obj.page_id = _this.formTwo.type;
				return obj; 
			},
			chartLineShow(){
	            this.dialogVisible=true;
			},
			handleClick(tab, event){
				// console.log('不进行处理');
			},
			oneHandleCurrentChange(val){
				formOne.page=val-1;
				this.getOneData();
			},
			twoHandleCurrentChange(val){
				formTwo.page=val-1;
				this.getTwoData();
			},
			getOneData(){
       			var _this = this;
       			var params=_this.searchConditionGift();
				axios.get(wechatget+'/ydlManage/server/index.php/NewActivity/getGiftRecord', {params: params})
				.then((res) => {
//					console.log(res.data.data.length)
					if(res.data.ret == 1) {
						if(res.data.data.length>0){
							_this.formOne.tabData = res.data.data;
						 	_this.formOne.totalPage = res.data.data.length;
                      	   // 根据获取数据，动态赋值折线图所需的数据
                           // 赋值前先清空
                           _this.chartLineData.xAxis[0].data = [];
                           _this.chartLineData.series[0].data = [];
                           _this.chartLineData.series[1].data = [];
                           _this.chartLineData.series[2].data = [];
                           _this.chartLineData.series[3].data = [];
                           for(var i = res.data.data.length-1; i >= 0; i--){
                               _this.chartLineData.xAxis[0].data.push(_this.formOne.tabData[i].time); // 横坐标
                               _this.chartLineData.series[0].data.push(_this.formOne.tabData[i].price); // 消耗都避暑
//                             _this.chartLineData.series[1].data.push(_this.formOne.tabData[i].active_ARPPU); // 活跃ARPU值
//                             _this.chartLineData.series[2].data.push(_this.formOne.tabData[i].consume_rate); // 日付费率
//                             _this.chartLineData.series[3].data.push(_this.formOne.tabData[i].cumulative_consume_rate); // 累计付费率
                           }
						}else{
							_this.formTwo.tabData =[];
							baseConfig.successTipMsg(_this, "暂无数据");
						}
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
			getTwoData(){
       			var _this = this;
       			var params=_this.searchConditionData();
				axios.get(wechatget+'/ydlManage/server/index.php/NewActivity/getPageView', {params: params})
				.then((res) => {
//					console.log(res)
					if(res.data.ret == 1) {
						if(res.data.data.length>0){
							_this.formTwo.tabData = res.data.data;
						}else{
							_this.formTwo.tabData =[];
							baseConfig.successTipMsg(_this, "暂无数据");
						}
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
	        show() {
	            this.$nextTick(function() {
	                this.chartLine = echarts.init(document.querySelector('.chartLine'));
	                document.querySelector('.chartLine').style.border = "1px solid #4488BB";
	                this.chartLine.setOption(this.chartLineData);
	            })
	        }
		}
	}
</script>

<style>
</style>