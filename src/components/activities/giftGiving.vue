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
					@click="bannerNewloading.dialogShow=true;">赠送</el-button>
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
				<el-table-column prop="time" label="时间" width="150"></el-table-column>
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
				<el-form-item label="用户Uid" :label-width="formLabelWidth">
					<el-input v-model="bannerNewloading.params.uid"></el-input>
				</el-form-item>
				<el-form-item label="赠送奖励" :label-width="formLabelWidth">
					<el-select 
                    v-model="bannerNewloading.params.reward_id" 
                    placeholder="请选择">
                        <el-option 
                        v-for="(item, key) of bannerNewloading.params.reward_id_arr" 
                        :key="key" 
                        :label="item.name" 
                        :value="item.id"></el-option>
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
				TotalPage: 1000, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			bannerNewloading: {
				dialogShow: false,
				params: {
					uid: '',
					operation_name: '',
					reward_id: '',
					reward_id_arr: [
						{id: '633', type: '2', name: '礼物-好评专属X1', num: '1', },
						{id: '400012', type: '4', name: '道具-30元充值优惠券X1', num: '1', },
						{id: '400007', type: '4', name: '道具-偷听大卡X1', num: '1', },
						{id: '400020', type: '4', name: '道具-通话大卡X1', num: '1', },
						{id: '900059', type: '5', name: '装扮-赞X3', num: '3', },
						{id: '900014', type: '5', name: '装扮-鹿角X5', num: '5', },
						{id: '110002', type: '6', name: '座驾-小马X3', num: '3', },
					],
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
			var url = '/Activity/justGiveSomeThingRecord';
			var params = {
				page: _this.formOne.Page,
			};
			if(params==null) {
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							_this.formOne.TabData = [];
							for(var key in res.data) {
								if(key=='ret') {} else if(key=='msg') {} else {
									// 其它的信息可以直接存储起来
									var times = new Date(res.data[key]*1000);
									var timesStr = times.getFullYear()+'-'+
										((times.getMonth()-0+1)<10?'0'+(times.getMonth()-0+1):(times.getMonth()-0+1))+'-'+
										(times.getDate()<10?'0'+times.getDate():times.getDate())+' '+
										(times.getHours()<10?'0'+times.getHours():times.getHours())+':'+
										(times.getMinutes()<10?'0'+times.getMinutes():times.getMinutes())+':'+
										(times.getSeconds()<10?'0'+times.getSeconds():times.getSeconds());
									_this.formOne.TabData.push({
										time: timesStr,
										content: key,
									});
								}
							}
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
				console.log(_this.bannerNewloading.params.reward_id);
				_this.bannerNewloading.params.reward_id_arr.forEach((item, index) => {
					if(item.id==_this.bannerNewloading.params.reward_id) {
						console.log(item)
						formData.append('type', item.type);	
						formData.append('reward_id', item.id);	
						formData.append('num', item.num);	
					}
				})
				formData.append('uid', _this.bannerNewloading.params.uid);
				formData.append('operation_name', _this.bannerNewloading.params.operation_name);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/Activity/justGiveSomeThing', formData, config)
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
            _this.bannerNewloading.params.operation_name = store.state.user.name;
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
