<template>
    <el-tabs 
	v-model="activeName" 
	type="border-card" 
	@tab-click="handleClick">
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<div class="caseStatus">状态：{{caseStatus}}</div>
			<el-button 
			type="primary" 
			style="margin-bottom:10px;"
			@click="formTwo.dialogFormVisible = true">修改状态</el-button>
			<div class="caseStatus">比例：{{poolAllot}}</div>
			<el-button 
			type="primary" 
			style="margin-bottom:10px;"
			@click="rat.dialogFormVisible = true">修改比例</el-button>
		</el-col>
		
		<el-table 
		:data="formTwo.tabData" 
		style="width:100%" 
		:height="tabSearchPageHeightOthers">
			<template>
			<el-table-column prop="level" label="级别"></el-table-column>
			<el-table-column prop="threshold" label="奖金池触发金额"></el-table-column>
			<el-table-column prop="progress" label="奖金池当前金额"></el-table-column>
            <el-table-column prop="rate" label="奖金池比例"></el-table-column>
            <el-table-column label="正在进行" width="80">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.status == 1">进行中</p>
							<p v-else-if="scope.row.status==0">--</p>
						</div>
					</template>
				</el-table-column>
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
		<el-dialog title="编辑" :visible.sync="edit.dialogFormVisible">
			<el-form>
				<el-input v-model="edit.threshold" placeholder="金额" style="width:50%"></el-input>
                <el-input v-model="edit.rate" placeholder="比例" style="margin-top:20px;width:50%"></el-input>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="handleEdit(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="handleEdit(1)">确 定</el-button>
			</div>
		</el-dialog>

        <el-dialog title="修改比例" :visible.sync="rat.dialogFormVisible">
			<el-form>
				<el-input v-model="rat.rat_num1" placeholder="修改比例" style="width:30%"></el-input>：<el-input v-model="rat.rat_num2" placeholder="修改比例" style="width:30%"></el-input>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="handleChangeRat(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="handleChangeRat(1)">确 定</el-button>
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
                rat:{
                    dialogFormVisible: false,
                    rat_num2:'',
                    rat_num2:''
                },
				cfMoneyMaks:false,
				cfMoney:'',
				level:'',
                poolAllot:'',
                edit:{
                    dialogFormVisible: false,
                    threshold:'',
                    rate:'',
                    id:''
                }
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
				axios.get(allget+'/NewTwistEgg/getPoolThresholdList', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formTwo.tabData = res.data.data.pool_threshold_list;
					
						_this.status = res.data.data.status;
                        _this.poolAllot = res.data.data.pool_allot;
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
            handleChangeRat(index){
                var _this = this;
				if(index == 0){
					_this.rat.dialogFormVisible = false;
				}else{
                    var params = {
                        "rate1":_this.rat.rat_num1,
                        "rate2":_this.rat.rat_num2,
                    }
                    if(_this.rat.rat_num2 == '' || _this.rat.rat_num1 == ''){
                        baseConfig.warningTipMsg(_this, '不能为空');
                        return false;
                    }
					axios.get(allget+'/NewTwistEgg/setPoolRate', {params: params})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							baseConfig.successTipMsg(_this, '修改成功');
							_this.rat.dialogFormVisible = false;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					})
				}
            },
			changeStatusSure(num){
				var _this = this;
				if(num == 0){
					_this.formTwo.dialogFormVisible = false;
				}else{
					axios.get(allget+'/NewTwistEgg/setStatus', {params: {status:_this.formTwo.status,type:3}})
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
				_this.edit.dialogFormVisible = true;
				_this.edit.threshold = rows.threshold;
                _this.edit.rate = rows.rate;
                _this.edit.id = rows.id;
			},
			handleEdit(num){
				let _this = this;
				if(num == 0){
					_this.edit.dialogFormVisible = false;
				}else{
                    var params = {
                        threshold:_this.edit.threshold,
                        rate:_this.edit.rate,
                        id:_this.edit.id,
                    }
					axios.get(allget+'/NewTwistEgg/setPoolThreshold', {params: params})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							baseConfig.successTipMsg(_this, '修改成功');
							_this.edit.dialogFormVisible = false;
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