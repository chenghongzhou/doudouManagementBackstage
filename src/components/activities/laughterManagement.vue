<template>
	<!-- K歌 -->
	<section>
			<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
				<el-form :inline="true" style="overflow:hidden;" :model="formOne">
					<el-form-item label="UID">
						<el-input 
						v-model="formOne.uid"></el-input>
					</el-form-item>
					<el-form-item label="审核状态">
						<el-select v-model="formOne.status">
							<el-option label="审核中" value="0"></el-option>
							<el-option label="审核通过" value="1"></el-option>
							<el-option label="审核不通过" value="2"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item>
						<el-button 
						type="primary" 
						@click="getData">查询</el-button>
					</el-form-item>
				</el-form>
			</el-col>
			<!--数据展示列表-->
			<template>
				<el-table 
				ref="tableHeight" 
				:data="formOne.tabData" 
				border fit highlight-current-row 
				style="width:100%;" 
				:height="searchPageHeight">
					<el-table-column prop="uid" label="用户id"></el-table-column>
					<el-table-column prop="nickname" label="用户昵称"></el-table-column>
					<el-table-column prop="song_time" label="歌曲上传时间"></el-table-column>
					<el-table-column prop="charm" label="热度值"></el-table-column>
					<el-table-column prop="like" label="人气值"></el-table-column>
					<el-table-column prop="like" label="收礼数">
						<template slot-scope="scope">
							{{scope.row.charm/10}}
						</template>
					</el-table-column>
					<el-table-column prop="song" label="歌曲" min-width="200">
						<template slot-scope="scope">
							<audio 
							:src="scope.row.song" 
							controls="controls" 
							preload="none"></audio>
						</template>
					</el-table-column>
					
					<el-table-column label="操作" min-width="100">
						<template slot-scope="scope">
							<el-button 
							type="primary" 
							v-if="formOne.status == 0"
							@click="setStatus(scope.row)" 
							size="small">审核</el-button>		
							<el-button 
							v-if="formOne.status == 1 || formOne.status == 2"
							type="info"
							size="small">审核</el-button>							
						</template>
					</el-table-column>
				</el-table>
				<el-col :span="24" class="toolbar">
					<el-pagination 
					layout="total,prev,pager,next,jumper" 
					@current-change="oneHandleCurrentChange" 
					:page-size="20" :total="formOne.totalPage" 
					style="float:right;"></el-pagination>
				</el-col>
			</template>
			<el-dialog title="审核" :visible.sync="formTwo.dialogFormVisible">
				<el-form :model="formTwo">
					<el-form-item label="选择审核状态">
						<el-select v-model="formTwo.status">
							<el-option label="审核通过" value="1"></el-option>
							<el-option label="审核不通过" value="2"></el-option>
						</el-select>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="changeStatusSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="changeStatusSure(1)">确 定</el-button>
				</div>
			</el-dialog>
	</section>
</template>

<script>
import Event from './../../public_js/event.js';
import { allget } from '../../api/api';
import store from '../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			searchPageHeight: null,
			formOne: {
				uid: '',
				status: '0',
				tabData: [],
				totalPage: 1000, 
				page: 1,
			},
			formTwo: {
				status: 1,
				dialogFormVisible: false,
				uid:'',
				status:'1'
			},
			formLabelWidth: '130px',
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
			_this.formOne.page = val;
			_this.getData();
		},
		// 获取家族数据统计的数据
		getData() {
			var _this = this ;
			var url = '/NewActivity/getLaughterList';
			var params = {
				uid: _this.formOne.uid,
				page: _this.formOne.page,
				status: _this.formOne.status,
			};
			axios.get(allget+url, { params: params })
			.then((res) => {
				if(res.data.ret) {
					_this.formOne.tabData = res.data.data;
				} else {
					baseConfig.warningTipMsg(_this, res.data.msg);
				}
			}).catch((error) => {
				console.log(error);
			});
		},
		setStatus(row){
			var _this = this;
			_this.formTwo.uid = row.uid;
			_this.formTwo.dialogFormVisible = true;
		},
		changeStatusSure(index){
			var _this = this;
			if(index == 1){
				_this.listLoading = true;
				this.$confirm('此操作将立即生效, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					var url = '/NewActivity/setLaughterStatus';
					let params = {
						uid : _this.formTwo.uid,
						status : _this.formTwo.status
					};
					axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if (res.data.ret) {
							_this.getData();
							_this.formTwo.dialogFormVisible = false;
							this.$message({
								type: 'success',
								message: '成功!'
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
						message: '已取消'
					});          
				});
			}else{
				_this.formTwo.dialogFormVisible = false;
			}
		}
		
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.searchPageHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getData();
		})
	}
};
</script>

<style lang="css" scoped>
/* 引入的图片展示样式 */
.fileinput{
	float: left;
	margin-top: 8px;
}
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
.demo-ruleForm{
	width: 500px;
}
.el-tab-pane{
	height: 800px;
}
</style>
