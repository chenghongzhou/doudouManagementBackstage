<template>
    <el-tabs 
	v-model="activeName" 
	type="border-card" 
	@tab-click="handleClick">
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<el-button 
			type="primary" 
			style="margin-bottom:10px;"
			@click="formTwo.dialogFormVisible = true">添加暴走</el-button>
		</el-col>
		<el-table 
		:data="formTwo.tabData" 
		style="width:100%" 
		:height="tabSearchPageHeightOthers">
			<template>
				<el-table-column prop="start_time" label="暴走开始时间"></el-table-column>
				<el-table-column prop="end_time" label="暴走结束时间"></el-table-column>
				<el-table-column prop="admin" label="操作人"></el-table-column>
				<el-table-column prop="weekday" label="暴走周期">
					<template slot-scope="scope">
						{{getWeek(scope.row.weekday)}}
					</template>
				</el-table-column>
			</template>	  
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
		<!-- 修改宝箱状态 -->
		<el-dialog title="修改状态" :visible.sync="formTwo.dialogFormVisible">
			<el-form :model="formTwo">
				<el-form-item label="打豆豆">
					<el-select v-model="formTwo.weekday" >
						<el-option label="周日" value="0"></el-option>
						<el-option label="周一" value="1"></el-option>
						<el-option label="周二" value="2"></el-option>
						<el-option label="周三" value="3"></el-option>
						<el-option label="周四" value="4"></el-option>
						<el-option label="周五" value="5"></el-option>
						<el-option label="周六" value="6"></el-option>
					</el-select>
				</el-form-item>
				<el-time-select
					v-model="formTwo.start_time"
					:picker-options="{
						start: '00:00',
						step: '00:60',
						end: '23:00'
					}"
					placeholder="开始时间">
				</el-time-select>
				<el-time-select
					v-model="formTwo.end_time"
					:picker-options="{
						start: '00:00',
						step: '00:60',
						end: '23:00'
					}"
					placeholder="结束时间">
				</el-time-select>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="changeStatusSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="changeStatusSure(1)">确 定</el-button>
			</div>
		</el-dialog>
		
    </el-tabs>
	
</template>

<script>
	import Event from './../../../public_js/event.js';
	import store from '../../../vuex/store';
	import axios from 'axios';
	import { allget } from '../../../api/api.js';
	import echarts from 'echarts';
	export default{
		data(){
			return{
				tabHeight: null,
				activeName:'first',
				tabSearchPageHeight:null,
				tabSearchPageHeightOthers:null,
           		dialogWidth: null,	
				caseStatus:'已关闭',
				formLabelWidth:'150px',
				status:'',
				getList:[],
				formTwo:{
					tabData:[],
					page: 1,
					weekday:'1',
					totalPage:1,
					dialogFormVisible:false,
					dialogFormVisibleOther:false,
					type:'1',
					chat_gold:'',
					name:'',
					thresholdParmas:'',
					end_time:'',
					start_time:'',
				},
			}
		},
		computed: {
			onePageTabData() {
				var _this = this;
				return _this.formOne.tabData.slice(_this.formOne.star, _this.formOne.end);
			},
		},
		mounted(){
			var _this = this;
			_this.$nextTick(function() { 
		        _this.dialogWidth = lookWidth*0.8+'px'; //设置进行dialog的宽度进行设置为屏幕的80%
				_this.tabHeight = baseConfig.lineNumber(tabHeight-35);
				_this.tabPageHeight = baseConfig.lineNumber(tabPageHeight);
				_this.tabSearchPageHeight = baseConfig.lineNumber(tabSearchPageHeight);
				_this.tabSearchPageHeightOthers = baseConfig.lineNumber(tabHeight-75);
				_this.getTwoData();
			})			
		},
		methods:{
			searchConditionData(){
				var _this = this;
				var obj = {};
				obj.page=_this.formTwo.page;
				return obj; 
			},
			handleClick(tab, event){
				console.log('进行了盒子风格切换');
				// console.log(tab.name);
			},
			twoHandleCurrentChange(val){
				this.formTwo.page=val;
				this.getTwoData();
			},
			getTwoData(){
       			var _this = this;
       			var params=_this.searchConditionData();
				axios.get(allget+'/NewBean/getModeList', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formTwo.tabData =res.data.data;
						//_this.formTwo.tabData.push(res.data.data);
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
			getWeek(num){
				var _this = this;
				switch (num) {
					case 0:
						return '周日';
					break;
					case 1:
						return '周一';
					break;
					case 2:
						return '周二';
					break;
					case 3:
						return '周三';
					break;
					case 4:
						return '周四';
					break;
					case 5:
						return '周五';
					break;
					case 6:
						return '周六';
					break;
				}
			},
			changeStatusSure(num){
				var _this = this;
				if(num == 0){
					_this.formTwo.dialogFormVisible = false;
				}else{
					let params = {
						weekday: _this.formTwo.weekday,
						start_time: _this.formTwo.start_time,
						end_time: _this.formTwo.end_time,
						admin: store.state.user.name
					}
					axios.get(allget+'/NewBean/setMode', {params: params})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							baseConfig.successTipMsg(_this, '添加成功');
							_this.formTwo.dialogFormVisible = false;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					})
				}
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
.caseStatus{
	line-height: 40px;
	font-size: 16px;
	margin-left: 20px;
	display: inline-block; 
	margin-right: 10px;
}
</style>