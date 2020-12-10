<template>
	<!-- 密令 -->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 密令管理 -->
			<el-tab-pane 
			label="密令管理" 
			name="first" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formOne">
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
							<el-button 
							type="primary" 
							@click="getTableFind">查询</el-button>
						</el-form-item>
                        <el-form-item>
							<el-button 
							type="primary" 
							@click="bannerNewloading.dialogShow=true;">生成密令</el-button>							
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="onePageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="create_time" label="日期"></el-table-column>
						<el-table-column prop="create_time" label="生成时间"></el-table-column>
						<el-table-column prop="name" label="物品"></el-table-column>
                        <el-table-column prop="type" label="物品类型">
                            <template slot-scope="scope">
                                <div slot="reference" class="name-wrapper">
                                    <p v-if="scope.row.type == 1">背包礼物</p>
                                    <p v-else-if="scope.row.type==2">装扮</p>
                                    <p v-else-if="scope.row.type==3">座驾</p>
                                    <p v-else-if="scope.row.type==4">称号</p>
                                    <p v-else-if="scope.row.type==5">道具</p>
                                </div>
                            </template>
                        </el-table-column>
						<el-table-column prop="num" label="数量"></el-table-column>
						<el-table-column prop="secret_order" label="密令内容"></el-table-column>
						<el-table-column prop="limit_num" label="限领数量"></el-table-column>
                        <el-table-column prop="receive_num" label="实际领取"></el-table-column>
                        <el-table-column prop="admin" label="领取人"></el-table-column>
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
			</el-tab-pane>
			<!-- 领取详情 -->
			<el-tab-pane 
			label="领取详情" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item>
							<div class="block">
								<span class="registerTime">日期</span>
								<el-date-picker 
								v-model="formTwo.choiceDate" 
								type="daterange" 
								range-separator=" 至 " 
								placeholder="选择日期范围"></el-date-picker>
							</div>
						</el-form-item>
						<el-form-item label="UID">
							<el-input 
							v-model="formTwo.uid" 
							auto-complete="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableManage">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="twoPageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="date" label="日期"></el-table-column>
						<el-table-column prop="uid" label="uid"></el-table-column>
						<el-table-column prop="nickname" label="昵称"></el-table-column>
						<el-table-column prop="secret_order" label="密令"></el-table-column>
						<el-table-column prop="receive_time" label="领取时间"></el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="twoHandleCurrentChange" 
						:page-size="20" 
						:total="formTwo.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- banner条新增 -->
			<el-dialog 
			title="密令生成" 
			:visible.sync="bannerNewloading.dialogShow">
				<el-form :model="bannerNewloading.params">
                    <el-form-item label="物品类型" :label-width="formLabelWidth">
                        <el-select v-model="bannerNewloading.params.type" @change="checkType">
                            <el-option label="背包礼物" value="1"></el-option>
                            <el-option label="装扮" value="2"></el-option>
                            <el-option label="座驾" value="3"></el-option>
                            <el-option label="称号" value="4"></el-option>
                            <el-option label="道具" value="5"></el-option>
                        </el-select>
                        </el-form-item> 
                        <el-form-item label="物品" :label-width="formLabelWidth">
                            <el-select v-model="bannerNewloading.params.prop_id" @change="checkPic()">
                                <el-option 
                                :label="item.name" 
                                :value="item.id"
                                :key="index"
                                v-for="(item,index) in wp">
                                </el-option>
                            </el-select>
                        </el-form-item> 
                        <el-form-item label="图片" :label-width="formLabelWidth">
                            <img 
                            :src="bannerNewloading.params.icon" 
                            style="width:200px;height:auto;margin-left:200px;"/>
                        </el-form-item>
                        <el-form-item label="数量" :label-width="formLabelWidth">
                            <el-input 
                            v-model="bannerNewloading.params.num" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="限领人数" :label-width="formLabelWidth">
                            <el-input 
                            v-model="bannerNewloading.params.limit_num" 
                            auto-complete="off"></el-input>
                        </el-form-item>
                        <el-form-item label="密令内容" :label-width="formLabelWidth">
                            <el-input 
                            v-model="bannerNewloading.params.secret_order" 
                            auto-complete="off"></el-input>
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
		</el-tabs>
	</section>
</template>

<script>
import Event from './../../public_js/event.js';
import store from '../../vuex/store';
import axios from 'axios';
import { allget } from '../../api/api.js';
export default {
	data() {
		return {
			tableHeight: null, 
            tabSearchHeight: null,
            wp:[],
			formOne: {
				choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				family_id: '',
				TabData: [],
				TotalPage: null, 
				Page: 1, 
				star: '0',
				end: '20',
			},
			formTwo: {
				choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				uid: '',
				TabData: [], 
				TotalPage: null, 
				Page: 1, 
				star: '0',
				end: '20',
			},
			bannerNewloading: {
				dialogShow: false,
				params: {
                    type:'1',
                    prop_id:'',
                    num:'',
                    limit_num:'',
                    secret_order:'',
                    icon:''
				},
			},
			listLoading: false, 
			tabActiveName: 'first', // 设置为tab切换栏的选中不同的状态(first、second)
			formLabelWidth: '130px', 
		};
	},
	computed:{
		onePageData() {
			var _this = this;
			return _this.formOne.TabData.slice(_this.formOne.star, _this.formOne.end);
		},
		twoPageData() {
			var _this = this;
			return _this.formTwo.TabData.slice(_this.formTwo.star, _this.formTwo.end);
		},
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
			_this.formOne.Page = val-1;
			_this.formOne.start_date = (_this.formOne.Page)*20;
			_this.formOne.end_date = _this.formOne.star+20;
		},
		twoHandleCurrentChange(val) {
			var _this = this;
			_this.formTwo.Page = val-1;
			_this.formTwo.start_date = (_this.formTwo.Page)*20;
			_this.formTwo.end_date = _this.formTwo.star+20;
		},
		searchConditionFind() {
			var _this = this;
            var obj = {};
           // obj.formTwo.Page = val-1;
			obj.start_date = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.end_date = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			return obj; 
		},
		// 获取家族数据统计的数据
		getTableFind() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewSecretOrder/getSecretOrder';
			var params = _this.searchConditionFind();
			if(params==null) {
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							_this.formOne.TotalPage = res.data.data.secret_order_list.length; 
							_this.formOne.TabData = res.data.data.secret_order_list;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 在列表中进行搜索条件
		searchConditionManage() {
			var _this = this;
			var obj = {};
			obj.start_date = baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0);
			obj.end_date = baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0);
			obj.uid = _this.formTwo.uid;
			return obj;
		},
		// 获取banner管理的数据
		getTableManage() {
			var _this = this ;
			var url = '/NewSecretOrder/getUserSecretOrder';
			var params = _this.searchConditionManage();
			axios.get(allget+url, { params: params })
            .then((res) => {
                _this.listLoading = false;	
                if(res.data.ret) {
                    _this.formTwo.TotalPage = res.data.data.stat_list.length; 
                    _this.formTwo.TabData = res.data.data.stat_list;
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg); 
                }
            })
            .catch((err) => {
                console.log(err);
            });
		},
		// banner条新增
		addBannerSure(type) {
			var _this = this;
			if(type==0) {
				// console.log('点击了取消按钮');
				_this.bannerNewloading.dialogShow = false;
			} else if(type==1) {
				// console.log('点击了确认按钮');
				_this.listLoading = true;
				// 进行添加操作
				let formData = new FormData();
				formData.append('type', _this.bannerNewloading.params.type);
				formData.append('prop_id', _this.bannerNewloading.params.prop_id);
				formData.append('num', _this.bannerNewloading.params.num);
				formData.append('limit_num', _this.bannerNewloading.params.limit_num);
                formData.append('secret_order', _this.bannerNewloading.params.secret_order);
                formData.append('admin', store.state.user.name);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/NewSecretOrder/addSecretOrder', formData, config)
					.then((res) => {
						console.log(res.data);
						_this.listLoading = false;	
						_this.bannerNewloading.dialogShow = false;								
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							_this.getTableFind();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
        },
        checkType(){
			var _this = this;
			_this.wp = [];
			let url = "/NewEgg/getGiftOrPropList";
            let params = {
                type: _this.bannerNewloading.params.type
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        _this.wp = res.data.data;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.error(err);
                });
        },
        checkPic(){
			var _this = this;
			var id = _this.bannerNewloading.params.prop_id;
			_this.wp.forEach((item) => {
				if(item.id == id){
					_this.bannerNewloading.params.icon = item.icon;
				}
			});
		},
		// 顶部tab进行页面的切换
		handleClick(tab, event) {
			var _this = this;
			// console.log(tab.label);
		},
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(tabSearchPageHeight);
			_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight);
			_this.getTableFind();
			_this.getTableManage();
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
