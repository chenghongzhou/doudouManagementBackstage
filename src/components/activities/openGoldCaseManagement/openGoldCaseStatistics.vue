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
                            <el-option label="白金宝箱" value="2"></el-option>
							<el-option label="氪金宝箱" value="3"></el-option>
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
			:data="formOne.tabData" 
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
				activityList:[],
				formOne:{
					tabData:[],
					tabOneTitleData:[],
					uid:'',
					page:1,
					box:'1',
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
				obj.box =_this.formOne.box;
				obj.page=_this.formOne.page;
				obj.start_date = _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0):'';
                obj.end_date = _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0):'';
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
				axios.get(allget+'/NewTreasureBox/getLotteryDateStat', {params: params})
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
				axios.get(allget+'/NewTreasureBox/getLotteryUserStat', {params: params})
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