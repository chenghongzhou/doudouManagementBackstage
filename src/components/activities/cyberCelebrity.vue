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
					<el-form-item label="活动类型">
						<el-select v-model="formOne.type" placeholder="请选择的网红活动">
							<el-option 
								:label="item.name" 
								:value="item.code" 
								:key='index'
								v-for="(item,index) in activityList"></el-option> 
						</el-select>
					</el-form-item>
					<el-form-item label="礼物类型">
						<el-select v-model="formOne.gift_id" placeholder="全部">
							<el-option label="全部" value="0"></el-option>
							<el-option label="礼物一" value="1"></el-option>
							<el-option label="礼物二" value="2"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item>
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
					<el-form-item label="活动类型">
						<el-select v-model="formTwo.type" placeholder="全部">
							<el-option 
								:label="item.name" 
								:value="item.code" 
								:key='index'
								v-for="(item,index) in activityList"></el-option> 
						</el-select>
					</el-form-item>
					<el-form-item label="页面按钮">
						<el-select v-model="formTwo.page_id" placeholder="全部">
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
				tabHeight: null,
				activeName:'first',
				tabSearchPageHeight:null,
           		dialogWidth: null,	
				activityList:[],
				formOne:{
					tabData:[],
					type:'online_celebrity_2',
					gift_id_list: [],					
					gift_id: '0',
					page:0,
					totalPage:0,
				},
				formTwo:{
					tabData:[],
					type:'online_celebrity_2',
					page_id:'0',
					page: 0,
					totalPage:0,
				},
			}
		},
		mounted(){
			var _this = this;
			_this.$nextTick(function() { 
		        _this.dialogWidth = lookWidth*0.8+'px'; //设置进行dialog的宽度进行设置为屏幕的80%
				_this.tabHeight = baseConfig.lineNumber(tabHeight);
				_this.tabPageHeight = baseConfig.lineNumber(tabPageHeight);
				_this.tabSearchPageHeight = baseConfig.lineNumber(tabSearchPageHeight);
				_this.getActivity();
				_this.getOneData();
				_this.getTwoData();
			})			
		},
		methods:{
			// 搜索条件
			searchConditionGift(){
				var _this = this;
				var obj = {};
				obj.activity = _this.formOne.type;
				obj.gift_id = _this.formOne.gift_id;
				obj.page=_this.formOne.page;
				return obj; 
			},
			searchConditionData(){
				var _this = this;
				var obj = {};
				obj.activity = _this.formTwo.type;
				obj.page_id = _this.formTwo.page_id;
				obj.page=_this.formTwo.page;
				return obj; 
			},
			handleClick(tab, event){
				console.log('进行了盒子风格切换');
				// console.log(tab.name);
			},
			oneHandleCurrentChange(val){
				this.formOne.page=val-1;
				this.getOneData();
			},
			twoHandleCurrentChange(val){
				this.formTwo.page=val-1;
				this.getTwoData();
			},
			//获取活动
			getActivity(){
       			var _this = this;
				axios.get(wechatget+'/ydlManage/server/index.php/NewActivity/getOnlineCelebrityActivityList', {})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.activityList = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
			getOneData(){
       			var _this = this;
       			var params=_this.searchConditionGift();
				axios.get(wechatget+'/ydlManage/server/index.php/NewActivity/getGiftRecord', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						if(res.data.data.length>0){
							_this.formOne.tabData = res.data.data;
						 	_this.formOne.totalPage = res.data.data.length;
						}else{
							_this.formOne.tabData = []
						//	_this.formTwo.tabData =[];
						 	_this.formOne.totalPage = 0;
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
					if(res.data.ret == 1) {
						if(res.data.data.length>0){
							_this.formTwo.tabData = res.data.data;
							_this.formTwo.totalPage = res.data.data.length;
						}else{
							_this.formTwo.tabData =[];
							_this.formTwo.totalPage = 0;
						}
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

<style>
</style>