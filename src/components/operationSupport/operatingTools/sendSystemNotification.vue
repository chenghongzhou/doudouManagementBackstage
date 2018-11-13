<template>
	<!-- 发送系统通知 -->
	<!-- dom结构内容 -->
	<section>
		<!-- 工具条/头部的搜索条件搜索 -->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" style="overflow: hidden;" :model="formOne">
				<el-form-item>
					<div class="block">
						<span class="registerTime">日期</span>
						<el-date-picker v-model="formOne.choiceDate" type="daterange" range-separator=" 至 " placeholder="选择日期范围"></el-date-picker>
					</div>
				</el-form-item>
				<el-form-item>
					<span>发送状态</span>
					<el-select v-model="formOne.send_status">
						<el-option label="全部" value=""></el-option>
						<el-option label="未发送" value="0"></el-option>
						<el-option label="已发送" value="1"></el-option>
						<el-option label="取消发送" value="2"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item>
					<el-button type="primary" @click="formCondition.dialogFormVisible=true;">按条件发送</el-button>
					<el-button type="primary" @click="formUid.dialogFormVisible=true;">按UID发送</el-button>
					<el-button type="primary" @click="getTableData">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<!--用户的数据展示列表-->
		<template>
			<el-table ref="tableHeight" :data="onePageTabData" border fit highlight-current-row v-loading="listLoading" style="width: 100%;" :height="tableHeight">
				<el-table-column prop="time" label="添加时间" width="150" sortable ></el-table-column>
				<el-table-column prop="send_time" label="发送时间" width="150" sortable ></el-table-column>
				<el-table-column prop="uid_list" label="接收人列表" width="150" sortable ></el-table-column>
				<el-table-column label="接收人地区" width="80" sortable >
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.address==0">全国</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="接收人性别" width="80" sortable >
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.condition_sex==0">全部</p>
							<p v-else-if="scope.row.condition_sex==1">男</p>
							<p v-else-if="scope.row.condition_sex==2">女</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="content" label="发送内容" min-width="150" sortable ></el-table-column>
				<el-table-column prop="operation_name" label="发送人" width="150" sortable ></el-table-column>
				<el-table-column label="发送状态" width="150" sortable >
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.send_status==0">未发送</p>
							<p v-else-if="scope.row.send_status==1">已发送</p>
							<p v-else-if="scope.row.send_status==2">取消发送</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="是否定时" width="100" sortable >
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.is_timing==0">不定时</p>
							<p v-else-if="scope.row.is_timing==1">定时</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="操作" width="100">
					<template slot-scope="scope">
						<el-button v-if="scope.row.send_status==0" type="primary" @click.native.prevent="cacelOneUserData(scope.$index, scope.row)" size="small">取消发送</el-button>
						<p v-else-if="scope.row.send_status==1||scope.row.send_status==2">不可改</p>
					</template>
				</el-table-column>
			</el-table>
			<!--工具条-->
			<el-col :span="24" class="toolbar">
				<el-pagination layout="total,prev,pager,next,jumper" @current-change="handleCurrentChange" :page-size="20" :total="totalpage" style="float:right;"></el-pagination>
			</el-col>
		</template>
		<!-- 按条件发送--对应的dialog -->
		<el-dialog title="按条件发送" :visible.sync="formCondition.dialogFormVisible">
			<el-form :model="formCondition">
				<el-form-item label="性别" :label-width="formLabelWidth">
					<el-radio v-model="formCondition.condition_sex" label="0">全部</el-radio>
					<el-radio v-model="formCondition.condition_sex" label="1">男用户</el-radio>
					<el-radio v-model="formCondition.condition_sex" label="2">女用户</el-radio>
				</el-form-item>
				<el-form-item label="注册时间" :label-width="formLabelWidth">
					<div class="block">
						<el-date-picker v-model="formCondition.registerTime" type="daterange" range-separator=" 至 " start-placeholder="开始日期" end-placeholder="结束日期"></el-date-picker>
					</div>
				</el-form-item>
				<el-form-item label="登陆时间" :label-width="formLabelWidth">
					<div class="block">
						<el-date-picker v-model="formCondition.loginTime" type="daterange" range-separator=" 至 " start-placeholder="开始日期" end-placeholder="结束日期"></el-date-picker>
					</div>
				</el-form-item>
				<el-form-item label="地区" :label-width="formLabelWidth">
					<el-select v-model="formCondition.address">
						<el-option label="全国" value="0"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="系统" :label-width="formLabelWidth">
					<el-radio v-model="formCondition.channel" label="0">全部</el-radio>
					<el-radio v-model="formCondition.channel" label="1">ios</el-radio>
					<el-radio v-model="formCondition.channel" label="2">android</el-radio>
				</el-form-item>
				<el-form-item label="版本号" :label-width="formLabelWidth">
					<el-input v-model="formCondition.version_name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="发送群体" :label-width="formLabelWidth">
					<el-select v-model="formCondition.identity">
						<el-option label="全部" value="0"></el-option>
						<el-option label="主播" value="1"></el-option>
						<el-option label="vip用户" value="2"></el-option>
						<el-option label="代理" value="3"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="发送文字" :label-width="formLabelWidth">
					<el-input type="textarea" :rows="2" :maxlength="5000" v-model="formCondition.content" auto-complete="off"></el-input>
					<p style="font-weight: bold;">在发送文字正文中相应位置插入{messageDisplay}超链接才生效</p>
				</el-form-item>
				<el-form-item label="显示的文字" :label-width="formLabelWidth">
					<el-input v-model="formCondition.display" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="链接类型" :label-width="formLabelWidth">
					<el-select v-model="formCondition.Link_type">
						<el-option label="应用" value="1"></el-option>
						<el-option label="H5页面" value="2"></el-option>
						<el-option label="跳转外部浏览器打开" value="3"></el-option>
						<el-option label="跳转打电话界面" value="4"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="ios中链接地址" :label-width="formLabelWidth">
					<el-input v-model="formCondition.ios_link" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="安卓中链接地址" :label-width="formLabelWidth">
					<el-input v-model="formCondition.android_link" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="是否定时" :label-width="formLabelWidth">
					<el-radio v-model="formCondition.is_timing" label="0">无</el-radio>
					<el-radio v-model="formCondition.is_timing" label="1">定时</el-radio>
				</el-form-item>
				<el-form-item v-if="formCondition.is_timing==1" label="定时时间" :label-width="formLabelWidth">
					<div class="block">
						<el-date-picker v-model="formCondition.send_time" type="datetime" placeholder="选择发送的时间"></el-date-picker>
					</div>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="conditionSure(0)">取 消</el-button>
				<el-button type="primary" @click="conditionSure(1)">确 定</el-button>
			</div>
		</el-dialog>
		<!-- 按uid发送--对应的dialog -->
		<el-dialog title="按Uid发送" :visible.sync="formUid.dialogFormVisible">
			<el-form :model="formUid">
				<el-form-item label="发送的Uid" :label-width="formLabelWidth">
					<el-input type="textarea" :rows="2" :maxlength="50" v-model="formUid.uid_list" auto-complete="off"></el-input>
					<p>每个Uid，用英文逗号隔开(例如:10000,10001,12201)</p>
				</el-form-item>
				<el-form-item label="发送文字" :label-width="formLabelWidth">
					<el-input type="textarea" :rows="2" :maxlength="50" v-model="formUid.content" auto-complete="off"></el-input>
					<p style="font-weight: bold;">在发送文字正文中相应位置插入{messageDisplay}超链接才生效</p>
				</el-form-item>
				<el-form-item label="显示的文字" :label-width="formLabelWidth">
					<el-input v-model="formUid.display" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="链接类型" :label-width="formLabelWidth">
					<el-select v-model="formUid.Link_type">
						<el-option label="应用" value="1"></el-option>
						<el-option label="H5页面" value="2"></el-option>
						<el-option label="跳转外部浏览器打开" value="3"></el-option>
						<el-option label="跳转打电话界面" value="4"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="ios中链接地址" :label-width="formLabelWidth">
					<el-input v-model="formUid.ios_link" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="安卓中链接地址" :label-width="formLabelWidth">
					<el-input v-model="formUid.android_link" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="是否定时" :label-width="formLabelWidth">
					<el-radio v-model="formUid.is_timing" label="0">无</el-radio>
					<el-radio v-model="formUid.is_timing" label="1">定时</el-radio>
				</el-form-item>
				<el-form-item v-if="formUid.is_timing==1" label="定时时间" :label-width="formLabelWidth">
					<div class="block">
						<el-date-picker v-model="formUid.send_time" type="datetime" placeholder="选择发送的时间"></el-date-picker>
					</div>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="uidSure(0)">取 消</el-button>
				<el-button type="primary" @click="uidSure(1)">确 定</el-button>
			</div>
		</el-dialog>
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
			tableHeight: null, 
			formOne: {
				choiceDate: [new Date()-180*24*60*60*1000, new Date()], 
				send_status: '',
			},
			formCondition: {
				dialogFormVisible: false, 
				appoint_type: '0', //条件发送值为0
				// 开始
				condition_sex: '0', //性别
				registerTime: [new Date()-180*24*60*60*1000, new Date()], //注册时间
				loginTime: [new Date()-180*24*60*60*1000, new Date()], //登入时间
				addtime_start: '', //注册时间
				addtime_end: '', //注册时间
				lasttime_start: '', //登入时间
				lasttime_end: '', //登入时间
				address: '0', //地区
				channel: '0', //系统渠道（全部、ios、安卓）
				version_name: '', //版本号
				identity: '0', //发送群体
				// 结束
				// *****
				uid_list: ' ', //uid列表
				// *****
				content_type: '0', //默认为发送文字0,发送图片为1
				content: '', //发送文字的内容
				display: '', //显示的文字内容
				Link_type: '1', //链接类型
				ios_link: '', //ios链接地址
				android_link: '', //安卓链接地址
				is_timing: '0', //是否定时（0->不定时，1->定时）
				send_time: '', //定时的时间
				operation_name: '', //操作人
			},
			formUid: {
				dialogFormVisible: false, // 控制dialog弹框的显示和隐藏
				appoint_type: '1', //uid发送值为1
				// *****
				condition_sex: '0', //性别
				addtime_start: '', //注册时间
				addtime_end: '', //注册时间
				lasttime_start: '', //登录时间
				lasttime_end: '', //登录时间
				address: '0', //地区
				channel: '0', //系统渠道（全部、ios、安卓）
				version_name: '', //版本号
				identity: '0', //发送群体
				// *****
				// 开始
				uid_list: ' ', //uid列表
				// 结束
				content_type: '0', //默认为发送文字0,发送图片为1
				content: '', //发送文字的内容
				display: '', //显示的文字内容
				Link_type: '1', //链接类型
				ios_link: '', //ios链接地址
				android_link: '', //安卓链接地址
				is_timing: '0', //是否定时（0->不定时，1->定时）
				send_time: '', //定时的时间
				operation_name: '', //操作人
			},
			listLoading: false,
			tabData: [], 
			totalpage: null,  
			page: 1, 
			star: '0', 
			end: '20', 
			formLabelWidth: '120px',
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
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.send_status = _this.formOne.send_status; 
			for(var key in obj) {
				if(key=='send_status') {
				} else {
					if(obj[key]=='') {
						baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
						return null;
					}
				}
			}
			return obj;
		},
		conditionSure(val) {
			var _this = this;
			if(val==0) {
				_this.formCondition.dialogFormVisible = false; 
			} else if(val==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('condition_sex', _this.formCondition.condition_sex);
			  	formData.append('addtime_start', baseConfig.changeDateTime(_this.formCondition.registerTime[0], 0));
			  	formData.append('addtime_end', baseConfig.changeDateTime(_this.formCondition.registerTime[1], 0));
			  	formData.append('lasttime_start', baseConfig.changeDateTime(_this.formCondition.loginTime[0], 0));
			  	formData.append('lasttime_end', baseConfig.changeDateTime(_this.formCondition.loginTime[1], 0));
			  	formData.append('address', _this.formCondition.address);
			  	formData.append('appoint_type', _this.formCondition.appoint_type);
			  	formData.append('uid_list', _this.formCondition.uid_list);
			  	formData.append('content_type', _this.formCondition.content_type);
			  	formData.append('content', _this.formCondition.content);
			  	formData.append('display', _this.formCondition.display);
			  	formData.append('Link_type', _this.formCondition.Link_type);
			  	formData.append('ios_link', _this.formCondition.ios_link);
			  	formData.append('android_link', _this.formCondition.android_link);
			  	formData.append('operation_name', _this.formCondition.operation_name);
			  	formData.append('channel', _this.formCondition.channel);
			  	formData.append('version_name', _this.formCondition.version_name);
			  	formData.append('is_timing', _this.formCondition.is_timing);
			  	formData.append('send_time', baseConfig.changeDateTime(_this.formCondition.loginTime, 1));
			  	formData.append('identity', _this.formCondition.identity);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/GlobalSet/sendSystemMessage', formData, config)
				.then((res) => {
					_this.listLoading = false;	
					_this.formCondition.dialogFormVisible = false;									
					if(res.data.ret) {	
						baseConfig.successTipMsg(_this, '发送成功！');
						_this.getTableData();
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);						
					}
				}).catch((error) => {
					console.log(error);
				});
			} 
		},
		// uid发送的确定发送
		uidSure(val) {
			var _this = this;
			if(val==0) {
				_this.formUid.dialogFormVisible = false; 
			} else if(val==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('condition_sex', _this.formUid.condition_sex);
			  	formData.append('addtime_start', '');
			  	formData.append('addtime_end', '');
			  	formData.append('lasttime_start', '');
			  	formData.append('lasttime_end', '');
			  	formData.append('address', _this.formUid.address);
			  	formData.append('appoint_type', _this.formUid.appoint_type);
			  	formData.append('uid_list', _this.formUid.uid_list);
			  	formData.append('content_type', _this.formUid.content_type);
			  	formData.append('content', _this.formUid.content);
			  	formData.append('display', _this.formUid.display);
			  	formData.append('Link_type', _this.formUid.Link_type);
			  	formData.append('ios_link', _this.formUid.ios_link);
			  	formData.append('android_link', _this.formUid.android_link);
			  	formData.append('operation_name', _this.formUid.operation_name);
			  	formData.append('channel', _this.formUid.channel);
			  	formData.append('version_name', _this.formUid.version_name);
			  	formData.append('is_timing', _this.formUid.is_timing);
			  	formData.append('send_time', baseConfig.changeDateTime(_this.formUid.loginTime, 1));
			  	formData.append('identity', _this.formUid.identity);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/GlobalSet/sendSystemMessage', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.formUid.dialogFormVisible = false;									
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '发送成功！');
							_this.getTableData();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
					})
					.catch((err) => {
						console.log(err);
					});
			} 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/GlobalSet/findSystemMessageLog';
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
		// 取消发送操作
		cacelOneUserData(index, rows) {
			var _this = this;
			index = index + (_this.page-1)*20;
			var id = rows.id;	
			_this.listLoading = true;
			let formData = new FormData();
			formData.append('id', id);
			let config = {
				headers: {
					'Content-Type': 'multipart/form-data'
				}
			};		
			axios.post(allget+'/GlobalSet/delSystemMessage', formData, config)
				.then((res) => {
					_this.listLoading = false;	
					if(res.data.ret) {	
						baseConfig.successTipMsg(_this, '取消发送成功！');
						_this.getTableData();
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
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getTableData();
			_this.formCondition.operation_name = store.state.user.name; 
			_this.formUid.operation_name = store.state.user.name; 
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
</style>
