<template>
	<!-- 下发豆票 -->
	<section>
		<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
			<el-form :inline="true" style="overflow:hidden;" :model="formOne">
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
                        <span>家族ID</span>
                        <el-input 
                        style="width:150px;" 
                        v-model="formOne.family_id" 
                        placeholder="请输入家族ID"></el-input>
                    </el-form-item>
                <el-form-item>
					<el-button 
					type="primary" 
					@click="getTableData">查询</el-button>
                    <el-button 
					type="primary" 
					@click="formSend.dialogFormVisible=true;">下发豆票</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			ref="tableHeight" 
			:data="onePageTabData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="date" label="日期"></el-table-column>
				<el-table-column prop="family_id" label="家族ID"></el-table-column>
				<el-table-column prop="family_name" label="家族名"></el-table-column>
				<el-table-column prop="chat_ticket" label="下发豆票" ></el-table-column>
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
		
		<!-- 下发豆票 -->
		<el-dialog title="下发豆票" :visible.sync="formSend.dialogFormVisible">
			<el-form :model="formSend">
				<el-form-item label="家族ID" :label-width="formLabelWidth">
					<el-input 
					v-model="formSend.family_id" 
					placeholder="输入用户当前的家族ID"
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="用户ID" :label-width="formLabelWidth">
					<el-input 
					v-model="formSend.uid" 
					placeholder="输入下发的用户ID"
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="下发豆票数量" :label-width="formLabelWidth">
					<el-input 
					v-model="formSend.chat_ticket"
					placeholder="输入豆票数量"
					></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click="sendSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click="sendSure(1)">确 定</el-button>
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
				family_id: '',
			},
			formFind: {
				dialogFormVisible: false,
				tabData: [],
			},
			formSend: {
				dialogFormVisible: false,
				uid: '',
				family_id: '',
				chat_ticket:'',
				operation_name:''
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
			obj.start_date = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.end_date = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.family_id = _this.formOne.family_id;
			obj.page = _this.page;
			return obj;
		},
		// 发送
		sendSure(val) {
			var _this = this;
			if(val==0) {
				_this.formSend.dialogFormVisible=false;
			} else if(val==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('family_id', _this.formSend.family_id);
			  	formData.append('uid', _this.formSend.uid);
			  	formData.append('chat_ticket', _this.formSend.chat_ticket);
			  	formData.append('operation_name', _this.formSend.operation_name);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/NewMoney/giveFamilyChatTicket', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.formSend.dialogFormVisible=false;									
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
			var url = '/NewMoney/getFamilyChatTicketList';
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
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getTableData();
			_this.formSend.operation_name = store.state.user.name; 
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
	width:100%; height: 50px; line-height: 50px; font-weight:bold;
	background: #e3efff; text-align:center;
}
.excelBox .excelInput{
	width:100%; height: 60px;
}
.excelBox .select{
	width:100%; height: 80px;
}
.excelBox .excelInput p,
.excelBox .select p{
	width:100%; height: 36px; text-indent: 20px; line-height: 36px;
}
.excelBox .excelInput input{
    width:300px; display:block; margin: 0 auto;
}
.excelBox .select>div{
	width:300px; display:block; margin: 0 auto;
}
.btns{
    width:100%; height: 50px;
}
.btns button{
    width: 80px; height: 40px; text-align:center; line-height: 40px;
    border: none; border-radius: 5px;
    background-color: #78B2FF; margin-top:20px; color: #fff;
}
.btns button:nth-of-type(1){
    margin-left: 150px; cursor: pointer;
}
.btns button:nth-of-type(2){
    margin-left: 50px; cursor: pointer;
}
</style>
