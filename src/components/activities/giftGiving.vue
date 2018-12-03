<template>
	<!-- 好评送礼 -->
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
					<el-button 
					type="primary" 
					@click="getTable">新增</el-button>
					<el-button 
					type="primary" 
					@click="getTable">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			ref="tableHeight" 
			:data="formOne.TabData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="time" label="时间" min-width="200"></el-table-column>
				<el-table-column prop="content" label="内容" min-width="200"></el-table-column>
			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
				layout="total,prev,pager,next,jumper" 
				@current-change="oneHandleCurrentChange" 
				:page-size="20" 
				:total="formOne.TotalPage" 
				style="float:right;"></el-pagination>
			</el-col>
		</template>
		<el-dialog 
		title="活动新增" 
		:visible.sync="bannerNewloading.dialogShow">
			<el-form :model="bannerNewloading.params">
				<el-form-item label="主活动顶部图片" :label-width="formLabelWidth">
					<el-input v-model="bannerNewloading.params.banner" auto-complete="off"></el-input>
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
	</section>
</template>

<script>
import Event from './../../public_js/event.js';
import store from '../../vuex/store';
import axios from 'axios';
import { allget, wechatget } from '../../api/api.js';
export default {
	data() {
		return {
			tableHeight: null, 
			formOne: {
				choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				family_id: '',
				TabData: [],
				TotalPage: null, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			bannerNewloading: {
				dialogShow: false,
				params: {
					choiceDate: '',
					family_id: '',
					theme: '',
					content: ['', '', ''],
					banner: '',
					message_banner: '',
					pop_banner: '',
				},
			},
			listLoading: false, 
			tabActiveName: 'second', // 设置为tab切换栏的选中不同的状态(first、second)
			formLabelWidth: '130px', 
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
			_this.formOne.Page = val-1;
			_this.getTable();
		},
		// 获取家族数据统计的数据
		getTable() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/HManage/justGiveSomeThingRecord';
			var params = {
				page: _this.formOne.Page,
			};
			if(params==null) {
				_this.listLoading = false; 
			} else {
				axios.get('http://test-app-h.dianliaoapp.com'+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							console.log(res);
							_this.formOne.TotalPage = res.data.data.length; 
							_this.formOne.TabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// banner条新增
		addBannerSure(type) {
			var _this = this;
			if(type==0) {
				_this.bannerNewloading.dialogShow = false;
			} else if(type==1) {
				_this.listLoading = true;
				// 进行添加操作
				let formData = new FormData();
				var start_time = baseConfig.changeDateTime(_this.bannerNewloading.params.choiceDate[0], 1);
				var end_time = baseConfig.changeDateTime(_this.bannerNewloading.params.choiceDate[1], 1);
				for(var i=0; i<_this.bannerNewloading.params.content.length; i++) {
					_this.bannerNewloading.params.content[i] = decodeURIComponent(_this.bannerNewloading.params.content[i]);
				}
				var content = JSON.stringify(_this.bannerNewloading.params.content);
				formData.append('start_time', start_time);
				formData.append('end_time', end_time);
				formData.append('theme', _this.bannerNewloading.params.theme);
				formData.append('family_id', _this.bannerNewloading.params.family_id);
				formData.append('content', content);
				formData.append('banner', _this.bannerNewloading.params.banner);
				formData.append('pop_banner', _this.bannerNewloading.params.pop_banner);
				formData.append('message_banner', _this.bannerNewloading.params.message_banner);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/Activity/addFamilyThemeActivity', formData, config)
					.then((res) => {
						console.log(res.data);
						_this.listLoading = false;	
						_this.bannerNewloading.dialogShow = false;								
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							_this.getTable();
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
			_this.getTable();
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
.demo-ruleForm{
	width: 500px;
}
.el-tab-pane{
	height: 800px;
}
</style>
