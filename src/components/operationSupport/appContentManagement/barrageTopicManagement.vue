<template>
	<!-- 弹幕话题管理 -->
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
					<span>展示位置</span>
					<el-select v-model="formOne.position">
						<el-option label="全部" value=""></el-option>
						<el-option label="随机匹配话题" value="0"></el-option>
						<el-option label="抢聊话题" value="1"></el-option>
						<el-option label="刺激话题" value="2"></el-option>
						<el-option label="组组匹配推送话题" value="3"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item>
					<el-button 
					type="primary" 
					@click="dialogFormVisible=true;">添加弹幕</el-button>
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
				<el-table-column prop="sort" label="弹幕序号" width="100" sortable ></el-table-column>
				<el-table-column prop="content" label="弹幕内容" min-width="400" sortable ></el-table-column>
				<el-table-column label="弹幕位置" width="400" sortable >
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.position==0">随机匹配话题</p>
							<p v-else-if="scope.row.position==1">抢聊话题</p>
							<p v-else-if="scope.row.position==2">刺激话题</p>
							<p v-else-if="scope.row.position==3">组组匹配推送话题</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="操作" width="200">
					<template slot-scope="scope">
						<el-button 
						type="primary" 
						@click.native.prevent="deleteOneUserData(scope.$index, scope.row)" 
						size="small">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
			<!--工具条-->
			<el-col :span="24" class="toolbar">
				<el-pagination 
				layout="total,prev,pager,next,jumper" 
				@current-change="handleCurrentChange" 
				:page-size="20" 
				:total="totalpage" 
				style="float:right;"></el-pagination>
			</el-col>
		</template>
		<!-- 新增--对应的dialog -->
		<el-dialog title="添加弹幕" :visible.sync="dialogFormVisible">
			<el-form :model="formTwo">
				<el-form-item label="弹幕内容" :label-width="formLabelWidth">
					<el-input v-model="formTwo.content" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="展示位置" :label-width="formLabelWidth">
					<el-select v-model="formTwo.position">
						<el-option label="随机匹配话题" value="0"></el-option>
						<el-option label="抢聊话题" value="1"></el-option>
						<el-option label="刺激话题" value="2"></el-option>
						<el-option label="组组匹配推送话题" value="3"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="弹幕序号" :label-width="formLabelWidth">
					<el-input v-model="formTwo.sort" auto-complete="off"></el-input>
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
				position: '', 
			},
			formTwo: {
				content: '',
				position: '0',
				sort: '0',
			},
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
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.position = _this.formOne.position;
			for(var key in obj) {
				if(key=='position') {

				} else {
					if(obj[key]=='') {
						baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
						return null;
					}
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
				let formData = new FormData();
				formData.append('content', encodeURIComponent(_this.formTwo.content));
				formData.append('position', _this.formTwo.position);
				formData.append('sort', _this.formTwo.sort);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};
				if(formData.get('content')!=''&&formData.get('position')!=''&&formData.get('sort')!='') {
					axios.post(allget+'/GlobalSet/addScreen', formData, config)
						.then((res) => {
							_this.listLoading = false;	
							_this.dialogFormVisible = false;								
							if(res.data.ret) {	
								baseConfig.successTipMsg(_this, '新增成功！');
								_this.getTableData();
							} else {
								baseConfig.warningTipMsg(_this, res.data.msg);						
							}
						}).catch((err) => {
							console.log(err);
						});
				} else {
					_this.listLoading = false; 
					baseConfig.warningTipMsg(_this, '新增的弹幕内容、排序不能为空！');
				}
			} 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/GlobalSet/findScreen';
			var params = _this.searchCondition();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						// 对数据进行处理，将最大的序列号进行加1操作保存
						for(var i=0; i<res.data.data.length; i++) { 
							res.data.data[i].content = (res.data.data[i].content);
							if(_this.formTwo.sort<=res.data.data[i].sort) {
								_this.formTwo.sort = res.data.data[i].sort-0+1;
							}
						}
						_this.totalpage = res.data.data.length;
						_this.tabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				})
			}
		},
		// 删除某一个已经存在确定的用户
		deleteOneUserData(index, rows) {
			var _this = this;
			index = index + (_this.page-1)*20; 
			var id = rows.id; 	
			// 下面的操作主要是为了进行将删除的用户调用删除接口进行删除
			_this.listLoading = true;
			let formData = new FormData();
			formData.append('id', id);
			let config = {
				headers: {
					'Content-Type': 'multipart/form-data'
				}
			};
			if(formData.get('id')!='') {
				axios.post(allget+'/GlobalSet/delScreen', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '删除成功！');
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
				baseConfig.warningTipMsg(_this, '删除的id不能为空');
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
