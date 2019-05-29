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
                <el-table-column prop="type" label="变更内容">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.type == 1">奖金池改次数</p>
							<p v-else-if="scope.row.type==2">次数改奖金池</p>
						</div>
					</template>
				</el-table-column>
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
				<div class="caseStatus">宝箱状态：{{caseStatus}}</div>
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
			  	<el-table-column prop="box" label="宝箱类型">
					  <template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.box == 1">黄金宝箱</p>
							<p v-else-if="scope.row.box==2">白金宝箱</p>
							<p v-else-if="scope.row.box==3">氪金宝箱</p>
						</div>
					</template>
				  </el-table-column>
				<el-table-column prop="threshold" label="阈值"></el-table-column>
				<el-table-column prop="count" label="次数"></el-table-column>
				<el-table-column prop="money" label="奖金池"></el-table-column>
				<el-table-column prop="income" label="收入"></el-table-column>
				<el-table-column prop="outgo" label="支出"></el-table-column>
                <el-table-column prop="type" label="当前大奖发放模式">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.type == 1">次数</p>
							<p v-else-if="scope.row.type==2">奖金池</p>
						</div>
					</template>
				</el-table-column>
				<!-- <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                                size="mini" 
                                type="primary" 
								v-if="status == 1"
                                @click="modify(scope.$index, scope.row)">修改</el-button>
							<el-button 
                                size="mini" 
                                type="info" 
								v-else-if="status == 0"
                                >修改</el-button>
                        </el-row>
                    </template>
                </el-table-column> -->
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
		<el-dialog title="修改宝箱状态" :visible.sync="formTwo.dialogFormVisible">
			<el-form :model="formTwo">
				<el-form-item label="宝箱">
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
		<el-dialog title="编辑发奖方式" :visible.sync="formTwo.dialogFormVisibleOther">
			<el-form :model="formTwo">
				<el-form-item label="大奖发放方式">
					<el-select v-model="formTwo.type">
						<el-option label="次数" value="1"></el-option>
						<el-option label="奖金池" value="2"></el-option>
					</el-select>
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
				status:'',
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
					box:'',
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
				axios.get(allget+'/NewTreasureBox/getPrizeLogList', {params: params})
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
				axios.get(allget+'/NewTreasureBox/getPrizeList', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formTwo.tabData = res.data.data.prize_list;
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
					axios.get(allget+'/NewTreasureBox/setStatus', {params: {status:_this.formTwo.status}})
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
			// modify(index, rows){
			// 	var _this = this;
			// 	_this.formTwo.dialogFormVisibleOther = true;
			// 	_this.formTwo.box = rows.box;
			// },
			// changeTypeSure(num){
			// 	var _this = this;
			// 	if(num == 0){
			// 		_this.formTwo.dialogFormVisibleOther = false;
			// 	}else{
			// 		var params = {
			// 			box : _this.formTwo.box,
			// 			type : _this.formTwo.type,
			// 			admin : store.state.user.name
			// 		};
			// 		axios.get(allget+'/NewTreasureBox/setPrizeType', {params: params})
			// 		.then((res) => {
			// 			if(res.data.ret == 1) {
			// 				_this.getTwoData();
			// 				_this.formTwo.dialogFormVisibleOther = false;
			// 				baseConfig.successTipMsg(_this, '编辑成功');
			// 			} else {
			// 				baseConfig.warningTipMsg(_this, res.data.msg);
			// 			}
			// 		})
			// 		.catch((err) => {
			// 			console.log(err);
			// 		})
			// 	}
			// }
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