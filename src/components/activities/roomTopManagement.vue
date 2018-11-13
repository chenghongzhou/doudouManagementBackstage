<template>
	<!-- 房间置顶管理（测试服） -->
	<!-- dom结构内容 -->
	<section>
		<!-- 工具条/头部的搜索条件搜索 -->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" style="overflow: hidden;" :model="formOne">
				<el-form-item>
					<div class="block">
						<span class="registerTime">日期</span>
						<el-date-picker v-model="formOne.choiceDate" type="daterange" range-separator=" 至 " placeholder="选择日期范围"></el-date-picker>
					</div>
				</el-form-item>
				<el-form-item>
					<span>置顶状态</span>
					<el-select style="width: 120px;" v-model="formOne.type">
						<el-option label="全部" value="0"></el-option>
						<el-option label="即将开始" value="1"></el-option>
						<el-option label="进行中" value="2"></el-option>
						<el-option label="已结束" value="3"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item>
					<span>置顶类型</span>
					<el-select style="width: 120px;" v-model="formOne.choice">
						<el-option label="精选" value="1"></el-option>
						<el-option label="推荐" value="2"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item>
					<el-input placeholder="房间ID" style="width: 120px;" v-model="formOne.room_id" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item>
					<el-input placeholder="房间名称" style="width: 120px;" v-model="formOne.room_name" auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item>
					<el-button type="primary" @click="dialogFormVisible=true;">添加房间置顶计划</el-button>
					<el-button type="primary" @click="getTableData">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<!--用户的数据展示列表-->
		<template>
			<el-table ref="tableHeight" :data="onePageTabData" border fit highlight-current-row v-loading="listLoading" style="width: 100%;" :height="tableHeight">
				<el-table-column prop="id" label="ID" width="80" sortable ></el-table-column>
				<el-table-column prop="room_id" label="家族ID" ></el-table-column>
				<el-table-column prop="room_name" label="家族名称" ></el-table-column>
				<el-table-column prop="choice" label="置顶类型" >
					<template slot-scope="scope">
						<p v-if="scope.row.choice == 1">精选</p>
						<p v-else-if="scope.row.choice == 2">推荐</p>
					</template>
				</el-table-column>
				<el-table-column prop="position" label="位置" width="80" sortable ></el-table-column>
				<el-table-column prop="start_time" label="开始时间" min-width="100" ></el-table-column>
				<el-table-column prop="end_time" label="结束时间" min-width="100" ></el-table-column>
				<el-table-column prop="create_time" label="操作时间" min-width="100" ></el-table-column>
				<el-table-column label="状态" width="150" >
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.status==0 && (new Date(scope.row.end_time) <= new Date()) ">已结束</p>
							<p v-else-if="scope.row.status==0 && (new Date(scope.row.end_time) > new Date()) ">取消置顶</p>
							<p v-else-if="scope.row.status==1">置顶</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column label="操作" width="150">
					<template slot-scope="scope">
						<p v-if="scope.row.status=='1' && (new Date(scope.row.end_time) <= new Date())"  size="small" >已结束</p>
						<el-button v-else-if="scope.row.status=='1' && (new Date(scope.row.end_time) > new Date())" type="primary" size="small" @click.native.prevent="cancelTop(scope.row)">取消置顶</el-button>
						<el-button v-else-if="scope.row.status=='0'" plain size="small">已取消</el-button>
					</template>
				</el-table-column>
			</el-table>
			<!--工具条-->
			<el-col :span="24" class="toolbar">
				<el-pagination layout="total,prev,pager,next,jumper" @current-change="handleCurrentChange" :page-size="20" :total="totalpage" style="float:right;"></el-pagination>
			</el-col>
		</template>
		<!-- 新增--对应的dialog -->
		<el-dialog title="新增房间置顶" :visible.sync="dialogFormVisible">
			<el-form :model="formTwo">
				<el-form-item>
					<span style="display: inline-block;margin: 0 10px 0 60px;">置顶类型</span>
					<el-select style="width: 120px;" v-model="formTwo.choice">
						<el-option label="精选" value="1"></el-option>
						<el-option label="推荐" value="2"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="房间ID" :label-width="formLabelWidth">
					<el-input v-model="formTwo.room_id" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="置顶位置" :label-width="formLabelWidth">
					<el-select v-model="formTwo.position">
						<el-option label="1" value="1"></el-option>
						<el-option label="2" value="2"></el-option>
						<el-option label="3" value="3"></el-option>
						<el-option label="4" value="4"></el-option>
						<el-option label="5" value="5"></el-option>
						<el-option label="6" value="6"></el-option>
						<el-option label="7" value="7"></el-option>
						<el-option label="8" value="8"></el-option>
						<el-option label="9" value="9"></el-option>
						<el-option label="10" value="10"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="置顶时间段" :label-width="formLabelWidth">
					<div class="block">
						<el-date-picker v-model="formTwo.choiceDate" type="datetimerange" range-separator=" 至 " start-placeholder="开始时间" end-placeholder="结束时间"></el-date-picker>
					</div>
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
import Event from './../../public_js/event.js';
import { allget } from '../../api/api';
import store from '../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null, 
			operate_user: '',
			formOne: {
				choice: "",
				choiceDate: [new Date()-30*24*60*60*1000, new Date()], 
				type: '0', 
				room_id: '',
				room_name: '',
			},
			formTwo: {
				choice: "",
				choiceDate: [new Date(), new Date()],
				room_id: '',
				position: '1',
			},
			listLoading: false, 
			tabData: [], 
			totalpage: null, 
			page: 1, 
			star: '0',
			end: '20',
			dialogFormVisible: false,
			formLabelWidth: '130px', 
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
		// 搜索条件
		searchCondition() {
			var _this = this;
			var obj = {};
			obj.type = _this.formOne.type;
			obj.choice = _this.formOne.choice;
			obj.room_id = _this.formOne.room_id;
			obj.room_name = _this.formOne.room_name;
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.choice == "" ? delete obj.choice : obj.choice = obj.choice;
			return obj; 
		},
		// 确定了新增传递过来对应的内容值(val对应的值：0->取消，1->确认)
		addSure(val) {
			var _this = this;
			if(val==0) {
				_this.dialogFormVisible = false;
			} else if(val==1) {
				_this.listLoading = true;
				// 进行添加的操作
				let formData = new FormData();
				formData.append('room_id', _this.formTwo.room_id);
				formData.append('choice', _this.formTwo.choice);
				formData.append('position', _this.formTwo.position);
				formData.append('start_time', baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 1));
				formData.append('end_time', baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 1));
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};
				if(formData.get('room_id')!='') {
					axios.post(allget+'/NewFamily/addRoomSortPlan', formData, config)
						.then((res) => {
							_this.listLoading = false;	
							_this.dialogFormVisible = false;					
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
				} else {
					_this.listLoading = false; 
					baseConfig.warningTipMsg(_this, '新增的家族ID不能为空！');
				}
			} 
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/NewFamily/getRoomSortPlan';
			var params = _this.searchCondition();
			if(params==null) {
				_this.listLoading = false;
			} else {
				axios.get(allget+url, { params: params })		
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
			}
		},
		// 取消置顶的方法
		cancelTop(rows) {
			var _this = this;
			var id = rows.id;
			var url = '/NewFamily/cancelRoomSortPlan';
			var params = {
				id: id,
			};
			axios.get(allget+url, { params: params })					
			.then(res => {
				if(res.data.ret) {
					baseConfig.successTipMsg(_this, '取消置顶成功！');
					_this.getTableData();
				} else {
					baseConfig.warningTipMsg(_this, res.data.msg);
				}
			}).catch(function(error){
				console.log(error);
			})
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
