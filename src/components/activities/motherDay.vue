<template>
    <el-tabs 
	v-model="activeName" 
	type="border-card" 
	@tab-click="handleClick">
   		<el-tab-pane 
		label="用户获取数量" 
		name="first" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<el-form 
				:inline="true" 
				:model="formOne">
					<el-form-item label="UID">
						<el-input 
						v-model="formOne.uid" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item>
						<el-button 
						type="primary" 
						@click="getOneData">查询</el-button>
					</el-form-item>
				</el-form>	
			</el-col>
			<div class="total_data" id="total_data">
				<el-row>
					<el-col :span="4">
						<div class="grid-content bg-purple">合计</div>
					</el-col>
					<el-col :span="5">
						<div class="grid-content bg-purple-light">五彩绳数量：{{formOne.totalDataObj.num_3}}</div>
					</el-col>
					<el-col :span="5">
						<div class="grid-content bg-purple-light">粽子数量：{{formOne.totalDataObj.num_2}}</div>
					</el-col>
					<el-col :span="5">
						<div class="grid-content bg-purple-light">龙舟数量：{{formOne.totalDataObj.num_1}}</div>
					</el-col>
					<el-col :span="5">
						<div class="grid-content bg-purple-light">香包数量：{{formOne.totalDataObj.num_4}}</div>
					</el-col>
				</el-row>
			</div>
   			<el-table 
			:data="formOne.tabData" 
			style="width:100%" 
			:height="tabSearchPageHeight">
			  	<el-table-column prop="uid" label="用户ID" min-width="100"></el-table-column>
		  		<el-table-column prop="nickname" label="用户昵称" min-width="100"></el-table-column>
		  		<el-table-column prop="num_3" label="五彩绳" min-width="100"></el-table-column>
		  		<el-table-column prop="num_2" label="粽子" min-width="100"></el-table-column>
		  		<el-table-column prop="num_1" label="龙舟" min-width="200"></el-table-column>
				  <el-table-column prop="num_4" label="香包" min-width="200"></el-table-column>
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
   		</el-tab-pane>
    	<el-tab-pane 
		label="用户获取详情" 
		name="second" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<el-form 
				:inline="true" 
				:model="formTwo">
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
					<el-form-item label="UID">
						<el-input 
						v-model="formTwo.uid" 
						auto-complete="off"></el-input>
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
			:height="tabSearchPageHeightOthers">
			  	<el-table-column prop="time" label="时间" min-width="100"></el-table-column>
		  		<el-table-column prop="gift_id" label="礼物ID" min-width="100"></el-table-column>
		  		<el-table-column prop="name" label="礼物名称" min-width="100"></el-table-column>
		  		<el-table-column prop="gift_num" label="礼物数量" min-width="100"></el-table-column>
				<el-table-column prop="give_uid" label="赠送用户信息用户ID" min-width="100"></el-table-column>
				<el-table-column prop="give_nickname" label="赠送用户信息用户昵称" min-width="100"></el-table-column>
				<el-table-column prop="uid" label="接收用户信息用户ID" min-width="100"></el-table-column>
				<el-table-column prop="nickname" label="接收用户信息用户昵称" min-width="100"></el-table-column>
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
				tabHeight: null,
				activeName:'first',
				tabSearchPageHeight:null,
				tabSearchPageHeightOthers:null,
           		dialogWidth: null,	
				activityList:[],
				formOne:{
					tabData:[],
					uid:'',
					page:1,
					totalPage:1000,
					totalDataObj:{
						num_1:'0',
						num_2:'0',
						num_3:'0',
						num_4:'0',
					}
				},
				formTwo:{
					tabData:[],
					choiceDate: [new Date()-30*24*60*60*1000, new Date()],
					uid:'',
					page: 1,
					totalPage:1000,
				},
			}
		},
		mounted(){
			var _this = this;
			_this.$nextTick(function() { 
		        _this.dialogWidth = lookWidth*0.8+'px'; //设置进行dialog的宽度进行设置为屏幕的80%
				_this.tabHeight = baseConfig.lineNumber(tabHeight);
				_this.tabPageHeight = baseConfig.lineNumber(tabPageHeight);
				_this.tabSearchPageHeight = baseConfig.lineNumber(tabSearchPageHeight-30);
				_this.tabSearchPageHeightOthers = baseConfig.lineNumber(tabSearchPageHeight);
				_this.getOneData();
				_this.getTwoData();
			})			
		},
		methods:{
			// 搜索条件
			searchConditionGift(){
				var _this = this;
				var obj = {};
				obj.uid =_this.formOne.uid;
				obj.page=_this.formOne.page;
				return obj; 
			},
			searchConditionData(){
				var _this = this;
				var obj = {};
				obj.uid = _this.formTwo.uid;
				obj.page=_this.formTwo.page;
				obj.start_date = _this.formTwo.choiceDate?baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0):'';
                obj.end_date = _this.formTwo.choiceDate?baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0):'';
				return obj; 
			},
			handleClick(tab, event){
				console.log('进行了盒子风格切换');
				// console.log(tab.name);
			},
			oneHandleCurrentChange(val){
				this.formOne.page=val;
				this.getOneData();
			},
			twoHandleCurrentChange(val){
				this.formTwo.page=val;
				this.getTwoData();
			},
			getOneData(){
       			var _this = this;
       			var params=_this.searchConditionGift();
				axios.get(wechatget+'/ydlManage/server/index.php/NewActivity/getMotherDayGiftList', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formOne.tabData = res.data.data.list;
						_this.formOne.totalDataObj.num_1 = res.data.data.total.num_1;
						_this.formOne.totalDataObj.num_2 = res.data.data.total.num_2;
						_this.formOne.totalDataObj.num_3 = res.data.data.total.num_3;
						_this.formOne.totalDataObj.num_4 = res.data.data.total.num_4;
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
				axios.get(wechatget+'/ydlManage/server/index.php/NewActivity/getMotherDayGiftRecordList', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formTwo.tabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
		}
	}
</script>

<style scoped>
#total_data {
    width: 100%;
    height: 30px;
	overflow: hidden;
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
.el-table .warning-row {
	background: oldlace;
}
.bg-purple-dark {
    background: #99a9bf;
}
.bg-purple {
    background: #d3dce6;
}
.bg-purple-light {
    background: #e5e9f2;
}
</style>