<template>
    <el-tabs 
	v-model="activeName" 
	type="border-card" 
	@tab-click="handleClick">
   		<el-tab-pane 
		label="整体数据" 
		name="first" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<el-form 
				:inline="true" 
				:model="formOne">
                    <el-form-item>
                        <span>宝箱等级：</span>
                        <el-select style="width:200px;" v-model="formOne.box">
                            <el-option label="黄金宝箱" value="1"></el-option>
                            <el-option label="钻石宝箱" value="2"></el-option>
                        </el-select>
                    </el-form-item>
					<el-form-item>
						<div class="block">
							<span class="registerTime">日期</span>
							<el-date-picker 
							v-model="formOne.choiceDate" 
							type="daterange" 
							range-separator=" 至 " 
							placeholder="选择日期范围"></el-date-picker>
						</div>
					</el-form-item>
					<el-form-item>
						<el-button 
						type="primary" 
						@click="getOneData">查询</el-button>
					</el-form-item>
				</el-form>	
			</el-col>
   			<el-table 
			:data="onePageTabData" 
			style="width:100%" 
			:height="tabSearchPageHeightOthers">
			<template v-for="(item, index) in formOne.tabOneTitleData">
			  	<el-table-column 
				  :prop="item.identification" 
				  :label="item.name"
				  :key="index"
				  min-width="100"
				  ></el-table-column>
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
		label="个人数据" 
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
                        <span>宝箱等级：</span>
                        <el-select style="width:200px;" v-model="formTwo.box">
                            <el-option label="黄金宝箱" value="1"></el-option>
                            <el-option label="钻石宝箱" value="2"></el-option>
                        </el-select>
                    </el-form-item>
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
			:data="TwoPageTabData" 
			style="width:100%" 
			:height="tabSearchPageHeightOthers">
			  <template v-for="(item, index) in formTwo.tabOneTitleData">
			  	<el-table-column 
				  :prop="item.identification" 
				  :label="item.name"
				  :key="index"
				  min-width="100"
				  ></el-table-column>
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
		<el-tab-pane 
		label="作弊后台" 
		name="three" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<el-form 
				:inline="true" 
				:model="formThree">
					<el-form-item>
						<el-button 
						type="primary" 
						@click="formThree.dialogFormVisibleOther = true">添加</el-button>
					</el-form-item>
				</el-form>
			</el-col>
   			<el-table 
			:data="formThree.tabData" 
			style="width:100%" 
			:height="tabSearchPageHeightOthers">
			  	<el-table-column prop="uid" label="用户DID"></el-table-column>
				<el-table-column prop="name" label="作弊礼物"></el-table-column>
				<el-table-column prop="chat_gold" label="作弊豆币价值"></el-table-column>
				<el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                                size="mini" 
                                type="danger"
                                @click="deleteNoReal(scope.$index, scope.row)">删除</el-button>
                        </el-row>
                    </template>
                </el-table-column>
   			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
					layout="total,prev,pager,next,jumper" 
					@current-change="threeHandleCurrentChange" 
					:page-size="20" 
					:total="formThree.totalPage" 
					style="float:right;">
				</el-pagination>
			</el-col>
    	</el-tab-pane>
		<el-tab-pane 
		label="作弊操作日志" 
		name="four" 
		:style="{height:tabPageHeight+'px'}">
   			<el-table 
			:data="formFour.tabData" 
			style="width:100%" 
			:height="tabPageHeight">
			  	<el-table-column prop="time" label="操作时间"></el-table-column>
				<el-table-column prop="admin" label="操作人"></el-table-column>
				<el-table-column prop="cheat_id" label="作弊id"></el-table-column>
				<el-table-column label="作弊对象">
					<template slot-scope="scope" prop="uid">
						向{{scope.row.uid}}用户发送了奖励
					</template>
				</el-table-column>
   			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
					layout="total,prev,pager,next,jumper" 
					@current-change="fourHandleCurrentChange" 
					:page-size="20" 
					:total="formFour.totalPage" 
					style="float:right;">
				</el-pagination>
			</el-col>
    	</el-tab-pane>
		<!-- <el-tab-pane 
		label="黑名单" 
		name="five" 
		:style="{height:tabHeight+'px'}">
			<el-col 
			:span="24" 
			class="toolbar" 
			style="padding-bottom:0px;">
				<el-form 
				:inline="true" 
				:model="formFive">
					<el-form-item>
						<el-button 
						type="primary" 
						@click="formFive.dialogFormVisibleOther = true">添加</el-button>
					</el-form-item>
				</el-form>
			</el-col>
   			<el-table 
			:data="formFive.tabData" 
			style="width:100%" 
			:height="tabSearchPageHeightOthers">
			  	<el-table-column prop="uid" label="用户DID"></el-table-column>
				<el-table-column prop="admin" label="操作人"></el-table-column>
				<el-table-column prop="time" label="过期时间"></el-table-column>
				<el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-row>
                            <el-button 
                                size="mini" 
                                type="danger"
                                @click="deleteBliackList(scope.$index, scope.row)">删除</el-button>
                        </el-row>
                    </template>
                </el-table-column>
   			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
					layout="total,prev,pager,next,jumper" 
					@current-change="fiveHandleCurrentChange" 
					:page-size="20" 
					:total="formThree.totalPage" 
					style="float:right;">
				</el-pagination>
			</el-col>
    	</el-tab-pane> -->
		<!-- 作弊后台添加 -->
		<el-dialog title="作弊后台添加" :visible.sync="formThree.dialogFormVisibleOther">
			<el-form :model="formThree">
				<el-form-item label="作弊礼物" :label-width="formLabelWidth">
					<el-select v-model="formThree.prize_id">
						<el-option :label="item.name" :value="item.prize_id"
						:key="index"
						v-for="(item,index) in getPrize"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="用户ID" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="formThree.uid" 
					auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="addNoReal(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="addNoReal(1)">确 定</el-button>
			</div>
		</el-dialog>
		<!-- 黑名单添加 -->
		<el-dialog title="黑名单添加" :visible.sync="formFive.dialogFormVisibleOther">
			<el-form :model="formFive">
				<el-form-item label="用户ID" :label-width="formLabelWidth">
					<el-input 
					style="width:250px"
					v-model="formFive.uid" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="过期时间" :label-width="formLabelWidth">
						<el-date-picker 
						v-model="formFive.time" 
						type="datetime" 
						placeholder="选择日期范围" style="width:250px"></el-date-picker>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="addBlackList(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="addBlackList(1)">确 定</el-button>
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
				formLabelWidth: "120px",
				tabPageHeight:null,
           		dialogWidth: null,	
				activityList:[],
                box:'1',
				formOne:{
					tabData:[],
					tabOneTitleData:[],
					uid:'',
					page:0,
					box:'1',
					star: '0',
                	end: '20',
					totalPage:1000,
					choiceDate: [new Date()-180*24*60*60*1000, new Date()], // 对应选择的日期,给默认时间180之前到现在
					totalDataObj:{
						num_1:'0',
						num_2:'0',
						num_3:'0',
						num_4:'0',
					}
				},
				formTwo:{
					tabData:[],
					tabOneTitleDat:[],
					choiceDate: [new Date()-30*24*60*60*1000, new Date()],
					uid:'',
					page: 0,
					star: '0',
					box:'1',
                	end: '20',
					totalPage:1000,
				},
				formThree:{
					tabData:[],
					page: 1,
					totalPage:1000,
					dialogFormVisibleOther:false,
					box:'',
					prize_id:'',
					uid:''
				},
				formFour:{
					tabData:[],
					page: 1,
					totalPage:1000,
				},
				formFive:{
					tabData:[],
					page: 1,
					totalPage:1000,
					dialogFormVisibleOther:false,
					uid:'',
					time:'',
				},
				getPrize:[],
			}
		},
		computed: {
			onePageTabData() {
				var _this = this;
				return _this.formOne.tabData.slice(_this.formOne.star, _this.formOne.end);
			},
			TwoPageTabData() {
				var _this = this;
				return _this.formTwo.tabData.slice(_this.formTwo.star, _this.formTwo.end);
			},
		},
		mounted(){
			var _this = this;
			_this.$nextTick(function() { 
		        _this.dialogWidth = lookWidth*0.8+'px'; //设置进行dialog的宽度进行设置为屏幕的80%
				_this.tabHeight = baseConfig.lineNumber(tabHeight);
				_this.tabPageHeight = baseConfig.lineNumber(tabHeight-55);
				_this.tabSearchPageHeight = baseConfig.lineNumber(tabSearchPageHeight-30);
				_this.tabSearchPageHeightOthers = baseConfig.lineNumber(tabSearchPageHeight);
				_this.getOneData();
				_this.getTwoData();
				_this.getThreeData();
				_this.getFourData();
			//	_this.getFiveData();
				_this.checkSort();
			})			
		},
		methods:{
			// 搜索条件
			searchConditionGift(){
				var _this = this;
				var obj = {};
				obj.page=_this.formOne.page;
                obj.box=_this.formOne.box;
				obj.start_date = _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0):'';
                obj.end_date = _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0):'';
				return obj; 
			},
			searchConditionData(){
				var _this = this;
				var obj = {};
				obj.uid = _this.formTwo.uid;
				obj.page=_this.formTwo.page;
                obj.box=_this.formTwo.box;
				obj.start_date = _this.formTwo.choiceDate?baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0):'';
                obj.end_date = _this.formTwo.choiceDate?baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0):'';
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
				//this.getOneData();
			},
			twoHandleCurrentChange(val){
				var _this = this;
				this.formTwo.page=val-1;
				 _this.formTwo.star = (_this.formTwo.page)*20;
           		 _this.formTwo.end = _this.formTwo.star+20;
				//this.getTwoData();
			},
			threeHandleCurrentChange(val){
				this.formThree.page = val;
				this.getThreeData();
			},
			fourHandleCurrentChange(val){
				this.formFour.page = val;
				this.getFourData();
			},
			// fiveHandleCurrentChange(val){
			// 	this.formFive.page = val;
			// 	this.getFiveData();
			// },
			getOneData(){
       			var _this = this;
       			var params=_this.searchConditionGift();
				axios.get(allget+'/NewBox/getDateStat', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
						var oneTitleStatic = [];
						var arr = [];
						var total = [];
						oneTitleStatic = [
							{identification:'date',name:'日期'},
							{identification:'count',name:'游戏次数'},
							{identification:'chat_gold',name:'游戏消耗豆币'},
							{identification:'chat_ticket',name:'送出豆票'},
						]
						_this.formOne.tabOneTitleData = oneTitleStatic.concat(res.data.data.prize_list);
						_this.formOne.tabOneTitleData.forEach((item) => {
							if(item.prize_id){
								var prizeId = item.prize_id;
								item.identification = 'count_'+prizeId;
							};
							arr.push(item);
						});
						res.data.data.stat_total.date = '总计';
						_this.formOne.tabOneTitleData = arr
						_this.formOne.tabData.unshift(res.data.data.stat_total);
						_this.formOne.tabData = res.data.data.stat_list;
						console.log(_this.formOne.tabData.unshift(res.data.data.stat_total),_this.formOne.tabData)
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
				axios.get(allget+'/NewBox/getUserStat', {params: params})
				.then((res) => {
					if(res.data.ret == 1) {
					//	_this.formTwo.tabData = res.data.data;
						var oneTitleStatic = [];
						var arr = [];
						var total = [];
						oneTitleStatic = [
							{identification:'uid',name:'UID'},
							{identification:'count',name:'游戏次数'},
							{identification:'chat_gold',name:'游戏消耗豆币'},
							{identification:'chat_ticket',name:'送出豆票'},
						]
						_this.formTwo.tabOneTitleData = oneTitleStatic.concat(res.data.data.prize_list);
						_this.formTwo.tabOneTitleData.forEach((item) => {
							if(item.prize_id){
								var prizeId = item.prize_id;
								item.identification = 'count_'+prizeId;
							};
							arr.push(item);
						});
						res.data.data.stat_total.uid = '总计';
						_this.formTwo.tabOneTitleData = arr
						_this.formTwo.tabData.unshift(res.data.data.stat_total);
						_this.formTwo.tabData = res.data.data.stat_list;
						console.log(_this.formTwo.tabData.unshift(res.data.data.stat_total),_this.formTwo.tabData)
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
			getThreeData(){
       			var _this = this;
				axios.get(allget+'/NewGoldenEgg/getCheatList', {params: {status:0,page:_this.formThree.page}})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formThree.tabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
			getFourData(){
       			var _this = this;
				axios.get(allget+'/NewGoldenEgg/getCheatList', {params: {status:1,page:_this.formFour.page}})
				.then((res) => {
					if(res.data.ret == 1) {
						_this.formFour.tabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			},
			// getFiveData(){
       		// 	var _this = this;
			// 	axios.get(allget+'/NewTreasureBox/getBlackList', {params: {page:_this.formFive.page}})
			// 	.then((res) => {
			// 		if(res.data.ret == 1) {
			// 			_this.formFive.tabData = res.data.data;
			// 		} else {
			// 			baseConfig.warningTipMsg(_this, res.data.msg);
			// 		}
			// 	})
			// 	.catch((err) => {
			// 		console.log(err);
			// 	})
			// },
			//作弊添加获取礼物	
			checkSort(){
				var _this = this;
				let url = "/NewGoldenEgg/getAvailablePrizeList";
				let params = {
					
				};
				axios.get(allget+url, {params: params})
					.then((res) => {
						if (res.data.ret) {
							_this.getPrize = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.error(err);
					});	
			},
			//添加作弊后台
			addNoReal(num){
				var _this = this;
				// if(num == 0){
				// 	_this.formThree.dialogFormVisibleOther = false;
				// 	_this.formThree.box = '';
				// 	_this.formThree.prize_id = '';
				// 	_this.formThree.uid = '';
				// 	_this.getPrize = [];
				// }else{
				// 	let url = "/NewGoldenEgg/addCheat";
				// 	let params = {
				// 		uid: _this.formThree.uid,
				// 		prize_id: _this.formThree.prize_id,
				// 		admin: store.state.user.name
				// 	};
				// 	axios.get(allget+url, {params: params})
				// 		.then((res) => {
				// 			if (res.data.ret) {
				// 				_this.getThreeData();
				// 				_this.formThree.dialogFormVisibleOther = false;
				// 				baseConfig.successTipMsg(_this, '添加成功');
				// 			} else {
				// 				baseConfig.warningTipMsg(_this, res.data.msg);
				// 			}
				// 		})
				// 		.catch((err) => {
				// 			console.error(err);
				// 		});	
				// };
			},
			//删除作弊
			deleteNoReal(index, row){
				var _this = this;
				_this.listLoading = true;
				this.$confirm('此操作将永久删除, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					var url = '/NewGoldenEgg/deleteCheat';
					let params = {
						cheat_id : row.cheat_id
					};
					axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if (res.data.ret) {
							_this.getThreeData();
							this.$message({
								type: 'success',
								message: '删除成功!'
							});
						} else {
							this.$message({
								type: 'info',
								message: res.data.msg
							});
						}
					})
					.catch((err) => {
						console.error(err);
					});   
				}).catch(() => {
					_this.listLoading = false;
					this.$message({
						type: 'info',
						message: '已取消删除'
					});          
				});
			},
			//添加黑名单
			addBlackList(num){
				var _this = this;
				if(num == 0){
					_this.formFive.dialogFormVisibleOther = false;
					_this.formFive.time = '';
					_this.formFive.uid = '';
				}else{
					let url = "/NewTreasureBox/setBlack";
					let params = {
						uid: _this.formFive.uid,
						time: _this.formFive.time,
						admin: store.state.user.name
					};
					axios.get(allget+url, {params: params})
						.then((res) => {
							if (res.data.ret) {
								_this.getFiveData();
								_this.formFive.dialogFormVisibleOther = false;
								baseConfig.successTipMsg(_this, '添加成功');
							} else {
								baseConfig.warningTipMsg(_this, res.data.msg);
							}
						})
						.catch((err) => {
							console.error(err);
						});	
				};
			},
			//删除黑名单
			deleteBliackList(index,row){
				var _this = this;
				_this.listLoading = true;
				this.$confirm('此操作将永久删除, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					var url = '/NewTreasureBox/deleteBlack';
					let params = {
						uid : row.uid
					};
					axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if (res.data.ret) {
							_this.getFiveData();
							this.$message({
								type: 'success',
								message: '删除成功!'
							});
						} else {
							this.$message({
								type: 'info',
								message: res.data.msg
							});
						}
					})
					.catch((err) => {
						console.error(err);
					});   
				}).catch(() => {
					_this.listLoading = false;
					this.$message({
						type: 'info',
						message: '已取消删除'
					});          
				});
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