<template>
	<!-- 活动通知管理 -->
	<!-- dom结构内容 -->
	<section>
		<!-- 工具条/头部的搜索条件搜索 -->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" style="overflow: hidden;">
				<el-form-item label="模糊搜索">
					<el-input v-model="find" auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item>
					<el-button type="primary" @click="formTwo.dialogVisible=true;">新增</el-button>
					<el-button type="primary" @click="getTableData">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<!--用户的数据展示列表-->
		<template>
			<el-table ref="tableHeight" :data="onePageTabData" border fit highlight-current-row v-loading="listLoading" style="width: 100%;" :height="tableHeight">
				<el-table-column prop="create_time" label="创建时间" width="150" sortable ></el-table-column>
				<el-table-column prop="sort" label="序号" width="100"></el-table-column>
				<el-table-column prop="valid_time" label="有效截止日期" width="150"></el-table-column>
				<el-table-column label="缩略图" width="400">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<img :src="scope.row.img_url" alt="" style="width: 300px; height: 200px;">
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="title" label="活动标题" width="100"></el-table-column>
				<el-table-column prop="activity_url" label="活动链接" min-width="100"></el-table-column>
				<el-table-column label="操作" width="150">
					<template slot-scope="scope">
						<el-button type="primary" @click.native.prevent="changeOneUserData(scope.$index, scope.row)" size="small">编辑</el-button>
						<el-button type="primary" @click.native.prevent="deleteOneUserData(scope.$index, scope.row)" size="small">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
			<!--工具条-->
			<el-col :span="24" class="toolbar">
				<el-pagination layout="total,prev,pager,next,jumper" @current-change="handleCurrentChange" :page-size="20" :total="totalpage" style="float:right;"></el-pagination>
			</el-col>
			<!-- 编辑修改--对应的dialog -->
			<el-dialog title="编辑修改" :visible.sync="formOne.dialogVisible">
				<el-form :model="formOne">
					<el-form-item label="活动ID" :label-width="formLabelWidth">
						<el-input disabled v-model="formOne.id" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="活动标题" :label-width="formLabelWidth">
						<el-input v-model="formOne.title" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="活动详情" :label-width="formLabelWidth">
						<el-input v-model="formOne.desc" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="活动排序" :label-width="formLabelWidth">
						<el-input v-model="formOne.sort" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="有效截止时间" :label-width="formLabelWidth">
						<div class="block">
							<el-date-picker v-model="formOne.valid_time" type="date" placeholder="选择日期"></el-date-picker>
						</div>
					</el-form-item>
					<el-form-item label="活动缩略图" :label-width="formLabelWidth">
						<input class="filepic fileinput" @change="uploading($event, 1)" type="file" style="display: block;">
				        <img :src="formOne.pic_src" style="width: 300px; height: 200px; margin-top: 20px;"/>
					</el-form-item>
					<el-form-item label="活动地址" :label-width="formLabelWidth">
						<el-input v-model="formOne.activity_url" auto-complete="off"></el-input>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button @click="editorSure(0)">取 消</el-button>
					<el-button type="primary" @click="editorSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<!-- 新增修改--对应的dialog -->
			<el-dialog title="新增活动通知" :visible.sync="formTwo.dialogVisible">
				<el-form :model="formTwo">
					<el-form-item label="活动标题" :label-width="formLabelWidth">
						<el-input v-model="formTwo.title" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="活动详情" :label-width="formLabelWidth">
						<el-input v-model="formTwo.desc" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="活动排序" :label-width="formLabelWidth">
						<el-input v-model="formTwo.sort" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="有效截止时间" :label-width="formLabelWidth">
						<div class="block">
							<el-date-picker v-model="formTwo.valid_time" type="date" placeholder="选择日期"></el-date-picker>
						</div>
					</el-form-item>
					<el-form-item label="活动缩略图" :label-width="formLabelWidth">
						<input class="filepic fileinput" @change="uploading($event, 0)" type="file" style="display: block;">
				        <img :src="formTwo.pic_src" style="width: 300px; height: 200px; margin-top: 20px;"/>
					</el-form-item>
					<el-form-item label="活动地址" :label-width="formLabelWidth">
						<el-input v-model="formTwo.activity_url" auto-complete="off"></el-input>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button @click="addSure(0)">取 消</el-button>
					<el-button type="primary" @click="addSure(1)">确 定</el-button>
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
			formOne: { // 编辑
				dialogVisible: false,
				id: '',
				title: '',
				desc: '',
				sort: '',
				valid_time: '',
				pic: '',
				pic_src: '',
				activity_url: '',
			},
			formTwo: { // 新增
				dialogVisible: false,			
				operation_name: '',
				title: '',
				desc: '',
				sort: '',
				valid_time: '',
				pic: '', 
				pic_src: '', // 显示图片的src
				activity_url: '',
			},
			listLoading: false, 
			find: '', 
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
			obj.find = _this.find;
			for(var key in obj) { 
				if(obj[key]=='') {
					//baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
					return null;
				}
			}
			return obj; 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/GlobalSet/findActivityMessage';
			var params = _this.searchCondition();
			if(params==null) {
				params = '';
			} else {
				// 不进行处理
			}
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
		// 得到上传文件type(0->活动新增，1->活动编辑修改)
        uploading(event, type) {
			var _this = this;
			if(type==0) { // 活动新增
				_this.formTwo.pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.formTwo.pic_src = windowURL.createObjectURL(event.target.files[0]);
			} else if(type==1) { // 活动编辑修改
				_this.formOne.pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.formOne.pic_src = windowURL.createObjectURL(event.target.files[0]);
			}
      	}, 
		// 编辑修改某一条渠道号名称
		changeOneUserData(index, rows){
			var _this = this;
			_this.formOne.id = rows.id;
			_this.formOne.title = rows.title;
			_this.formOne.desc = rows.desc;
			_this.formOne.sort = rows.sort;
			_this.formOne.valid_time = rows.valid_time;
			_this.formOne.pic_src = rows.img_url;
			_this.formOne.activity_url = rows.activity_url;
			_this.formOne.dialogVisible = true; 
		},
		// 编辑修改确定
		editorSure(type) {
			var _this = this;
			if(type==0) {
				_this.formOne.dialogVisible = false;
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('id', _this.formOne.id);
			  	formData.append('title', _this.formOne.title);
			  	formData.append('desc', _this.formOne.desc);
			  	formData.append('sort', _this.formOne.sort);
			  	formData.append('valid_time', baseConfig.changeDateTime(_this.formOne.valid_time, 0));
			  	formData.append('pic', _this.formOne.pic);
			  	formData.append('activity_url', _this.formOne.activity_url);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/GlobalSet/editActivityMessage', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.formOne.dialogVisible = false;									
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '编辑修改成功！');
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
		// 添加新的活动通知操作
		addSure(type) {
			var _this = this;
			if(type==0) {
				_this.formTwo.dialogVisible = false;
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('operation_name', _this.formTwo.operation_name);
			  	formData.append('title', _this.formTwo.title);
			  	formData.append('desc', _this.formTwo.desc);
			  	formData.append('sort', _this.formTwo.sort);
			  	formData.append('valid_time', baseConfig.changeDateTime(_this.formTwo.valid_time, 0));
			  	formData.append('pic', _this.formTwo.pic);
			  	formData.append('activity_url', _this.formTwo.activity_url);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/GlobalSet/sendActivityMessage', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.formTwo.dialogVisible = false;									
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
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
		// 删除某一条活动通知记录
		deleteOneUserData(index, rows) {
			var _this = this;
			var id = rows.id;	
			_this.listLoading = true;
			var url = '/GlobalSet/delActivityMessage';
			var params = {
				id: id,
			};
			axios.get(allget+url, {params: params})
				.then(res => {
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
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getTableData(); 
			_this.formTwo.operation_name = store.state.user.name; 			
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
