<template>
	<!-- 注册用户招呼语 -->
	<!-- dom结构内容 -->
	<section>
		<!-- 提交Excel的弹框 -->
		<div class="excelBox" v-show="excelBoxActivity">
			<p>请上传Excel文件，只限上传为Excel文件</p>
			<div class="excelInput">
				<p>请选择Excel文件:</p>
				<input @change="uploading($event)" type="file" accept="application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" />
			</div>
			<div class="select">
				<p>请选择性别:</p>
				<div>
					<el-select v-model="formThree.threeSexValue" placeholder="请进行选择" style="width: 100%;">
						<el-option label="男" value="1"></el-option>
						<el-option label="女" value="2"></el-option>
					</el-select>
				</div>
				
			</div>
			<div class="btns">
				<!-- 按钮操作部分 -->
	            <button  @click="excelBoxActivity=false;">取消</button>
		        <button  @click="submit()">提交</button>
			</div>
		</div>
		<!-- 工具条/头部的搜索条件搜索 -->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" style="overflow: hidden;" :model="formOne">
				<el-form-item>
					<span>性别</span>
					<el-select v-model="formOne.sexValue">
						<el-option label="男" value="1"></el-option>
						<el-option label="女" value="2"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item>
					<el-button type="primary" @click="dialogFormVisible=true;">新增</el-button>
					<el-button type="primary" @click="excelBoxActivity=true;">上传</el-button>
					<el-button type="primary" @click="getTableData">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<!--用户的数据展示列表-->
		<template>
			<el-table ref="tableHeight" :data="onePageTabData" border fit highlight-current-row v-loading="listLoading" style="width: 100%;" :height="tableHeight">
				<el-table-column type="index" width="100" ></el-table-column>
				<el-table-column prop="value" label="用户招呼语" width="800" sortable ></el-table-column>
				<el-table-column label="操作" min-width="200">
					<template slot-scope="scope">
						<el-button type="primary" @click.native.prevent="deleteOneUserData(scope.$index, scope.row)" size="small">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
			<!--工具条-->
			<el-col :span="24" class="toolbar">
				<el-pagination layout="total,prev,pager,next,jumper" @current-change="handleCurrentChange" :page-size="20" :total="totalpage" style="float:right;"></el-pagination>
			</el-col>
		</template>
		<!-- 新增--对应的dialog -->
		<el-dialog title="新增用户招呼语" :visible.sync="dialogFormVisible">
			<el-form :model="formTwo">
				<el-form-item label="请输入招呼语" :label-width="formLabelWidth">
					<el-input v-model="formTwo.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="请选择性别" :label-width="formLabelWidth">
				<el-select v-model="formTwo.region" placeholder="请进行选择" style="width: 100%;">
					<el-option label="男" value="1"></el-option>
					<el-option label="女" value="2"></el-option>
				</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="addSure(0)">取 消</el-button>
				<el-button type="primary" @click="addSure(1)">确 定</el-button>
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
				sexValue: '1', 
			},
			formTwo: {
				name: '', 
				region: '1', 
			},
			formThree: {
				file: '', 
				threeSexValue: '1', 
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
		// 搜索条件
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.sex = _this.formOne.sexValue;
			for(var key in obj) {
				if(obj[key]=='') {
					baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
					return null;
				}
			}
			return obj;
		},
		// 确定了新增传递过来对应的内容值(val对应的值：0->取消，1->确认)
		addSure(val) {
			var _this = this;
			if(val==0) {
				_this.dialogFormVisible = false;
			} else if(val==1) {
				_this.listLoading = true;
				var url = '/User/addRegisterSayHelloRedis';
				let formData = new FormData();
				formData.append('content', _this.formTwo.name);
				formData.append('position', _this.formTwo.region);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				if(formData.get('content')!=''&&(formData.get('sex')=='1'||formData.get('sex')=='2')){
					_this.dialogFormVisible = false; 
					axios.post(allget+url, formData, config)					
						.then((res) => {
							if(res.data.ret) {
								_this.listLoading = false;
								baseConfig.successTipMsg(_this, '新增成功！');
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
					baseConfig.warningTipMsg(_this, '请填写新增的内容和对应的性别选择');
				}
			} 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/User/getRegisterSayHelloRedis';
			var params = _this.searchCondition();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							_this.totalpage = res.data.data.length;
							var arrObjData = [];
							for(var i=0; i<res.data.data.length; i++) {
								var obj = {};
								obj.value = decodeURIComponent(res.data.data[i]);
								arrObjData.push(obj);
							} 
							_this.tabData = arrObjData;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 删除某一条注册语
		deleteOneUserData(index, rows) {
			var _this = this;
			index = index + (_this.page-1)*20; 
			var content = rows.value; 
			_this.listLoading = true;
			var url = '/User/delRegisterSayHelloRedis';
			var params = {
				content: decodeURIComponent(content),
				sex: _this.formOne.sexValue,
			};
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '删除成功'); 
						_this.getTableData();
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 获取文件保存的操作
		uploading(event) {
			var _this = this;
      	    var windowURL = window.URL || window.webkitURL;
            _this.formThree.file = event.target.files[0];
      	},
		// 提交整个Excel表的操作 
      	submit() {
            var _this = this;
		  	let formData = new FormData();
          	formData.append('file', _this.formThree.file);
          	formData.append('sex', _this.formThree.threeSexValue);
          	let config = {
                headers: {
                    'Content-Type': 'multipart/form-data'  
                }
            };
            axios.post(allget+'/User/addRegisterSayHelloByExcel', formData, config)
				.then((res) => {
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '增加成功'); 
						_this.getTableData();
						_this.excelBoxActivity = false; 
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
			    })
				.catch((err) =>{
					console.log(err);
			    });
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
