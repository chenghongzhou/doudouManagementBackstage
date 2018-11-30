<template>
	<!-- 歌曲风格管理 -->
	<section>
		<el-col 
		:span="24" 
		class="toolbar" 
		style="padding-bottom:0px;">
			<el-form 
			:inline="true" 
			style="overflow:hidden;">
				<el-form-item label="歌曲风格标签">
					<el-select
						v-model="formCondition.name"
						filterable
						remote
						reserve-keyword
						clearable
						placeholder="请输入关键词"
						:remote-method="remoteMethod"
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
			:data="onePageTabData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="id" label="标签ID" min-width="50"></el-table-column>
				<el-table-column prop="create_time" label="创建时间" min-width="150"></el-table-column>
				<el-table-column prop="name" label="标签名" min-width="100"></el-table-column>
				<el-table-column prop="position" label="标签位置" min-width="100"></el-table-column>
				<el-table-column prop="sort" label="排序ID" min-width="100"></el-table-column>
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
			<el-dialog title="编辑歌曲风格标签" :visible.sync="formOne.dialogVisible">
				<el-form :model="formOne">
					<el-form-item label="ID" :label-width="formLabelWidth">
						<el-input disabled="disabled" v-model="formOne.id" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="名称" :label-width="formLabelWidth">
						<el-input v-model="formOne.name" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="排序ID" :label-width="formLabelWidth">
						<el-input v-model="formOne.sort" auto-complete="off"></el-input>
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
			<el-dialog title="新增标签" :visible.sync="formTwo.dialogVisible">
				<el-form :model="formTwo">
					<el-form-item label="名称" :label-width="formLabelWidth">
						<el-input placeholder="注意名称不要重复啦~" v-model="formTwo.name" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="排序ID" :label-width="formLabelWidth">
						<el-input v-model="formTwo.sort" auto-complete="off"></el-input>
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
				status: '',
				sort: '',
			},
			formTwo: { // 新增
				dialogVisible: false,			
				name: '',
				status: '',
				sort: '',
			},
			formCondition: {
				name: '', 
				name_list: [],
				status: '',
			},
			listLoading: false, 
			loading: false,
			tabData: [], 
			totalpage: null, 
			page: 0, 
			star: '0', 
			end: '20', 
			formLabelWidth: '120px',
		};
	},
	computed: {
		onePageTabData() {
            var _this = this;
            return _this.tabData.slice(_this.star, _this.end);
        },
	},
	methods: {
		judgeStatus(row, column) {
			return row.status==0?"上线中":row.status==1?"已下线":"";
		},
		handleCurrentChange(val) {
			var _this = this;
			_this.page = val-1;
			_this.star = (_this.page)*20;
			_this.end = _this.star+20;
		},
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.name = _this.formCondition.name;
			obj.status = _this.formCondition.status;
			return obj; 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewMusic/getSingerLabelList';
			var params = _this.searchCondition();
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						_this.tabData = res.data.data;
						_this.totalpage = res.data.data.length;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 编辑修改某一条渠道号名称
		changeOneUserData(index, rows){
			var _this = this;
			_this.formOne.id = rows.id;
			_this.formOne.name = rows.name;
			_this.formOne.sort = rows.sort;
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
			  	formData.append('sort', _this.formOne.sort);
			  	formData.append('status', _this.formOne.status);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/NewMusic/editSingerLabel', formData, config)
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
			  	formData.append('name', _this.formTwo.name);
			  	formData.append('sort', _this.formTwo.sort);
			  	formData.append('status', _this.formTwo.status);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/NewMusic/addSingerLabel', formData, config)
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
		remoteMethod(query) {
			var _this = this;
			if (query !== '') {
				_this.loading = true;
				var params = {
					name: query,
				};
				axios.get(allget+'/NewMusic/getSingerLabelList', {params: params})
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
			_this.formCondition.name = '';
			}
		}
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
