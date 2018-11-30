<template>
	<!-- 广告限制次数 -->
	<!-- dom结构内容 -->
	<section>
		<!--用户的数据展示列表-->
		<template>
			<el-table ref="tableHeight" :data="onePageTabData" border fit highlight-current-row v-loading="listLoading" style="width: 100%;" :height="tableHeight">
				<el-table-column prop="id" label="ID" width="100" sortable ></el-table-column>
				<el-table-column prop="key" label="键" width="200"></el-table-column>
				<el-table-column prop="value" label="值" width="200"></el-table-column>
				<el-table-column prop="desc" label="描述" width="200"></el-table-column>
				<el-table-column label="操作" min-width="300">
					<template slot-scope="scope">
						<el-button type="primary" @click.native.prevent="changeOneUserData(scope.$index, scope.row)" size="small">编辑</el-button>
					</template>
				</el-table-column>
			</el-table>
			<!--工具条-->
			<el-col :span="24" class="toolbar">
				<el-pagination layout="total,prev,pager,next,jumper" @current-change="handleCurrentChange" :page-size="20" :total="totalpage" style="float:right;"></el-pagination>
			</el-col>
			<!-- 新增--对应的dialog -->
			<el-dialog title="编辑修改值、描述（ID、键不能修改）" :visible.sync="dialogFormVisible">
				<el-form :model="formTwo">
					<el-form-item label="ID" :label-width="formLabelWidth">
						<el-input v-model="formTwo.id" auto-complete="off" disabled></el-input>
					</el-form-item>
					<el-form-item label="键" :label-width="formLabelWidth">
						<el-input v-model="formTwo.key" auto-complete="off" disabled></el-input>
					</el-form-item>
					<el-form-item label="值" :label-width="formLabelWidth">
						<el-input v-model="formTwo.value" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="描述" :label-width="formLabelWidth">
						<el-input v-model="formTwo.desc" auto-complete="off"></el-input>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button @click="editorSure(0)">取 消</el-button>
					<el-button type="primary" @click="editorSure(1)">确 定</el-button>
				</div>
			</el-dialog>
		</template>
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
			formTwo: {
				id: '', 
				key: '', 
				value: '',
				desc: '',
				rows: null, 
				index: null, 
			},
			formThree: {
				file: '',
			},
			excelBoxActivity: false, 
			listLoading: false, 
			tabData: [], 
			totalpage: null,
			page: 1,
			star: '0',
			end: '20',
			dialogFormVisible: false,
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
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/User/getAdCondition';
			var params = {};
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
		},
		// 编辑修改某一条随机昵称
		changeOneUserData(index, rows){
			var _this = this;
			var id = rows.id; 	
			_this.formTwo.id = id;
			_this.formTwo.key = rows.key;
			_this.formTwo.value = rows.value;
			_this.formTwo.desc = rows.desc;
			_this.dialogFormVisible = true; // 点击了编辑修改显示dialog框
		},
		// 确定编辑修改随机昵称(val对应的值：0->取消，1->确认)
		editorSure(val) {
			var _this = this;
			if(val==0) {
				_this.dialogFormVisible = false; 
			} else if(val==1) {
				_this.listLoading = true;
				var url = '/User/editAdCondition';
				var params = {
					id: _this.formTwo.id,
					key: _this.formTwo.key,
					value: _this.formTwo.value,
					// 此处是进行id标识进行识别，不会产生对应的中文编码的问题
					desc: _this.formTwo.desc,
				};
				// 判断必须是要有输入value和desc才能进行修改成立
				if(params.value!='' && params.desc!=''){
					_this.dialogFormVisible = false; 
					axios.get(allget+url, {params: params})
						.then(res => {
							if(res.data.ret) {
								_this.listLoading = false;
								baseConfig.successTipMsg(_this, '编辑修改成功！');
								_this.getTableData();
							} else {
								baseConfig.warningTipMsg(_this, res.data.msg);
							}
						})
						.catch((err) => {
							console.log(err);
						});
				} else {
					_this.listLoading = false; 
					baseConfig.warningTipMsg(_this, '修改的值、描述不能为空！');
				}
			} 
		},
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(pageHeight); 
			_this.getTableData();
		})
	}
};
</script>

<style lang="css" scoped>
/* 页面样式css内容 */
.excelBox{
	width: 500px; height: 190px; margin-left: -150px; background: #f1f7ff;
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
