<template>
    <el-tabs 
	v-model="activeName" 
	type="border-card" 
	@tab-click="handleClick">
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<div class="caseStatus">砸金蛋：{{caseStatus}}</div>
			<el-button 
			type="primary" 
			style="margin-bottom:10px;"
			@click="formTwo.dialogFormVisible = true">修改状态</el-button>
			<div class="caseStatus">总支出:{{formTwo.tabData.outgo}}</div>
			<div class="caseStatus">总收入:{{formTwo.tabData.income}}</div>
		</el-col>
		
		<el-table 
		:data="formTwo.tabData.stat_list" 
		style="width:100%" 
		:height="tabSearchPageHeightOthers">
			<template>
			<el-table-column prop="level" label="级别"></el-table-column>
			<el-table-column prop="threshold" label="奖金池触发金额"></el-table-column>
			<el-table-column prop="progress" label="奖金池当前金额"></el-table-column>
			<el-table-column label="操作">
				<template slot-scope="scope">
					<el-row>
						<el-button 
							size="mini" 
							type="primary" 
							@click="changeMoney(scope.$index, scope.row)">编辑</el-button>
						<el-button 
							size="mini" 
							type="primary" 
							@click="deletePress(scope.$index, scope.row)">删除奖金池进度</el-button>
					</el-row>
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
				<el-form-item label="摇摇乐">
					<el-select v-model="formTwo.status">
						<el-option label="开启" value="1"></el-option>
						<el-option label="关闭" value="0"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="changeStatusSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="changeStatusSure(1)">确 定</el-button>
			</div>
		</el-dialog>
		<!-- 修改宝箱状态 -->
		<el-dialog title="修改触发金额" :visible.sync="cfMoneyMaks">
			<el-form>
				<el-input v-model="cfMoney" placeholder="输入修改触发金额"></el-input>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="iscfMoney(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="iscfMoney(1)">确 定</el-button>
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
					tabData:{},
					page: 1,
					status:'1',
					totalPage:1,
					dialogFormVisible:false,
					dialogFormVisibleOther:false,
					type:'1',
					chat_gold:'',
					name:'',
					thresholdParmas:''
				},
				cfMoneyMaks:false,
				cfMoney:'',
				level:'',
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
				axios.get(allget+'/NewEgg/getStat', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formTwo.tabData = res.data.data;
					
						_this.status = res.data.data.status;
						if(res.data.data.status == 0){
							_this.caseStatus = '已关闭';
						}else{
							_this.caseStatus = '已开启';
						}
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
			changeStatusSure(num){
				var _this = this;
				if(num == 0){
					_this.formTwo.dialogFormVisible = false;
				}else{
					axios.get(allget+'/NewEgg/setStatus', {params: {status:_this.formTwo.status}})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							baseConfig.successTipMsg(_this, '修改成功');
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
			changeMoney(index, rows){
				let _this = this;
				_this.cfMoneyMaks = true;
				_this.level = rows.level;
			},
			iscfMoney(num){
				let _this = this;
				if(num == 0){
					_this.cfMoneyMaks = false;
				}else{
					axios.get(allget+'/NewEgg/setThreshold', {params: {level:_this.level,threshold:_this.cfMoney}})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							baseConfig.successTipMsg(_this, '修改成功');
							_this.cfMoneyMaks = false;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					})
				}
			},
			deletePress(index, rows){
				let _this = this;
				axios.get(allget+'/NewEgg/deleteProgress', {params: {level:rows.level}})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							baseConfig.successTipMsg(_this, '删除成功成功');
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					})
			}
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