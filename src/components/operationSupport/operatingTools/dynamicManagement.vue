<template>
	<!-- 跑马灯管理 -->
	<section>
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<el-form 
			:inline="true" 
			style="overflow:hidden;" 
			:model="formOne">
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
					<span>关键字查找</span>
					<el-input 
					style="width:150px;" 
					v-model="formOne.key_word" 
					placeholder="请输入关键字查找"></el-input>
				</el-form-item>
        <el-form-item>
					<el-button 
					type="primary" 
					@click="getTableData">查询</el-button>
					<el-button 
					type="primary"
					@click="addNewloading.dialogFormVisible=true;">添加新动态</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			ref="tableHeight" 
			:data="TabData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="add_time" label="发布时间" ></el-table-column>
				<el-table-column prop="index_position" label="动态排序" width="40" ></el-table-column>
				<el-table-column prop="title" label="动态标题" ></el-table-column>
				<el-table-column prop="share_content" label="动态内容" width="200" ></el-table-column>
				<el-table-column label="图片" width="150">
						<template slot-scope="scope">
							<div slot="reference" class="name-wrapper">
								<img 
								:src="scope.row.image_url" 
								style="width:100px;height:auto;">
							</div>
						</template>
					</el-table-column>
				<el-table-column label="跳转类型" width="100">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.type==1">应用内</p>
							<p v-if="scope.row.type==3">应用外</p>
							<p v-else-if="scope.row.type==2">H5页面</p>
							<p v-else-if="scope.row.type==6">偷听界面</p>
							<p v-else-if="scope.row.type==0">不跳转</p>
							<p v-else-if="scope.row.type==12">跳转到游戏</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="jump_url" label="跳转链接" width="200" ></el-table-column>				
				<el-table-column label="展示时间段" >
					<template slot-scope="scope">
						<div v-if="scope.row.show_s_time && scope.row.show_e_time">
							{{scope.row.show_s_time}}-{{scope.row.show_e_time}}
						</div>
					</template>
					</el-table-column>				
				<el-table-column label="展示频率" width="50">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.show_type==0">每次都显示</p>
							<p v-else-if="scope.row.show_type==1">一天显示一次</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="上线状态" width="50">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.is_show==0">已下线</p>
							<p v-else-if="scope.row.is_show==1">已上线</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="操作">
					<template slot-scope="scope">
							<el-button 
									size="mini" 
									type="primary" 
									@click="editorBtn(scope.$index, scope.row)">编辑</el-button>
							<el-button 
									size="mini" 
									type="primary"
									style="margin-left:0;" 
									v-if="scope.row.is_show == 0"
									@click="setShow(scope.$index, scope.row,1)">上线</el-button>
							<el-button 
									size="mini" 
									type="primary"
									style="margin-left:0;" 
									v-if="scope.row.is_show == 1"
									@click="setShow(scope.$index, scope.row,0)">下线</el-button>		
							<el-button 
									size="mini" 
									type="danger" 
									style="margin-left:0;" 
									@click="setDelete(scope.$index, scope.row)">删除</el-button>
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
		<!-- 动态新增 -->
			<el-dialog title="动态新增" :visible.sync="addNewloading.dialogFormVisible">
				<el-form :model="addNewloading.params">
					<!-- <el-form-item label="动态排序" :label-width="formLabelWidth">
						<el-select v-model="addNewloading.params.index_position">
							 <el-option
								v-for="item in addSort"
								:key="item"
								:label="item"
								:value="item">
							</el-option>
						</el-select>
					</el-form-item> -->
					<div class="addTitle">选择排序参考：</div>
					<template>
						<el-table 
						:data="addSort" 
						v-loading="listLoading" 
						border fit highlight-current-row 
						style="width:100%;margin-bottom:10px;" 
						:height="tableHeightOther"
						>
							<el-table-column prop="index_position" label="可选序号" width='200px'></el-table-column>
							<el-table-column prop="is_show" label="是否显示在客户端">
								<template slot-scope="scope">
									<div slot="reference" class="name-wrapper">
										<p v-if="scope.row.is_show==0">不显示</p>
										<p v-else-if="scope.row.is_show==1">显示</p>
									</div>
								</template>
							</el-table-column>
						</el-table> 
					</template>     
					<el-form-item label="动态排序" :label-width="formLabelWidth">
						<el-input 
						style="width:250px"
						v-model="addNewloading.params.index_position" 
						placeholder="请输入数字,数字不能为1,2"
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="banner标题" :label-width="formLabelWidth">
						<el-input 
						v-model="addNewloading.params.title" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="动态内容" :label-width="formLabelWidth">
						<el-input
							type="textarea"
							:autosize="{ minRows: 2, maxRows: 4}"
							placeholder="请输入内容"
							:maxlength="140"
							v-model="addNewloading.params.share_content">
						</el-input>
					</el-form-item>
					<el-form-item label="图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filepic fileinput" 
						@change="uploading($event, 0)" 
						type="file">
						<span class="showname">{{addNewloading.pic_name}}</span>
				        <img 
						:src="addNewloading.src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="跳转类型" :label-width="formLabelWidth">
						<el-select v-model="addNewloading.params.type">
							<el-option label="不跳转" value="0"></el-option>
							<el-option label="H5跳转" value="2"></el-option>
							<el-option label="应用内" value="1"></el-option>
							<el-option label="应用外" value="3"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="显示开始时间" :label-width="formLabelWidth">
							<el-date-picker 
							v-model="addNewloading.params.show_s_time" 
							type="datetime" 
							value-format="yyyy-MM-dd HH:mm:ss"
							placeholder="选择日期范围" style="width:100%"></el-date-picker>
					</el-form-item>
					<el-form-item label="显示结束时间" :label-width="formLabelWidth">
							<el-date-picker 
							v-model="addNewloading.params.show_e_time" 
							type="datetime" 
							value-format="yyyy-MM-dd HH:mm:ss"
							placeholder="选择日期范围" style="width:100%"></el-date-picker>
					</el-form-item>
					<el-form-item label="跳转链接" :label-width="formLabelWidth">
						<el-input 
						v-model="addNewloading.params.jump_url" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="展示频率" :label-width="formLabelWidth">
						<el-select v-model="addNewloading.params.show_type">
							<el-option label="每次都显示" value="0"></el-option>
							<el-option label="一天显示一次" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="上线设置" :label-width="formLabelWidth">
						<el-select v-model="addNewloading.params.is_show">
							<el-option label="上线" value="1"></el-option>
							<el-option label="下线" value="0"></el-option>
						</el-select>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="addBannerSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="addBannerSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<!-- 动态修改 -->
			<el-dialog title="动态修改" :visible.sync="editorloading.dialogFormVisible">
				<el-form :model="editorloading.params">
					<!-- <el-form-item label="动态排序" :label-width="formLabelWidth">
						<el-select v-model="editorloading.params.index_position">
							 <el-option
								v-for="item in editorSort"
								:key="item"
								:label="item"
								:value="item">
							</el-option>
						</el-select>
					</el-form-item> -->
					<div class="addTitle">选择排序参考：</div>
					<template>
						<el-table 
						:data="editorSort" 
						v-loading="listLoading" 
						border fit highlight-current-row 
						style="width:100%;margin-bottom:10px;" 
						:height="tableHeightOther"
						>
							<el-table-column prop="index_position" label="可选序号" width='200px'></el-table-column>
							<el-table-column prop="is_show" label="是否显示在客户端">
								<template slot-scope="scope">
									<div slot="reference" class="name-wrapper">
										<p v-if="scope.row.is_show==0">不显示</p>
										<p v-else-if="scope.row.is_show==1">显示</p>
									</div>
								</template>
							</el-table-column>
						</el-table> 
					</template>     
					<el-form-item label="动态排序" :label-width="formLabelWidth">
						<el-input 
						style="width:250px"
						v-model="editorloading.params.index_position" 
						placeholder="请输入数字,数字不能为1,2"
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="banner标题" :label-width="formLabelWidth">
						<el-input 
						v-model="editorloading.params.title" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="动态内容" :label-width="formLabelWidth">
						<el-input
							type="textarea"
							:autosize="{ minRows: 2, maxRows: 4}"
							placeholder="请输入内容"
							:maxlength="140"
							v-model="editorloading.params.share_content">
						</el-input>
					</el-form-item>
					<el-form-item label="图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filepic fileinput" 
						@change="uploading($event, 1)" 
						type="file">
						<span class="showname">{{editorloading.pic_name}}</span>
				        <img 
						:src="editorloading.src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="跳转类型" :label-width="formLabelWidth">
						<el-select v-model="editorloading.params.type">
							<el-option label="不跳转" value="0"></el-option>
							<el-option label="H5跳转" value="2"></el-option>
							<el-option label="应用内" value="1"></el-option>
							<el-option label="应用外" value="3"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="显示开始时间" :label-width="formLabelWidth">
						<el-date-picker 
						v-model="editorloading.params.show_s_time" 
						type="datetime" 
						value-format="yyyy-MM-dd HH:mm:ss"
						placeholder="选择日期范围" style="width:100%"></el-date-picker>
					</el-form-item>
					<el-form-item label="显示结束时间" :label-width="formLabelWidth">
						<el-date-picker 
						v-model="editorloading.params.show_e_time" 
						type="datetime" 
						value-format="yyyy-MM-dd HH:mm:ss"
						placeholder="选择日期范围" style="width:100%"></el-date-picker>
					</el-form-item>
					<el-form-item label="跳转链接" :label-width="formLabelWidth">
						<el-input 
						v-model="editorloading.params.jump_url" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="展示频率" :label-width="formLabelWidth">
						<el-select v-model="editorloading.params.show_type">
							<el-option label="每次都显示" value="0"></el-option>
							<el-option label="一天显示一次" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="上线设置" :label-width="formLabelWidth">
						<el-select v-model="editorloading.params.is_show">
							<el-option label="上线" value="1"></el-option>
							<el-option label="下线" value="0"></el-option>
						</el-select>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="editorBannerSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="editorBannerSure(1)">确 定</el-button>
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
			operate_user: '', 
			formOne : {
				choiceDate: [new Date()-180*24*60*60*1000, new Date()], 
				position: '',
				key_word : '',
			},
			addNewloading: {
				dialogFormVisible: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '',
					index_position: '',
					title: '',
					type: '2',
					show_s_time: '',
					show_e_time: '',
					jump_url: '',
					page_param: '',
					show_type: '0',
					share_content:'',
					is_show:'0',
				},
			},
			editorloading: {
				dialogFormVisible: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '',
					index_position: '',
					title: '',
					type: '2',
					show_s_time: '',
					show_e_time: '',
					jump_url: '',
					page_param: '',
					show_type: '0',
					share_content:'',
					is_show:'0',
					id:'',
					s_time_spare:'',
					e_time_spare:'',
				},
			},
			listLoading: false, 
			TabData: [], 
			addSort:null,
			editorSort:null,
			totalpage: null,  
			page: 0, 
			dialogFormVisible: false, 
			formLabelWidth: '130px', 
			tableHeightOther: '200px'
		};
	},
	methods: {
		handleCurrentChange(val) {
			var _this = this;
			_this.page = val-1;
			_this.getTableData();
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewContent/getContentBannerList';
			var params = {
				start_date : baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0),
				end_date : baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0),
				keyword : _this.formOne.key_word
			}
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						_this.TotalPage = res.data.data.length; 
						_this.TabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
					baseConfig.warningTipMsg(_this, 'error');
				});
		},
		//获取动态排序
		getSort(){
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewContent/getContentBannerIndexPositionList';
			var params = {
				
			}
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						_this.addSort = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
					baseConfig.warningTipMsg(_this, 'error');
				});
		},
		// 得到上传文件type(0->新增banner，1->编辑banner)
		uploading(event, type) {
			var _this = this;
			if(type==0) {
				_this.addNewloading.file_pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.addNewloading.src_pic = windowURL.createObjectURL(event.target.files[0]);
				_this.addNewloading.pic_name = event.target.files[0].name;
			} else if(type==1) {
				_this.editorloading.file_pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.editorloading.src_pic = windowURL.createObjectURL(event.target.files[0]);
				_this.editorloading.pic_name = event.target.files[0].name;
			}
  	}, 
		// banner条新增
		addBannerSure(type) {
			var _this = this;
			if(type==0) {
				_this.addNewloading.dialogShow = false;
				_this.resetForm();
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
				formData.append('index_position', _this.addNewloading.params.index_position);
				formData.append('title', _this.addNewloading.params.title);
				formData.append('type', _this.addNewloading.params.type);
				formData.append('jump_url', _this.addNewloading.params.jump_url);
				formData.append('show_type', _this.addNewloading.params.show_type);
				formData.append('share_content', _this.addNewloading.params.share_content);
				formData.append('is_show', _this.addNewloading.params.is_show);
			  	formData.append('image', _this.addNewloading.file_pic); //提交的新增图标的文件
			  	formData.append('show_s_time', _this.addNewloading.params.show_s_time);
			  	formData.append('show_e_time', _this.addNewloading.params.show_e_time);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/NewContent/setContentBanner', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.addNewloading.dialogShow = false;								
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							_this.getTableData();
							_this.addNewloading.dialogShow = true;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
						_this.resetForm();
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		//编辑
		editorBtn(index,rows){
			var _this = this;
			var url = '/NewContent/getContentBannerIndexPositionList';
			var params = {
				index_position : rows.index_position
			}
			axios.get(allget+url, {params: params})
				.then((res) => {
					if(res.data.ret) {
						_this.editorSort = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
					baseConfig.warningTipMsg(_this, 'error');
				});
				_this.editorloading.params.index_position = rows.index_position;
				_this.editorloading.params.id = rows.id;
				_this.editorloading.params.title = rows.title;
				_this.editorloading.params.type = rows.type;
				_this.editorloading.params.jump_url = rows.jump_url;
				_this.editorloading.params.show_type = rows.show_type;
				_this.editorloading.params.is_show = rows.is_show;
				_this.editorloading.params.share_content = rows.share_content;
				_this.editorloading.src_pic = rows.image_url;
				_this.editorloading.new_src_pic = rows.new_image_url;

				if(rows.show_s_time==null||rows.show_s_time=='null') {//为null时特殊处理
					_this.editorloading.params.show_s_time = '';
					_this.editorloading.params.s_time_spare  = '';
				} else {
					_this.editorloading.params.s_time_spare = rows.show_s_time;
					_this.editorloading.params.show_s_time = new Date(rows.show_s_time);
				}
				if(rows.show_e_time==null||rows.show_e_time=='null') {//为null时特殊处理
					_this.editorloading.params.show_e_time = '';
					_this.editorloading.params.e_time_spare = '';
				} else {
					_this.editorloading.params.e_time_spare = rows.show_e_time;
					_this.editorloading.params.show_e_time = new Date(rows.show_e_time);
				}
				_this.editorloading.dialogFormVisible = true;
		},
		editorBannerSure(type){
			var _this = this;
			if(type==0) {
				_this.editorloading.dialogShow = false;
				_this.resetForm();
			} else if(type==1) {
				_this.listLoading = true;
				if(_this.editorloading.params.show_s_time.toString().indexOf('(中国标准时间)') != '-1'){
					_this.editorloading.params.show_s_time = _this.editorloading.params.s_time_spare;
				}
				if(_this.editorloading.params.show_e_time.toString().indexOf('(中国标准时间)') != '-1'){
					_this.editorloading.params.show_e_time = _this.editorloading.params.e_time_spare;
				}
				let formData = new FormData();
				formData.append('index_position', _this.editorloading.params.index_position);
				formData.append('id', _this.editorloading.params.id);
				formData.append('title', _this.editorloading.params.title);
				formData.append('type', _this.editorloading.params.type);
				formData.append('jump_url', _this.editorloading.params.jump_url);
				formData.append('show_type', _this.editorloading.params.show_type);
				formData.append('share_content', _this.editorloading.params.share_content);
				formData.append('is_show', _this.editorloading.params.is_show);
			  	formData.append('image', _this.editorloading.file_pic); //提交的新增图标的文件
			  	formData.append('show_s_time', _this.editorloading.params.show_s_time);
			  	formData.append('show_e_time', _this.editorloading.params.show_e_time);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};
				axios.post(allget+'/NewContent/setContentBanner', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.editorloading.dialogShow = false;								
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '修改成功！');
							_this.getTableData();
							_this.editorloading.dialogShow = true;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
						_this.resetForm();
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		resetForm() {
			// 进行对应的重置
			var _this = this;
			_this.addNewloading = {
				dialogFormVisible: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '',
					index_position: '',
					title: '',
					type: '2',
					show_s_time: '',
					show_e_time: '',
					jump_url: '',
					page_param: '',
					show_type: '0',
					share_content:'',
					is_show:'0',
				},
			};
			_this.editorloading = {
				dialogFormVisible: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '',
					index_position: '',
					title: '',
					type: '2',
					show_s_time: '',
					show_e_time: '',
					e_time_spare: '',
					s_time_spare: '',
					jump_url: '',
					page_param: '',
					show_type: '0',
					share_content:'',
					is_show:'0',
					id:'',
				},
			};
		},
		//1上线 0下线
		setShow(index, rows, type){
			var _this = this;
			_this.listLoading = true;
			var url = '/NewContent/showContentBanner';
			var params = {
				id : rows.id,
				is_show : type
			}
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '设置成功');
						_this.getTableData();
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
					baseConfig.warningTipMsg(_this, 'error');
				});
		},
		//删除
		setDelete(index, rows){
			var _this = this;
            _this.listLoading = true;
            this.$confirm('此操作将永久删除, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                var url = '/NewContent/deleteContentBanner';
                let params = {
                    id : rows.id
                };
                axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                         _this.getTableData();
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
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getTableData();
			_this.getSort();
			_this.operate_user = store.state.user.name; 
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
.showbtn{
	position: absolute;
	left: 0;
	top: 0;
	width: 150px;
	height: 30px;
	line-height: 30px;
	text-align:center;
	font-size: 12px;
	color: #fff;
	background-color: #00b3ee;
	border: 2px solid #00b3ee;
}
.fileinput{
	position: absolute;
	left: 0;
	top: 0;
	width: 150px;
	height: 30px;
	opacity: 0;
	border: 2px solid red;
}
</style>
