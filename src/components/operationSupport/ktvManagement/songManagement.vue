<template>
	<!-- 歌手管理 -->
	<section>
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<el-form 
			:inline="true" 
			style="overflow:hidden;">
				<el-form-item label="歌手名">
					<el-select
						v-model="formCondition.singer_name"
						filterable
						remote
						reserve-keyword
						clearable
						placeholder="请输入关键词"
						:remote-method="remoteMethodSingerName"
						:loading="loading">
						<el-option
						v-for="item in formCondition.singer_name_list"
						:key="item.id"
						:label="item.name"
						:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="歌曲名">
					<el-select
						v-model="formCondition.name"
						filterable
						remote
						reserve-keyword
						clearable
						placeholder="请输入关键词"
						:remote-method="remoteMethodName"
						:loading="loading">
						<el-option
						v-for="item in formCondition.name_list"
						:key="item.id"
						:label="item.name"
						:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="上线状态">
					<el-select v-model="formCondition.status" placeholder="全部">
						<el-option label="全部" value=""></el-option> 
						<el-option label="上线中" value="0"></el-option> 
						<el-option label="已下线" value="1"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item>
					<el-button 
					type="primary" 
					@click="excelActivity.dialogVisible=true;">歌曲上传</el-button>
					<el-button 
					type="primary" 
					@click="formTwo.dialogVisible=true;">新增</el-button>
					<el-button 
					type="primary" 
					@click="getTableData">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			ref="tableHeight" 
			:data="tabData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="id" label="歌手ID" width="50"></el-table-column>
				<el-table-column prop="create_time" label="创建时间" width="150"></el-table-column>
				<el-table-column prop="name" label="歌曲名" width="80"></el-table-column>
				<el-table-column prop="singer_name" label="歌手名字" width="80"></el-table-column>
				<el-table-column prop="label_name" label="风格标签" width="80"></el-table-column>
				<el-table-column label="歌曲图片" width="300">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<img 
							:src="scope.row.icon"
							style="width:300px;height:200px;">
						</div>
					</template>
				</el-table-column>
				<el-table-column label="歌曲mv" width="300">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<video 
							:src="scope.row.url"
							controls="controls"
							style="width:300px;height:200px;"></video>
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="label_name" label="位置" width="50"></el-table-column>
				<el-table-column prop="total_play_num" label="播放次数" width="80"></el-table-column>
				<el-table-column prop="sort" label="排序ID" width="50"></el-table-column>
				<el-table-column prop="initial" label="首字母" width="50"></el-table-column>
				<el-table-column prop="sort" label="排序ID" width="50"></el-table-column>
				<el-table-column 
				:formatter="judgeStatus"
				prop="status" 
				label="状态" 
				width="100"></el-table-column>
				<el-table-column label="操作" min-width="150">
					<template slot-scope="scope">
						<el-button 
						type="primary" 
						@click.native.prevent="changeOneUserData(scope.$index, scope.row)" 
						size="small">编辑</el-button>
						<el-button 
						v-if="scope.row.status==0"
						type="danger" 
						@click.native.prevent="changeStatusData(scope.$index, scope.row)" 
						size="small">下线</el-button>
						<el-button 
						v-if="scope.row.status==1"
						type="success" 
						@click.native.prevent="changeStatusData(scope.$index, scope.row)" 
						size="small">上线</el-button>
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
			<el-dialog title="编辑歌手" :visible.sync="formOne.dialogVisible">
				<el-form :model="formOne">
					<el-form-item label="歌曲名" :label-width="formLabelWidth">
						<el-input v-model="formOne.name" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="歌曲图" :label-width="formLabelWidth">
						<input 
						class="filepic fileinput" 
						@change="uploading($event, 1)" 
						type="file" 
						style="display: block;">
				        <img 
						:src="formOne.icon_src" 
						style="width:300px;height:200px;margin-top:20px;"/>
					</el-form-item>
					<el-form-item label="歌曲地址" :label-width="formLabelWidth">
						<el-input v-model="formOne.url" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="歌手名" :label-width="formLabelWidth">
						<el-select
							v-model="formOne.singer_id"
							filterable
							remote
							reserve-keyword
							clearable
							placeholder="请输入关键词"
							:remote-method="remoteMethodSingerIdOne"
							:loading="loading">
							<el-option
							v-for="item in formOne.singer_id_list"
							:key="item.id"
							:label="item.name"
							:value="item.id">
							</el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="歌曲位置" :label-width="formLabelWidth">
						<el-select
							v-model="formOne.position"
							filterable
							remote
							reserve-keyword
							clearable
							placeholder="请输入关键词"
							:remote-method="remoteMethodPositionEdit"
							:loading="loading">
							<el-option
							v-for="item in formOne.position_list"
							:key="item.id"
							:label="item.name"
							:value="item.id">
							</el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="排序ID" :label-width="formLabelWidth">
						<el-input v-model="formOne.sort" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="歌曲首字母" :label-width="formLabelWidth">
						<el-input v-model="formOne.initial" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="状态" :label-width="formLabelWidth">
						<el-select v-model="formOne.status" placeholder="状态">
							<el-option label="上线" value="0"></el-option> 
							<el-option label="下线" value="1"></el-option>
						</el-select>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button @click="editorSure(0)">取 消</el-button>
					<el-button type="primary" @click="editorSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<el-dialog title="新增歌曲" :visible.sync="formTwo.dialogVisible">
				<el-form :model="formTwo">
					<el-form-item label="歌曲名" :label-width="formLabelWidth">
						<el-input v-model="formTwo.name" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="歌曲图" :label-width="formLabelWidth">
						<input 
						class="filepic fileinput" 
						@change="uploading($event, 0)" 
						type="file" 
						style="display: block;">
				        <img 
						:src="formTwo.icon_src" 
						style="width:300px;height:200px;margin-top:20px;"/>
					</el-form-item>
					<el-form-item label="歌曲地址" :label-width="formLabelWidth">
						<el-input v-model="formTwo.url" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="歌手名" :label-width="formLabelWidth">
						<el-select
							v-model="formTwo.singer_id"
							filterable
							remote
							reserve-keyword
							clearable
							placeholder="请输入关键词"
							:remote-method="remoteMethodSingerIdTwo"
							:loading="loading">
							<el-option
							v-for="item in formTwo.singer_id_list"
							:key="item.id"
							:label="item.name"
							:value="item.id">
							</el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="歌曲位置" :label-width="formLabelWidth">
						<el-select
							v-model="formTwo.position"
							filterable
							remote
							reserve-keyword
							clearable
							placeholder="请输入关键词"
							:remote-method="remoteMethodPositionAdd"
							:loading="loading">
							<el-option
							v-for="item in formTwo.position_list"
							:key="item.id"
							:label="item.name"
							:value="item.id">
							</el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="排序ID" :label-width="formLabelWidth">
						<el-input v-model="formTwo.sort" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="歌曲首字母" :label-width="formLabelWidth">
						<el-input v-model="formTwo.initial" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="状态" :label-width="formLabelWidth">
						<el-select v-model="formTwo.status" placeholder="状态">
							<el-option label="上线" value="0"></el-option> 
							<el-option label="下线" value="1"></el-option>
						</el-select>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button @click="addSure(0)">取 消</el-button>
					<el-button type="primary" @click="addSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<el-dialog title="上传" :visible.sync="excelActivity.dialogVisible">
				<el-form :model="excelActivity">
					<el-form-item label="excel表" :label-width="formLabelWidth">
						<input 
						class="filepic fileinput" 
						@change="uploading($event, 2)" 
						type="file" 
						style="display: block;">
					</el-form-item>                                                                                                                                                                                                                                                                          
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button @click="excelSure(0)">取 消</el-button>
					<el-button type="primary" @click="excelSure(1)">确 定</el-button>
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
				name: '',
				icon: '',
				icon_src: '', // 显示图片的src
				url: '',
				singer_id: '',
				singer_id_list: [],
				position: '',
				position_list: [],
				sort: '',
				initial: '',
				status: '0',
			},
			formTwo: { // 新增
				dialogVisible: false,			
				name: '',
				icon: '',
				icon_src: '', // 显示图片的src
				url: '',
				singer_id: '',
				singer_id_list: [],
				position: '',
				position_lsit: [],
				sort: '',
				initial: '',
				status: '0',
			},
			formCondition: {
				name: '', 
				name_list: [], 
				status: '',
				singer_name: '',
				singer_name_list: [],
			},
			excelActivity: {
				dialogVisible: false,
				file: '', // 提交Excel的表格的文件
			},
			listLoading: false, 
			loading: false,
			tabData: [], 
			totalpage: 1000, 
			page: 0, 
			star: '0', 
			end: '20', 
			formLabelWidth: '120px',
		};
	},
	methods: {
		judgeSex(row, column) {
			return row.sex==1?"男":row.sex==0?"女":row.sex==2?"无":"";
		},
		judgeStatus(row, column) {
			return row.status==0?"上线中":row.status==1?"已下线":"";
		},
		handleCurrentChange(val) {
			var _this = this;
			_this.page = val-1;
			_this.getTableData();
		},
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.name = _this.formCondition.name;
			obj.status = _this.formCondition.status;
			obj.singer_name = _this.formCondition.singer_name;
			obj.page = _this.page;
			return obj; 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewMusic/getMusicList';
			var params = _this.searchCondition();
			axios.get(allget+url, {params: params})
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
		},
		// 得到上传文件type(0->新增，1->编辑修改)
        uploading(event, type) {
			var _this = this;
			if(type==0) { // 活动新增
				_this.formTwo.icon = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.formTwo.icon_src = windowURL.createObjectURL(event.target.files[0]);
			} else if(type==1) { // 活动编辑修改
				_this.formOne.icon = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.formOne.icon_src = windowURL.createObjectURL(event.target.files[0]);
			} else if(type==2) { // 上传文件
				_this.excelActivity.file = event.target.files[0];
			}
      	}, 
		// 编辑修改某一条渠道号名称
		changeOneUserData(index, rows){
			var _this = this;
			_this.formOne.id = rows.id;
			_this.formOne.name = rows.name;
			_this.formOne.icon_src = rows.icon;
			_this.formOne.url = rows.url;
			_this.formOne.singer_id = rows.singer_id;
			_this.formOne.singer_id_list = [
				{
					name: rows.singer_name,
					id: rows.singer_id,
				}
			];
			_this.formOne.position = rows.position;
			_this.formOne.position_list = [
				{
					name: rows.label_name,
					id: rows.position,
				}
			];
			_this.formOne.sort = rows.sort;
			_this.formOne.initial = rows.initial;
			_this.formOne.status = rows.status;
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
			  	formData.append('name', _this.formOne.name);
			  	formData.append('icon', _this.formOne.icon);
			  	formData.append('url', _this.formOne.url);
			  	formData.append('singer_id', _this.formOne.singer_id);
			  	formData.append('position', _this.formOne.position);
			  	formData.append('sort', _this.formOne.sort);
			  	formData.append('initial', _this.formOne.initial);
			  	formData.append('status', _this.formOne.status);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/NewMusic/editMusic', formData, config)
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
				_this.resetForm();
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('name', _this.formTwo.name);
			  	formData.append('icon', _this.formTwo.icon);
			  	formData.append('url', _this.formTwo.url);
			  	formData.append('singer_id', _this.formTwo.singer_id);
			  	formData.append('position', _this.formTwo.position);
			  	formData.append('sort', _this.formTwo.sort);
			  	formData.append('initial', _this.formTwo.initial);
			  	formData.append('status', _this.formTwo.status);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/NewMusic/addMusic', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.formTwo.dialogVisible = false;									
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							_this.getTableData();
							_this.resetForm();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 上下线的改变接口
		changeStatusData(index, rows) {
			var _this = this;
			var id = rows.id;	
			var status = rows.status;	
			status = status==0?1:status==1?0:"";
			_this.listLoading = true;
			var url = '/NewMusic/upDownMusic';
			var params = {
				id: id,
				status: status,
			};
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						if(params.status==0) {
							baseConfig.successTipMsg(_this, '上线成功啦~');
						} else if(params.status==1) {
							baseConfig.successTipMsg(_this, '已下线成功');
						}
						_this.getTableData();
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}

				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 上传excel表
		excelSure(type) {
			var _this = this;
			if(type==0) {
				_this.excelActivity.dialogVisible = false;
				_this.excelActivity.file = '';
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('file', _this.excelActivity.file);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/NewMusic/addMultiMusic', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.excelActivity.dialogVisible = false;									
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							_this.getTableData();
							_this.excelActivity.file = '';
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 新增的时候加上一个置空的选项
		resetForm() {
			var _this = this;
			_this.formTwo = {
				name: '',
				icon: '',
				icon_src: '',
				url: '',
				singer_id: '',
				singer_id_list: [],
				position: '',
				sort: '',
				initial: '',
				status: '0',
				dialogVisible: false,	
			};
			baseConfig.normalTipMsg(_this, '取消、确定都将新增列表选项恢复默认~');
		},
		// 进行对应的歌手选择框选择
		remoteMethodSingerName(query) {
			var _this = this;
			if (query !== '') {
				_this.loading = true;
				var params = {
					name: query,
				};
				axios.get(allget+'/NewMusic/getSingerList', {params: params})
					.then((res) => {
						_this.loading = false;
						_this.formCondition.singer_name_list = [];
						res.data.data.forEach((item, index) => {
							_this.formCondition.singer_name_list.push({
								name: item.name,
								id: item.name,
							});
						});
					})
					.catch((err) => {
						_this.loading = false;
						console.log(err);
					});
			} else {
				_this.formCondition.singer_name_list = [];
			}
		},
		// 进行对应的歌曲选择框选择
		remoteMethodName(query) {
			var _this = this;
			if (query !== '') {
				_this.loading = true;
				var params = {
					name: query,
				};
				axios.get(allget+'/NewMusic/getMusicList', {params: params})
					.then((res) => {
						_this.loading = false;
						_this.formCondition.name_list = [];
						res.data.data.forEach((item, index) => {
							_this.formCondition.name_list.push({
								name: item.name,
								id: item.name,
							});
						});
					})
					.catch((err) => {
						_this.loading = false;
						console.log(err);
					});
			} else {
				_this.formCondition.name_list = [];
			}
		},
		// 进行歌手对应的id的选择
		remoteMethodSingerIdTwo(query) {
			var _this = this;
			if (query !== '') {
				_this.loading = true;
				var params = {
					name: query,
				};
				axios.get(allget+'/NewMusic/getSingerList', {params: params})
					.then((res) => {
						_this.loading = false;
						_this.formTwo.singer_id_list = [];
						res.data.data.forEach((item, index) => {
							_this.formTwo.singer_id_list.push({
								name: item.name,
								id: item.id,
							});
						});
					})
					.catch((err) => {
						_this.loading = false;
						console.log(err);
					});
			} else {
				_this.formTwo.singer_id_list = [];
			}
		},
		remoteMethodSingerIdOne(query) {
			var _this = this;
			if (query !== '') {
				_this.loading = true;
				var params = {
					name: query,
				};
				axios.get(allget+'/NewMusic/getSingerList', {params: params})
					.then((res) => {
						_this.loading = false;
						_this.formOne.singer_id_list = [];
						res.data.data.forEach((item, index) => {
							_this.formOne.singer_id_list.push({
								name: item.name,
								id: item.id,
							});
						});
					})
					.catch((err) => {
						_this.loading = false;
						console.log(err);
					});
			} else {
				_this.formOne.singer_id_list = [];
			}
		},
		remoteMethodPositionAdd(query) {
			var _this = this;
			if (query !== '') {
				_this.loading = true;
				var params = {
					name: query,
				};
				axios.get(allget+'/NewMusic/getSingerLabelList', {params: params})
					.then((res) => {
						_this.loading = false;
						_this.formTwo.position_list = [];
						res.data.data.forEach((item, index) => {
							_this.formTwo.position_list.push({
								name: item.name,
								id: item.position,
							});
						});
					})
					.catch((err) => {
						_this.loading = false;
						console.log(err);
					});
			} else {
				_this.formTwo.position_list = [];
			}
		},
		remoteMethodPositionEdit(query) {
			var _this = this;
			if (query !== '') {
				_this.loading = true;
				var params = {
					name: query,
				};
				axios.get(allget+'/NewMusic/getSingerLabelList', {params: params})
					.then((res) => {
						_this.loading = false;
						_this.formOne.position_list = [];
						res.data.data.forEach((item, index) => {
							_this.formOne.position_list.push({
								name: item.name,
								id: item.position,
							});
						});
					})
					.catch((err) => {
						_this.loading = false;
						console.log(err);
					});
			} else {
				_this.formOne.position_list = [];
			}
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
</style>
