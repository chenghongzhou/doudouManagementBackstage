<template>
	<!-- 超级转盘（新），房间数据、个人数据 -->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 房间数据 -->
			<el-tab-pane 
			label="房间数据" 
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
                            <span>等级</span>
                            <el-select style="width: 120px;" v-model="formOne.level">
                                <el-option label="等级一" value="1"></el-option>
                                <el-option label="等级二" value="2"></el-option>
                                <el-option label="等级三" value="3"></el-option>
                            </el-select>
                        </el-form-item>
						<el-form-item label="房间ID">
							<el-input 
							v-model="formOne.room_id" 
							auto-complete="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableFind">查询</el-button>
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
						<el-table-column prop="date" label="日期" width="200"></el-table-column>
						<el-table-column prop="num" label="次数" min-width="100"></el-table-column>
						<el-table-column prop="gain" label="收益" min-width="100"></el-table-column>
						<el-table-column prop="cost" label="支出" min-width="100"></el-table-column>
						<el-table-column prop="one" label="礼物一" min-width="100"></el-table-column>
						<el-table-column prop="two" label="礼物二" min-width="100"></el-table-column>
						<el-table-column prop="three" label="礼物三" min-width="100"></el-table-column>
						<el-table-column prop="four" label="礼物四" min-width="100"></el-table-column>
						<el-table-column prop="five" label="礼物五" min-width="100"></el-table-column>
						<el-table-column prop="six" label="礼物六" min-width="100"></el-table-column>
						<el-table-column prop="seven" label="礼物七" min-width="100"></el-table-column>
						<el-table-column prop="eight" label="礼物八" min-width="100"></el-table-column>
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
			<!-- 个人数据 -->
			<el-tab-pane 
			label="个人数据" 
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
						<el-form-item label="用户ID">
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
					:data="formTwo.Tabdata" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="date" label="日期" width="200"></el-table-column>
						<el-table-column prop="num" label="次数" min-width="80"></el-table-column>
						<el-table-column prop="gain" label="收益" min-width="80"></el-table-column>
						<el-table-column prop="cost" label="支出" min-width="80"></el-table-column>
						<el-table-column prop="one" label="礼物一" min-width="80"></el-table-column>
						<el-table-column prop="two" label="礼物二" min-width="80"></el-table-column>
						<el-table-column prop="three" label="礼物三" min-width="80"></el-table-column>
						<el-table-column prop="four" label="礼物四" min-width="80"></el-table-column>
						<el-table-column prop="five" label="礼物五" min-width="80"></el-table-column>
						<el-table-column prop="six" label="礼物六" min-width="80"></el-table-column>
						<el-table-column prop="seven" label="礼物七" min-width="80"></el-table-column>
						<el-table-column prop="eight" label="礼物八" min-width="80"></el-table-column>
						<el-table-column prop="nine" label="礼物九" min-width="80"></el-table-column>
						<el-table-column prop="ten" label="礼物十" min-width="80"></el-table-column>
						<el-table-column prop="eleven" label="礼物十一" min-width="80"></el-table-column>
						<el-table-column prop="twelve" label="礼物十二" min-width="80"></el-table-column>
						<el-table-column prop="thirteen" label="礼物十三" min-width="80"></el-table-column>
						<el-table-column prop="fourteen" label="礼物十四" min-width="80"></el-table-column>
						<el-table-column prop="fifteen" label="礼物十五" min-width="80"></el-table-column>
						<el-table-column prop="sixteen" label="礼物十六" min-width="80"></el-table-column>
						<el-table-column prop="seveteen" label="礼物十七" min-width="80"></el-table-column>
						<el-table-column prop="eighteen" label="礼物十八" min-width="80"></el-table-column>
						<el-table-column prop="nineteen" label="礼物十九" min-width="80"></el-table-column>
						<el-table-column prop="twenty" label="礼物二十" min-width="80"></el-table-column>
						<el-table-column prop="twentyone" label="礼物二一" min-width="80"></el-table-column>
						<el-table-column prop="twentytwo" label="礼物二二" min-width="80"></el-table-column>
						<el-table-column prop="twentythree" label="礼物二三" min-width="80"></el-table-column>
						<el-table-column prop="twentyfour" label="礼物二四" min-width="80"></el-table-column>
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
			formOne: {
                choiceDate: [new Date()-7*24*60*60*1000, new Date()],
                level: '1',
				room_id: '',
				TabData: [],
				TotalPage: null, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			formTwo: {
				choiceDate: [new Date()-7*24*60*60*1000, new Date()],
				uid: '',
				TabData: [], 
				TotalPage: null, 
				Page: 0, 
				star: '0',
				end: '20',
			},
			listLoading: false, 
			tabActiveName: 'second',
			formLabelWidth: '130px', 
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
            _this.formOne.Page = val-1;
            _this.getTableFind();
		},
		twoHandleCurrentChange(val) {
			var _this = this;
            _this.formTwo.Page = val-1;
            _this.getTableManage();
		},
		searchConditionFind() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.room_id = _this.formOne.room_id;
			obj.page = _this.formOne.Page;
			return obj; 
		},
		// 获取家族数据统计的数据
		getTableFind() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getTurnTableGameRoomData';
			var params = _this.searchConditionFind();
			if(params==null) {
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
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
		// 在列表中进行搜索条件
		searchConditionManage() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0);
			obj.uid = _this.formTwo.uid;
			obj.page = _this.formTwo.Page;
			return obj;
		},
		// 获取banner管理的数据
		getTableManage() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewActivity/getTurnTableGameOwnerData';
			var params = _this.searchConditionManage();
			if(params==null) { 
				_this.listLoading = false; 
			} else {
                axios.get(allget+url, { params: params })
					.then((res) => {
						_this.listLoading = false;	
						if(res.data.ret) {
							_this.formTwo.TotalPage = res.data.data.length; 
							_this.formTwo.TabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
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
			// _this.getTableFind();
			// _this.getTableManage();
		})
	}
};
</script>

<style lang="css" scoped>
.demo-ruleForm{
	width: 500px;
}
.el-tab-pane{
	height: 800px;
}
</style>
