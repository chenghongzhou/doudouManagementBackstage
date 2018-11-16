<template>
	<section>
		<el-tabs 
		v-model="activeName" 
		type="border-card"
		@tab-click="handleClick">
		    <el-tab-pane 
			label="收益数据" 
			name="first" 
			:style="{height:tabHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formOne">
						<el-form-item label="日期">
							<el-date-picker 
							v-model="formOne.choiceDate" 
							type="daterange" 
							range-separator=" 至 " 
							placeholder="选择日期范围"></el-date-picker>
						</el-form-item>
						<el-form-item label="上级UID">
							<el-input v-model="formOne.bind_uid" auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item label="下级UID">
							<el-input v-model="formOne.uid" auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button type="primary" @click="getOneData">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<el-table 
				:data="formOne.tabData" 
				style="width:100%" 
				ref="tabSearchPageHeight" 
				:height="tabSearchPageHeight">
				  	<el-table-column prop="" label="日期" min-width="200"></el-table-column>
			  		<el-table-column prop="uid" label="UID" min-width="200"></el-table-column>
			  		<el-table-column prop="nickname" label="昵称" min-width="200"></el-table-column>
			  		<el-table-column prop="total" label="总收益(聊票)" min-width="200"></el-table-column>
			  		<el-table-column prop="bind_uid" label="上级UID" min-width="200"></el-table-column>
			  		<el-table-column prop="" label="上级昵称" min-width="200"></el-table-column>
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
			label="通话数据" 
			name="second" 
			:style="{height:tabHeight+'px'}">
				<el-col 
				:span="24" 
				class="toolbar" 
				style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item label="日期">
							<el-date-picker 
							v-model="formTwo.choiceDate" 
							type="daterange" 
							range-separator=" 至 " 
							placeholder="选择日期范围"></el-date-picker>
						</el-form-item>
						<el-form-item label="上级UID">
							<el-input v-model="formTwo.bind_uid" auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item label="下级UID">
							<el-input v-model="formTwo.uid" auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button type="primary" @click="getTwoData">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<el-table 
				:data="formTwo.tabData" 
				style="width:100%" 
				ref="tabSearchPageHeight" 
				:height="tabSearchPageHeight">
				  	<el-table-column prop="" label="日期" min-width="200"></el-table-column>
			  		<el-table-column prop="uid" label="UID" min-width="200"></el-table-column>
			  		<el-table-column prop="nickname" label="昵称" min-width="200"></el-table-column>
			  		<el-table-column prop="num" label="有效电话数" min-width="200"></el-table-column>
			  		<el-table-column prop="bind_uid" label="上级UID" min-width="200"></el-table-column>
			  		<el-table-column prop="" label="上级昵称" min-width="200"></el-table-column>
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
	</section>
</template>

<script>
	import Event from './../../public_js/event.js';
	import { wechatget } from '../../api/api';
	import store from '../../vuex/store';
	import axios from 'axios';
	export default{
		data(){
			return{
				tabHeight: null,
				tabPageHeight: null,
				tabSearchPageHeight: null,	
				activeName:'first',
				formOne:{
					choiceDate:[new Date()-30*24*60*60*1000, new Date()],
					bind_uid:'',
					uid:'',
					totalPage:0,
					tabData:[],
					page:0
				},
				formTwo:{
					choiceDate:[new Date(), new Date()],
					bind_uid:'',
					uid:'',
					totalPage:0,
					tabData:[],
					page:0
				}
			}
		},
		mounted(){
			this.$nextTick(function() { 
				this.tabHeight = baseConfig.lineNumber(tabHeight);
				this.tabPageHeight = baseConfig.lineNumber(tabPageHeight);
				this.tabSearchPageHeight = baseConfig.lineNumber(tabSearchPageHeight);
				this.getOneData();
				this.getTwoData();
			})			
		},
		methods:{
			// 搜索条件
			searchConditionEarning() {
				var _this = this;
				var obj = {};
				obj.uid = _this.formOne.uid;
				obj.bind_uid = _this.formOne.bind_uid;
				obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
				obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
				obj.page=_this.formOne.page;
				return obj; 
			},
			searchConditionCall() {
				var _this = this;
				var obj = {};
				obj.uid = _this.formTwo.uid;
				obj.bind_uid = _this.formTwo.bind_uid;
				obj.date_s = baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0);
				obj.date_e = baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0);
				obj.page=_this.formTwo.page;
				return obj; 
			},
			oneHandleCurrentChange(val){
				formOne.page=val-1;
				this.getOneData();
			},
			twoHandleCurrentChange(val){
				formTwo.page=val-1;
				this.getTwoData();
			},
	        handleClick(tab, event) {
				console.log(tab);
       		},
       		getOneData(){
       			var _this = this;
				var params=_this.searchConditionEarning();
				if(params.bind_uid==''&&params.uid=='') {
					baseConfig.warningTipMsg(_this, '上下级uid至少输入一个进行搜索~');
					return;
				}   
				axios.get(
					wechatget+'/server/index.php/Agent/getInviteManageMemberGainData', 
					{params: params}
				)
					.then((res) => {
						if(res.data.code==1) {
							if(res.data.data.length>0){
								_this.formOne.tabData = res.data.data;
								_this.formOne.totalPage = res.data.data.length;
							}else{
								baseConfig.successTipMsg(_this, "暂无数据");
							}
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			},
			getTwoData(){
       			var _this = this;
				var params=_this.searchConditionCall();
				if(params.bind_uid==''&&params.uid=='') {
					baseConfig.warningTipMsg(_this, '上下级uid至少输入一个进行搜索~');
					return;
				}   
				axios.get(wechatget+'/server/index.php/Agent/getInviteManageMemberCallData', {params: params})
					.then((res) => {
						if(res.data.code==1) {
							if(res.data.data.length>0){
								_this.formTwo.tabData = res.data.data;
								_this.formTwo.totalPage = res.data.data.length;
							}else{
								baseConfig.successTipMsg(_this, "暂无数据");
							}
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		}
	}
</script>

<style scoped="scoped">
</style>