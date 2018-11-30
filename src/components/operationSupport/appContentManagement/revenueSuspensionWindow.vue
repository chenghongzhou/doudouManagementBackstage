<template>
	<!-- 收益悬浮窗->table为指的是数据table展示页面 -->
	<section>
		<el-tabs 
		v-model="activeName2" 
		type="border-card" 
		@tab-click="handleClick">
			<el-tab-pane 
			label="任务设置" 
			name="first" 
			:style="{height:tabHeight+'px'}">
				<el-form 
				:label-position="labelPosition" 
				class="demo-ruleForm" 
				label-width="120px" 
				:model="formLabelAlign" 
				style="padding-left:30px;">
					<el-form-item label="任务名称" style="padding-top: 30px;">
						<el-input v-model="formLabelAlign.name"></el-input>
					</el-form-item>
					<el-form-item label="任务图标">
						<el-input v-model="formLabelAlign.icon"></el-input>
					</el-form-item>
					<el-form-item label="任务描述">
						<el-input type="textarea" v-model="formLabelAlign.desc"></el-input>
					</el-form-item>
					<el-form-item label="任务类型">
						<el-select v-model="formLabelAlign.type" placeholder="请选择活动区域">
							<el-option label="一次性类型" value="0"></el-option>
							<el-option label="每日任务" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="奖励类型">
						<el-select v-model="formLabelAlign.reward_type">
							<el-option label="豆币" value="0"></el-option>
							<el-option label="好评分" value="1"></el-option>
							<el-option label="豆票" value="2"></el-option>
							<el-option label="无" value="3"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="奖励数量">
						<el-input v-model="formLabelAlign.reward"></el-input>
					</el-form-item>
					<el-form-item label="跳转类型">
						<el-select v-model="choice_url" :change="choiceUrlValue(choice_url)">
							<el-option label="无" value="0"></el-option>
							<el-option label="H5链接" value="1"></el-option>
							<el-option label="应用内" value="2"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="H5跳转地址" v-if="formLabelAlign.jump_h5_show">
						<el-input v-model="formLabelAlign.jump_url"></el-input>
					</el-form-item>
					<el-form-item label="IOS链接地址" v-if="formLabelAlign.jump_app_show">
						<el-select v-model="formLabelAlign.ios_link">
							<el-option label="首页" value="VOIHomeVC"></el-option>
							<el-option label="发现页" value="VOIFindUserVC"></el-option>
							<el-option label="修改资料页" value="VOIEditUserInfoVC"></el-option>
							<el-option label="抢聊大厅页" value="VOIRushChatVC"></el-option>
							<el-option label="发起抢聊页" value="VOIBegChatVC"></el-option>
							<el-option label="我的页面" value="VOIMineVC"></el-option>
							<el-option label="我的奖金页" value="VOIMyRewardVC"></el-option>
							<el-option label="分享给好友" value="VOIShareView"></el-option>
							<el-option label="会员中心" value="VOIVipCenterVC"></el-option>
							<el-option label="通话设置页" value="VOIConfigChargeVC"></el-option>
							<el-option label="我要上首页-选择频道页" value="VOIMarkVC"></el-option>
							<el-option label="家族页" value="VOIFamilyListVC"></el-option>
							<el-option label="排行榜-土豪榜" value="VOIRankingVC_Wealth"></el-option>
							<el-option label="排行榜-魅力榜" value="VOIRankingVC_Charm"></el-option>
							<el-option label="一键语聊" value="VOIRadarVC_OneKeyChat"></el-option>
							<el-option label="偷听学聊" value="VOIRadarVC_EavesdropLearnChat"></el-option>
							<el-option label="做任务赚豆币" value="VOIChatCoinVC"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="安卓链接地址" v-if="formLabelAlign.jump_app_show">
						<el-select v-model="formLabelAlign.android_link">
							<el-option label="首页" value="FirstMainActivity"></el-option>
							<el-option label="发现页" value="SearchUserActivity"></el-option>
							<el-option label="修改资料页" value="ModifyUserInfoActivity"></el-option>
							<el-option label="抢聊大厅页" value="GrabOrderFragment"></el-option>
							<el-option label="发起抢聊页" value="SendOrderFragment"></el-option>
							<el-option label="我的页面" value="FirstMyFragment"></el-option>
							<el-option label="我的奖金页" value="MyAwardActivity"></el-option>
							<el-option label="分享给好友" value="ShareDialog"></el-option>
							<el-option label="会员中心" value="VipCenterActivateActivity"></el-option>
							<el-option label="通话设置页" value="CallingPriceActivity"></el-option>
							<el-option label="我要上首页-选择频道页" value="SelectTopChannelActivity"></el-option>
							<el-option label="家族页" value="FamilyLobbyActivity"></el-option>
							<el-option label="排行榜-土豪榜" value="WealthRankFragment"></el-option>
							<el-option label="排行榜-魅力榜" value="CharmRankFragment"></el-option>
							<el-option label="一键语聊" value="ConversationMatchActivity"></el-option>
							<el-option label="偷听学聊" value="EavesdropMatchActivity"></el-option>
							<el-option label="做任务赚豆币" value="TaskActivity"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="排序">
						<el-input v-model="formLabelAlign.sort"></el-input>
					</el-form-item>
				</el-form>
				<el-button 
				type="primary" 
				style="margin-left:50px;margin-top:30px;" 
				@click="sendToTask()">确定发送</el-button>
			</el-tab-pane>
			<el-tab-pane 
			label="操作记录管理" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
					<el-form :inline="true" style="overflow: hidden;" :model="formOne">
						<el-form-item>
							<span>任务状态</span>
							<el-select v-model="formOne.status">
								<el-option label="上架" value="1"></el-option>
								<el-option label="下架" value="0"></el-option>
							</el-select>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableData">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeightAddTab" 
					:data="onePageTabData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column type="index" width="50" ></el-table-column>
						<el-table-column prop="id" label="ID" width="50"></el-table-column>
						<el-table-column prop="name" label="任务名称" width="100"></el-table-column>
						<el-table-column label="任务类型" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.type=='0'" :type="scope.row.type">一次性任务</p>
									<p v-if="scope.row.type=='1'" :type="scope.row.type">每日任务</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="desc" label="任务描述" width="100"></el-table-column>
						<el-table-column label="任务图标" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img :src="scope.row.icon" alt="" style="width:100px;height:100px;">
								</div>
							</template>
						</el-table-column>
						<el-table-column label="奖励类型" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.reward_type=='0'" :type="scope.row.reward_type">豆币</p>
									<p v-if="scope.row.reward_type=='1'" :type="scope.row.reward_type">好评分</p>
									<p v-if="scope.row.reward_type=='2'" :type="scope.row.reward_type">豆票</p>
									<p v-if="scope.row.reward_type=='3'" :type="scope.row.reward_type">无</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="reward" label="奖励数量" width="80"></el-table-column>
						<el-table-column label="跳转类型" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="(scope.row.jump_url==null&&
									scope.row.ios_link==null&&
									scope.row.android_link==null)
									||
									(scope.row.jump_url==''&&
									scope.row.ios_link==''&&
									scope.row.android_link=='')">无</p>
									<p v-else-if="scope.row.jump_url!=null&&scope.row.jump_url!=''">H5页面</p>
									<p v-else-if="scope.row.ios_link!=null">应用内</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="跳转链接" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="(scope.row.jump_url==null&&scope.row.ios_link==null&&scope.row.android_link==null)||(scope.row.jump_url==''&&scope.row.ios_link==''&&scope.row.android_link=='')">无</p>
									<p v-else-if="scope.row.jump_url!=null&&scope.row.jump_url!=''">{{scope.row.jump_url}}</p>
									<p v-else-if="scope.row.android_link=='FirstMainActivity'" androidlink="FirstMainActivity" ioslink="VOIHomeVC">首页</p>
									<p v-else-if="scope.row.android_link=='SearchUserActivity'" androidlink="SearchUserActivity" ioslink="VOIFindUserVC">发现页</p>
									<p v-else-if="scope.row.android_link=='ModifyUserInfoActivity'" androidlink="ModifyUserInfoActivity" ioslink="VOIEditUserInfoVC">修改资料页</p>
									<p v-else-if="scope.row.android_link=='GrabOrderFragment'" androidlink="GrabOrderFragment" ioslink="VOIRushChatVC">抢聊大厅页</p>
									<p v-else-if="scope.row.android_link=='SendOrderFragment'" androidlink="SendOrderFragment" ioslink="VOIBegChatVC">发起抢聊页</p>
									<p v-else-if="scope.row.android_link=='FirstMyFragment'" androidlink="FirstMyFragment" ioslink="VOIMineVC">我的页面</p>
									<p v-else-if="scope.row.android_link=='MyAwardActivity'" androidlink="MyAwardActivity" ioslink="VOIMyRewardVC">我的奖金页</p>
									<p v-else-if="scope.row.android_link=='ShareDialog'" androidlink="ShareDialog" ioslink="VOIShareView">分享给好友</p>
									<p v-else-if="scope.row.android_link=='VipCenterActivateActivity'" androidlink="VipCenterActivateActivity" ioslink="VOIVipCenterVC">会员中心</p>
									<p v-else-if="scope.row.android_link=='CallingPriceActivity'" androidlink="CallingPriceActivity" ioslink="VOIConfigChargeVC">通话设置页</p>
									<p v-else-if="scope.row.android_link=='SelectTopChannelActivity'" androidlink="SelectTopChannelActivity" ioslink="VOIMarkVC">我要上首页-选择频道页</p>
									<p v-else-if="scope.row.android_link=='FamilyLobbyActivity'" androidlink="FamilyLobbyActivity" ioslink="VOIFamilyListVC">家族页</p>
									<p v-else-if="scope.row.android_link=='WealthRankFragment'" androidlink="WealthRankFragment" ioslink="VOIRankingVC_Wealth">排行榜-土豪榜</p>
									<p v-else-if="scope.row.android_link=='CharmRankFragment'" androidlink="CharmRankFragment" ioslink="VOIRankingVC_Charm">排行榜-魅力榜</p>
									<p v-else-if="scope.row.android_link=='ConversationMatchActivity'" androidlink="ConversationMatchActivity" ioslink="VOIRadarVC_OneKeyChat">一键语聊</p>
									<p v-else-if="scope.row.android_link=='EavesdropMatchActivity'" androidlink="EavesdropMatchActivity" ioslink="VOIRadarVC_EavesdropLearnChat">偷听学聊</p>
									<p v-else-if="scope.row.android_link=='TaskActivity'" androidlink="TaskActivity" ioslink="VOIChatCoinVC">做任务赚豆币</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="sort" label="任务排序" width="100" sortable ></el-table-column>	
						<el-table-column label="上架状态" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.status=='0'" :type="scope.row.status">已下架</p>
									<p v-if="scope.row.status=='1'" :type="scope.row.status">上架中</p>
								</div>
							</template>
						</el-table-column>					
						<el-table-column label="操作" min-width="100">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								@click.native.prevent="changeOneUserData(scope.$index, scope.row)" 
								size="small">编辑</el-button>								
								<el-button 
								v-if="scope.row.status=='0'" 
								type="primary" 
								@click.native.prevent="grounding(scope.$index, scope.row)" 
								size="small">上架</el-button>
								<el-button 
								v-if="scope.row.status=='1'" 
								plain 
								@click.native.prevent="undercarriage(scope.$index, scope.row)" 
								size="small">下架</el-button>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="handleCurrentChange" 
						:page-size="20" 
						:total="totalpage"
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
		</el-tabs>
	</section>
</template>

<script>
import Event from './../../../public_js/event.js';
import { allget } from '../../../api/api';
import store from '../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tabHeight: null, 
			tableHeight: null,
			tabSearchHeight: null, 
			formOne: {
				status: '1', 
			},
			formTwo: {
				uid: '', 
			},
			labelPosition: 'left', // 设置任务设置页面的提示位置
			formLabelAlign: {
				id: '',
				name: '',
				icon: '',
				type: '',
				android_link: '',
				ios_link: '',
				desc: '',
				reward_type: '',
				reward: '',
				jump_url: '',
				jump_h5_show: false, //H5跳转地址是否进行显示
				jump_app_show: false, //应用内跳转地址是否进行显示
				sort: '',
			},
			choice_url: '', //配合上面的任务设置的form表单进行配置，在任务设置中的跳转类型
			listLoading: false,
			tabData: [], 
			totalpage: null, 
			page: 1,
			star: '0',
			end: '20',
			activeName2: 'second', // 设置为tab切换栏的选中不同的状态(first、second)
		};
	},
	computed:{
		onePageTabData() {
			var _this = this;
			return _this.tabData.slice(_this.star, _this.end);
		},
	},
	methods: {
		handleCurrentChange(val) {
			var _this = this;
			_this.page = val-1;
			_this.star = (_this.page)*20;
			_this.end = _this.star+20;
		},
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.status = _this.formOne.status; 
			for(var key in obj) { 
				if(obj[key]=='') {
					baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
					return null;
				}
			}
			return obj;
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Marquee/getRevenueFloatingWindowData';
			var params = _this.searchCondition();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
					.then((res) => { 
						_this.listLoading = false;
						if(res.data.ret) {
							_this.totalpage = res.data.data.length; 
							_this.tabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 编辑修改某一条随机昵称
		changeOneUserData(index, rows) {
			var _this = this;
			var id = rows.id; 
			_this.activeName2 = 'first'; // 跳转到任务设置页面
			_this.formLabelAlign = rows; // 对数据结构进行组装
			_this.formLabelAlign = {
				id: rows.id,
				name: rows.name,
				icon: rows.icon,
				type: rows.type,
				android_link: rows.android_link,
				ios_link: rows.ios_link,
				desc: rows.desc,
				reward_type: rows.reward_type,
				reward: rows.reward,
				jump_url: rows.jump_url,
				jump_h5_show: false, //H5跳转地址是否进行显示
				jump_app_show: false, //应用内跳转地址是否进行显示
				sort: rows.sort,
			};
			// 对相应的选择跳转类型进行显示
			if((_this.formLabelAlign.jump_url==null&&_this.formLabelAlign.ios_link==null&&_this.formLabelAlign.android_link==null)||(_this.formLabelAlign.jump_url==''&&_this.formLabelAlign.ios_link==''&&_this.formLabelAlign.android_link=='')) {
				_this.choice_url = '0'; // 选择的为无
			} else if(_this.formLabelAlign.jump_url!=null&&_this.formLabelAlign.jump_url!='') {
				_this.choice_url = '1'; // 选择的是h5跳转的链接
			} else {
				_this.choice_url = '2'; // 其他为应用内的
			}
		},
		// 上架的操作
		grounding(index, rows) {
			var _this = this;
			var id = rows.id;	
			var url = 'Marquee/upDownRevenueFloatingWindowData';
			var params = {
				id: id,
				status: 1, //进行上架
			};
			axios.get(allget+url, {params: params})
				.then((res) => {
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '上架成功！');
						_this.getTableData();
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 下架的操作
		undercarriage(index, rows) {
			var _this = this;
			var id = rows.id; 	
			var url = 'Marquee/upDownRevenueFloatingWindowData';
			var params = {
				id: id,
				status: 0, //进行上架
			};
			axios.get(allget+url, {params: params})
				.then((res) => { 
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '下架成功！');
						_this.getTableData();
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				}).catch(function(error){
					console.log(error);
				});
		},
		// 当tab状态栏切换到tab时候进行id...等信息进行清零
		handleClick(tab, event) {
			var _this = this;
			if(tab.label=='任务设置') {
				_this.choice_url = '';
				_this.formLabelAlign = {
					id: '',
					name: '',
					icon: '',
					type: '',
					android_link: '',
					ios_link: '',
					desc: '',
					reward_type: '',
					reward: '',
					jump_url: '',
					jump_h5_show: false, //H5跳转地址是否进行显示
					jump_app_show: false, //应用内跳转地址是否进行显示
					sort: '',
				};
			} 
		},
		// 设置为select跳转类型进行监听choiceUrlValue
		choiceUrlValue(value) {
			var _this = this;
			if(value=='0') { // 0->无；1->H5链接；2->应用内
				_this.formLabelAlign.android_link = '';
				_this.formLabelAlign.ios_link = '';
				_this.formLabelAlign.jump_url = '';
				_this.formLabelAlign.jump_h5_show = false;
				_this.formLabelAlign.jump_app_show = false;
			} else if(value=='1') {
				_this.formLabelAlign.android_link = '';
				_this.formLabelAlign.ios_link = '';
				_this.formLabelAlign.jump_h5_show = true;
				_this.formLabelAlign.jump_app_show = false;
			} else if(value=='2') {
				_this.formLabelAlign.jump_url = '';
				_this.formLabelAlign.jump_h5_show = false;
				_this.formLabelAlign.jump_app_show = true;				
			} else {
				//console.log('为空时，不进行处理'); //当跳转类型为空字符窜时，不进行处理
			}
		},
		// 设置任务发送设置，新增、编辑的确定发送，不同的状态进行不同属性的删减
		sendToTask() {
			var _this = this;
			var params = baseConfig.deepCopy(_this.formLabelAlign);
			var url = '';
			delete params.jump_h5_show;
			delete params.jump_app_show;
			if(_this.formLabelAlign.id=='') { // 新增发送
				delete params.id;
				url = '/Marquee/addRevenueFloatingWindowData';				
			} else { // 编辑修改
				url = '/Marquee/editRevenueFloatingWindowData';
			}
			_this.sendGoServer(url, params);
		},
		// 对服务器的接口进行接口请求，新增、编辑修改
		sendGoServer(url, params) {
			var _this = this;
			axios.get(allget+url, {params: params})
				.then((res) => { 
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '编辑修改成功！');
						_this.getTableData();
						_this.activeName2 = 'second';
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tabHeight = baseConfig.lineNumber(tabHeight);
			_this.tableHeight = baseConfig.lineNumber(tabSearchPageHeight);
			_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight);
			_this.getTableData();
		})
	}
};
</script>

<style lang="css" scoped>
/* 页面样式css内容 */
.excelBox{
	width: 500px; height: 270px; margin-left: -150px; background: #f1f7ff;
	position: absolute; left: 50%; top: 15%; z-index: 1000;
}
p{ margin: 0; }
.excelBox>p{
	width: 100%; height: 50px; line-height: 50px; font-weight: bold;
	background: #e3efff; text-align: center;
}
.excelBox .excelInput{
	width: 100%; height: 60px;
}
.excelBox .select{
	width: 100%; height: 80px;
}
.excelBox .excelInput p,
.excelBox .select p{
	width: 100%; height: 36px; text-indent: 20px; line-height: 36px;
}
.excelBox .excelInput input{
    width: 300px; display: block; margin: 0 auto;
}
.excelBox .select>div{
	width: 300px; display: block; margin: 0 auto;
}
.btns{
    width: 100%; height: 50px;
}
.btns button{
    width: 80px; height: 40px; text-align: center; line-height: 40px;
    border: none; border-radius: 5px;
    background-color: #78B2FF; margin-top: 20px; color: #fff;
}
.btns button:nth-of-type(1){
    margin-left: 150px; cursor: pointer;
}
.btns button:nth-of-type(2){
    margin-left: 50px; cursor: pointer;
}
.demo-ruleForm{
	width: 500px;
}
.el-tab-pane{
	height: 800px;
}
</style>
