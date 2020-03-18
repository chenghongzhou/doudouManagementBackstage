<template>
    <el-tabs 
	type="border-card" 
	@tab-click="handleClick">
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<div class="caseStatus">好运宝箱：{{caseStatus}}</div>
			<el-button 
			type="primary" 
			style="margin-bottom:10px;"
			@click="formTwo.dialogFormVisible = true">修改状态</el-button>
		</el-col>
		
		<el-table 
		:data="formTwo.tabData"
		style="width:100%"
		:style="{height:tabSearchPageHeightOthers+'px'}">
		<template>
            <el-table-column label="宝箱类型">
                <template slot-scope="scope">
                    <div slot="reference" class="name-wrapper">
                        <p v-if="scope.row.box == 1">黄金宝箱</p>
                        <p v-else-if="scope.row.box == 2">钻石宝箱</p>
                    </div>
                </template>
            </el-table-column>
			<el-table-column prop="level" label="级别"></el-table-column>
			<el-table-column prop="progress" label="奖金池当前金额"></el-table-column>
            <el-table-column prop="income" label="收入"></el-table-column>
            <el-table-column prop="outgo" label="支出"></el-table-column>
            <el-table-column prop="threshold_1" label="奖金池触发金额1"></el-table-column>
            <el-table-column prop="threshold_2" label="奖金池触发金额2"></el-table-column>
            <el-table-column prop="threshold_3" label="奖金池触发金额3"></el-table-column>
            <el-table-column prop="threshold_4" label="奖金池触发金额4"></el-table-column>
            <el-table-column prop="threshold_5" label="奖金池触发金额5"></el-table-column>
			<el-table-column label="操作">
				<template slot-scope="scope">
					<el-row>
						<el-button 
							size="mini" 
							type="primary" 
							@click="changeMoney(scope.$index, scope.row)">编辑</el-button>
					</el-row>
				</template>
			</el-table-column>
		</template>	  
		</el-table>
		<el-col :span="24" class="toolbar">
			<el-pagination 
				layout="total,prev,pager,next,jumper" 
				:page-size="20" 
				style="float:right;">
			</el-pagination>
		</el-col>
		<!-- 修改宝箱状态 -->
		<el-dialog title="修改状态" :visible.sync="formTwo.dialogFormVisible">
			<el-form :model="formTwo">
				<el-form-item label="好运宝箱">
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
				<label>奖金池触发金额1：</label>
                <el-input v-model="threshold_1" placeholder="奖金池触发金额1" style="width:200px;"></el-input>
                <el-button 
				type="primary" 
                style="margin:10px auto;"
				@click.native.prevent="iscfMoney(1,threshold_1)">确 定</el-button><br />

                <label>奖金池触发金额2：</label>
                <el-input v-model="threshold_2" placeholder="奖金池触发金额2" style="width:200px;"></el-input>
                <el-button 
				type="primary" 
                style="margin:10px auto;"
				@click.native.prevent="iscfMoney(2,threshold_2)">确 定</el-button><br />

                <label>奖金池触发金额3：</label>
                <el-input v-model="threshold_3" placeholder="奖金池触发金额3" style="width:200px;"></el-input>
                <el-button 
				type="primary" 
                style="margin:10px auto;"
				@click.native.prevent="iscfMoney(3,threshold_3)">确 定</el-button><br />

                <label>奖金池触发金额4：</label>
                <el-input v-model="threshold_4" placeholder="奖金池触发金额4" style="width:200px;"></el-input>
                <el-button 
				type="primary" 
                style="margin:10px auto;"
				@click.native.prevent="iscfMoney(4,threshold_4)">确 定</el-button><br />

                <label>奖金池触发金额5：</label>
                <el-input v-model="threshold_5" placeholder="奖金池触发金额5" style="width:200px;"></el-input>
                <el-button 
				type="primary" 
                style="margin:10px auto;"
				@click.native.prevent="iscfMoney(5,threshold_5)">确 定</el-button>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="iscfMoney(0,0)">关 闭</el-button>
				
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
				tabSearchPageHeight: null,
				tabSearchPageHeightOthers:null,
				activeName:'first',
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
                threshold_1:'',
                threshold_2:'',
                threshold_3:'',
                threshold_4:'',
                threshold_5:'',
                box:''
			}
		},
		mounted(){
			var _this = this;
			_this.$nextTick(function() {
		        _this.dialogWidth = lookWidth*0.8+'px'; //设置进行dialog的宽度进行设置为屏幕的80%
				_this.tabHeight = baseConfig.lineNumber(tabHeight-35);
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
			getTwoData(){
       			var _this = this;
       			var params=_this.searchConditionData();
				axios.get(allget+'/NewBox/getStat', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formTwo.tabData = res.data.data.box_list;
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
					axios.get(allget+'/NewBox/setStatus', {params: {status:_this.formTwo.status}})
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
                _this.threshold_1 = rows.threshold_1;
                _this.threshold_2 = rows.threshold_2;
                _this.threshold_3 = rows.threshold_3;
                _this.threshold_4 = rows.threshold_4;
                _this.threshold_5 = rows.threshold_5;
                _this.box = rows.box;
			},
			iscfMoney(type,num){
				let _this = this;
				if(num == 0){
					_this.cfMoneyMaks = false;
				}else{
                    var params = {
                        level:type,
                        box:_this.box,
                        threshold:num
                    };
					axios.get(allget+'/NewBox/setThreshold', {params: params})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							baseConfig.successTipMsg(_this, '修改成功');
							
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