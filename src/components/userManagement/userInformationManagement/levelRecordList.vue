<template>
	<!-- 等级流水查询 -->
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
					<el-input placeholder="请输入uid" style="width: 120px;" v-model="formOne.uid" auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item style="float:right;">
					<el-button type="primary" @click="getTableData(0)">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<!--用户的数据展示列表-->
		<template>
			<el-table ref="tableHeight" :data="tabData" border fit highlight-current-row v-loading="listLoading" style="width: 100%;" :height="tableHeight">
				<el-table-column prop="uid" label="UID" width="80" ></el-table-column>
				<el-table-column prop="nickname" label="昵称" width="80" ></el-table-column>
				<el-table-column prop="num" label="加经验值" width="80" ></el-table-column>
				<el-table-column prop="action_type" label="动作类型" min-width="100" >
                    <template slot-scope="scope">
                        <p v-if="scope.row.action_type==1">更换真实头像任务</p>
                        <p v-else-if="scope.row.action_type==3">填写个性签名任务</p>
                        <p v-else-if="scope.row.action_type==6">登录任务</p>
                        <p v-else-if="scope.row.action_type==16">通话任务</p>
                        <p v-else-if="scope.row.action_type==17">偷听任务</p>
                        <p v-else-if="scope.row.action_type==40">转盘任务</p>
                        <p v-else-if="scope.row.action_type==57">每日房间进入【经验专用】</p>
                        <p v-else-if="scope.row.action_type==58">每日房间上麦【经验专用】</p>
                        <p v-else-if="scope.row.action_type==59">首次填写职业【经验专用】</p>
                        <p v-else-if="scope.row.action_type==60">首次填写标签【经验专用】</p>
                        <p v-else-if="scope.row.action_type==61">首次分享房间【经验专用】</p>
                        <p v-else-if="scope.row.action_type==62">首次关注房间【经验专用】</p>
                        <p v-else-if="scope.row.action_type==63">首次获得礼物【经验专用】</p>
                        <p v-else-if="scope.row.action_type==64">每日赠送礼物【经验专用】</p>
                        <p v-else-if="scope.row.action_type==65">首次创建家族【经验专用】</p>
                        <p v-else-if="scope.row.action_type==66">首次加入家族【经验专用】</p>
                        <p v-else-if="scope.row.action_type==67">累积通话时长【经验专用】</p>
                        <p v-else-if="scope.row.action_type==68">累计偷听时长【经验专用】</p>
                        <p v-else-if="scope.row.action_type==69">累计分享次数【经验专用】</p>
                        <p v-else-if="scope.row.action_type==70">家族成员【经验专用】</p>
                        <p v-else-if="scope.row.action_type==71">用户等级【经验专用】</p>
                        <p v-else-if="scope.row.action_type==72">好友数【经验专用】</p>
                        <p v-else-if="scope.row.action_type==73">开通钻石会员【经验专用】</p>
                        <p v-else-if="scope.row.action_type==74">开通至尊会员【经验专用】</p>
                        <p v-else-if="scope.row.action_type==75">开通圣尊会员【经验专用】</p>
                        <p v-else-if="scope.row.action_type==76">一键出售失效道具【经验专用】</p>
                        <p v-else-if="scope.row.action_type==77">每日登录奖励【经验专用】</p>
                    </template>
                </el-table-column>
				<el-table-column prop="experience" label="当前经验值" min-width="100" ></el-table-column>
				<el-table-column prop="create_time" label="时间" min-width="100"></el-table-column>
			</el-table>
			<!--工具条-->
			<el-col :span="24" class="toolbar">
				<el-pagination layout="total,prev,pager,next,jumper" @current-change="handleCurrentChange" :page-size="20" :total="totalpage" style="float:right;"></el-pagination>
			</el-col>
		</template>
	</section>
</template>

<script>
import Event from './../../../public_js/event.js';
import { allget } from './../../../api/api';
import store from './../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null, 
			formOne: {
				choiceDate: [new Date()-30*24*60*60*1000, new Date()],
				uid: '', 
			},
			listLoading: false, 
			tabData: [], 
			totalpage: 1000,  
			page: 0, 
			formLabelWidth: '130px', 
		};
	},
	methods: {
		handleCurrentChange(val) {
            this.page = val - 1;
            this.getTableData();
		},
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.uid = _this.formOne.uid;
			obj.page = _this.page;
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			return obj; 
		},
		// 获取数据列表
		getTableData(type) {
            var _this = this ;
            if(type == 0){
                _this.page = 0;
            }
			_this.listLoading = true;
			var url = '/NewLevel/getLevelRecordList';
			var params = _this.searchCondition();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, { params: params })		
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
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
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
            _this.getTableData();
		})
	}
};
</script>

<style lang="css" scoped>
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
