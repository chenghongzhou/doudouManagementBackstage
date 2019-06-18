<template>
    <el-tabs 
	v-model="activeName" 
	type="border-card" 
	@tab-click="handleClick">
   		<el-tab-pane 
		label="修改日志" 
		name="first" 
		:style="{height:tabHeight+'px'}">
   			<el-table 
			:data="onePageTabData" 
			style="width:100%" 
			:height="tabSearchPageHeightOthers">
			<template>
			  	<el-table-column prop="time" label="修改时间"></el-table-column>
                <el-table-column prop="admin" label="操作人"></el-table-column>
				<el-table-column prop="old_threshold" label="旧的触发金额"></el-table-column>
				<el-table-column prop="new_threshold" label="新的触发金额"></el-table-column>
                
			</template>	  
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
		label="奖品发放设置" 
		name="second" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<div class="caseStatus">摇摇乐：{{caseStatus}}</div>
				<el-button 
				type="primary" 
				style="margin-bottom:10px;"
				@click="formTwo.dialogFormVisible = true">修改状态</el-button>
			</el-col>
   			<el-table 
			:data="formTwo.tabData" 
			style="width:100%" 
			:height="tabSearchPageHeight">
			  <template>
				<el-table-column prop="threshold" label="触发金额"></el-table-column>
				<el-table-column prop="income" label="收入"></el-table-column>
				<el-table-column prop="outgo" label="支出"></el-table-column>
				<el-table-column prop="progress" label="当前金额"></el-table-column>
				<el-table-column prop="name" label="奖品名称"></el-table-column>
				<el-table-column prop="chat_gold" label="奖品价格"></el-table-column>
                <el-table-column prop="status" label="状态">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.status == 0">关闭</p>
							<p v-else-if="scope.row.status==1">开启</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                                size="mini" 
                                type="primary" 
								v-if="status == 0"
                                @click="modify(scope.$index, scope.row)">修改</el-button>
							<el-button 
                                size="mini" 
                                type="info" 
								v-else-if="status == 1"
                                >修改</el-button>
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
    	</el-tab-pane>
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
		
		<!-- 编辑发奖方式 -->
		<el-dialog title="编辑" :visible.sync="formTwo.dialogFormVisibleOther">
			<el-form :model="formTwo">
				<el-form-item label="大奖触发金额" :label-width="formLabelWidth">
					<el-select v-model="formTwo.thresholdParmas" @change="getOthers">
						<el-option 
							:label="item.threshold" 
							:value="item.threshold"
							:key="index"
							v-for="(item,index) in getList">
							</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="派发礼物名称" :label-width="formLabelWidth" disabled>
					<el-input 
					style="width:250px"
					v-model="formTwo.name" 
					auto-complete="off" disabled></el-input>
				</el-form-item>
				<el-form-item label="礼物价值" :label-width="formLabelWidth" disabled>
					<el-input 
					style="width:250px"
					v-model="formTwo.chat_gold" 
					auto-complete="off" disabled></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="changeTypeSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="changeTypeSure(1)">确 定</el-button>
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
				formOne:{
					tabData:[],
					page:1,
					totalPage:1000,
					status:'1',
					star: '0',
                	end: '20'
				},
				formTwo:{
					tabData:[],
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
				_this.tabSearchPageHeightOthers = baseConfig.lineNumber(tabHeight-55);
				_this.getOneData();
				_this.getTwoData();
				_this.getAllPrize();
			})			
		},
		methods:{
			// 搜索条件
			searchConditionGift(){
				var _this = this;
				var obj = {};
				obj.page=_this.formOne.page;
				return obj; 
			},
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
			oneHandleCurrentChange(val){
				var _this = this;
				this.formOne.page=val-1;
				 _this.formOne.star = (_this.formOne.page)*20;
           		 _this.formOne.end = _this.formOne.star+20;
			},
			twoHandleCurrentChange(val){
				this.formTwo.page=val;
				this.getTwoData();
			},
			getOneData(){
       			var _this = this;
       			var params=_this.searchConditionGift();
				axios.get(allget+'/NewMoneyTree/getThresholdRecordList', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formOne.tabData = res.data.data;
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
				axios.get(allget+'/NewMoneyTree/getThreshold', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formTwo.tabData = [];
						_this.formTwo.tabData.push(res.data.data);
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
					axios.get(allget+'/NewMoneyTree/setStatus', {params: {status:_this.formTwo.status}})
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
			//获取大奖列表
			getAllPrize(){
				var _this = this;	
				axios.get(allget+'/NewMoneyTree/getThresholdList', {params:{}}).then((res) => {
					if(res.data.ret == 1){
						_this.getList = res.data.data;
					}else{
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
			},
			//获取对应的礼物名字和价格
			getOthers(){
				console.log(this.formTwo.thresholdParmas)
				var _this = this;
				_this.getList.forEach((item) => {
					if(item.threshold == _this.formTwo.thresholdParmas){
						console.log(item)
						_this.formTwo.chat_gold = item.chat_gold || '无';
						_this.formTwo.name = item.name || '0';
					}
				})
			},
			modify(index, rows){
				var _this = this;
				_this.formTwo.dialogFormVisibleOther = true;
				_this.formTwo.thresholdParmas = rows.threshold;
				_this.formTwo.chat_gold = rows.chat_gold;
				_this.formTwo.name = rows.name;
			},
			changeTypeSure(num){
				var _this = this;
				if(num == 0){
					_this.formTwo.dialogFormVisibleOther = false;
				}else{
					var params = {
						admin : store.state.user.name,
						threshold: _this.formTwo.thresholdParmas
					};
					axios.get(allget+'/NewMoneyTree/setThreshold', {params: params})
					.then((res) => {
						if(res.data.ret == 1) {
							_this.getTwoData();
							_this.formTwo.dialogFormVisibleOther = false;
							baseConfig.successTipMsg(_this, '编辑成功');
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