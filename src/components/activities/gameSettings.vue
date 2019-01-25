<template>
	<!-- 游戏设置 -->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" 
		@tab-click="handleClick">
			<!-- 游戏列表 -->
			<el-tab-pane 
			label="游戏列表" 
			name="first" 
			:style="{height:tabHeight+'px'}">
				<template>
					<el-table 
					ref="tableOneHeight" 
					:data="formOne.tabData" 
					border fit highlight-current-row 
					style="width:100%;" 
					:row-class-name="tableRowClassName"
					:height="tableOneHeight">
						<el-table-column prop="id" label="游戏ID" width="50"></el-table-column>
						<el-table-column prop="name" label="游戏名称" width="100"></el-table-column>
						<el-table-column label="游戏图标" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img :src="scope.row.icon" style="width:100px;height:auto;" />
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="url" label="游戏地址" width="250"></el-table-column>
						<el-table-column label="游戏等级" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.level==0">没有等级</p>
									<p v-else-if="scope.row.level==1">一级</p>
									<p v-else-if="scope.row.level==2">二级</p>
									<p v-else-if="scope.row.level==3">三级</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="min_gold" label="游戏最少金币" width="100"></el-table-column>
						<el-table-column label="横竖屏显示" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.is_vertical==0">横屏</p>
									<p v-else-if="scope.row.is_vertical==1">竖屏</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="游戏类型" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.game_type==0">转盘</p>
									<p v-else-if="scope.row.game_type==1">喵星人</p>
									<p v-else-if="scope.row.game_type==2">炸金花</p>
									<p v-else-if="scope.row.game_type==6">牛牛</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="显示类型" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p style="color:red;" v-if="scope.row.type==0">赌博</p>
									<p v-else-if="scope.row.type==1">游戏</p>								
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="is_show" label="全局显示" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.is_show==0">否</p>
									<p v-else-if="scope.row.is_show==1">是</p>								
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="sort" label="排序" width="100"></el-table-column>
						<el-table-column label="操作" min-width="150">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								size="small" 
								@click.native.prevent="editChange(scope.row)">编辑</el-button>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="oneHandleCurrentChange" 
						:page-size="20" 
						:total="formOne.totalpage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- 房间游戏列表 -->
			<el-tab-pane 
			label="房间游戏列表" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item label="房间ID">
							<el-input 
							v-model="formTwo.room_id" 
							auto-complete="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableManage">查询</el-button>
							<el-button 
							type="primary" 
							@click="phoneCode.dialogvisible=true;">发送验证码</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableTwoHeight" 
					:data="formTwo.tabData" 
					border fit highlight-current-row 
					style="width:100%;" 
					:row-class-name="tableRowClassNameElse"
					:height="tableTwoHeight">
						<el-table-column prop="yun_xin_room_id" label="房间云信ID" width="150"></el-table-column>
						<el-table-column prop="room_id" label="房间ID" min-width="80"></el-table-column>
						<el-table-column prop="room_status" label="房间开启状态" min-width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.room_status==0">开启</p>
									<p v-else-if="scope.row.room_status==1">未开启</p>
									<p v-else-if="scope.row.room_status==2">封禁</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="is_close" label="是否关门" min-width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.is_close==0">未关</p>
									<p v-else-if="scope.row.is_close==1">关</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="room_name" label="房间名称" min-width="80"></el-table-column>
						<el-table-column prop="room_pic" label="房间图片" min-width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img :src="scope.row.room_pic" style="width:100px;height:auto;" />
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="total_honour" label="总荣耀值" min-width="80"></el-table-column>
						<el-table-column prop="create_time" label="创建时间" min-width="80"></el-table-column>
						<el-table-column prop="last_open_time" label="最后开启时间" min-width="80"></el-table-column>
						<el-table-column prop="owner_uid" label="拥有者" min-width="80"></el-table-column>
						<el-table-column prop="level" label="房间等级" min-width="80"></el-table-column>
						<el-table-column prop="gambling_name" label="游戏列表" min-width="80"></el-table-column>
						<el-table-column label="操作" min-width="150">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								size="small" 
								@click.native.prevent="editRoomGame(scope.row)">编辑</el-button>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="twoHandleCurrentChange" 
						:page-size="20" 
						:total="formTwo.totalpage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
		</el-tabs>
		<!-- 发送手机验证码 -->
		<el-dialog title="发送手机验证码" :visible.sync="phoneCode.dialogvisible">
			<el-form :model="phoneCode">
				<el-form-item label="用户手机号码" :label-width="formLabelWidth">
					<el-input 
					v-model="phoneCode.phone" 
					auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click="sendPhoneCode(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click="sendPhoneCode(1)">确 定</el-button>
			</div>
		</el-dialog>
		<!-- 编辑房间游戏 -->
		<el-dialog title="编辑游戏" :visible.sync="editGame.dialogvisible">
			<el-form :model="editGame">
				<el-form-item label="房间云信ID" :label-width="formLabelWidth">
					<el-input 
					disabled="disabled"
					v-model="editGame.yun_xin_room_id" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="游戏等级" :label-width="formLabelWidth">
					<el-select
					v-model="editGame.gambling_list"
					multiple
					collapse-tags
					placeholder="请选择游戏">
						<el-option
						v-for="item in editGame.all_gambling_list"
						:key="item.value"
						:label="item.label"
						:value="item.value"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click="editRoomGameSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click="editRoomGameSure(1)">确 定</el-button>
			</div>
		</el-dialog>
		<!-- 编辑游戏 -->
		<el-dialog title="编辑游戏" :visible.sync="edit.dialogvisible">
			<el-form :model="edit">
				<el-form-item label="游戏ID" :label-width="formLabelWidth">
					<el-input 
					disabled="disabled"
					v-model="edit.id" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="游戏名称" :label-width="formLabelWidth">
					<el-input 
					v-model="edit.name" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="游戏图片" :label-width="formLabelWidth">
					<el-input 
					v-model="edit.icon" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="游戏地址" :label-width="formLabelWidth">
					<el-input 
					v-model="edit.url" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="游戏等级" :label-width="formLabelWidth">
					<el-select v-model="edit.level">
						<el-option label="没有等级" value="0"></el-option>
						<el-option label="一级" value="1"></el-option>
						<el-option label="二级" value="2"></el-option>
						<el-option label="三级" value="3"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="游戏最少金币" :label-width="formLabelWidth">
					<el-input 
					v-model="edit.min_gold" 
					auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="横竖屏" :label-width="formLabelWidth">
					<el-select v-model="edit.is_vertical">
						<el-option label="横屏" value="0"></el-option>
						<el-option label="竖屏" value="1"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="游戏类型" :label-width="formLabelWidth">
					<el-select v-model="edit.game_type">
						<el-option label="转盘" value="0"></el-option>
						<el-option label="喵星人" value="1"></el-option>
						<el-option label="炸金花" value="2"></el-option>
						<el-option label="牛牛" value="6"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="显示类型" :label-width="formLabelWidth">
					<el-select v-model="edit.type">
						<el-option style="color:red;" label="赌博" value="0"></el-option>
						<el-option label="游戏" value="1"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="全局显示" :label-width="formLabelWidth">
					<el-select v-model="edit.is_show">
						<el-option label="否" value="0"></el-option>
						<el-option label="是" value="1"></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="排序" :label-width="formLabelWidth">
					<el-input 
					v-model="edit.sort" 
					auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click="editSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click="editSure(1)">确 定</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
import Event from './../../public_js/event.js';
import store from '../../vuex/store';
import axios from 'axios';
import { allget, khserverget } from '../../api/api.js';
export default {
	data() {
		return {
			tabHeight: null,
			tableOneHeight: null, 
			tabSearchHeight: null,
			tableTwoHeight: null, 
			formOne: {
                choiceDate: [new Date()-7*24*60*60*1000, new Date()],
                level: '1',
				room_id: '',
				tabData: [],
				totalpage: 1000, 
				page: 0, 
				star: '0',
				end: '20',
			},
			phoneCode: {
				dialogvisible: false,
				phone: '',
			},
			edit: {
				dialogvisible: false,
				id: '',
				name: '',
				icon: '',
				url: '',
				level: '',
				min_gold: '',
				is_vertical: '',
				game_type: '',
				type: '',
				is_show: '',
				sort: '',
			},
			formTwo: {
				room_id: '',
				tabData: [], 
				totalpage: 1000, 
				page: 0, 
				star: '0',
				end: '20',
			},
			editGame: {
				dialogvisible: false,
				yun_xin_room_id: '',
				all_gambling_list: [],//全部的房间游戏可供选择
				gambling_list: [],//编辑房间游戏的需要
			},
			tabActiveName: 'second',//游戏列表、房间游戏列表
			formLabelWidth: '130px', 
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
			_this.formOne.page = val-1;
			_this.formOne.star = (_this.formOne.page)*20;
			_this.formOne.end = _this.formOne.star+20;
		},
		twoHandleCurrentChange(val) {
			var _this = this;
			_this.formTwo.page = val-1;
			_this.getTableManage();
		},
		// 获取游戏列表
		getTableFind() {
			var _this = this ;
			var url = '/NewActivity/getGameList';
			var params = {};
			axios.get(allget+url, { params: params })
				.then((res) => {
					if(res.data.ret) {
						_this.formOne.totalpage = res.data.data.length; 
						_this.formOne.tabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		editChange(row) {
			var _this = this;
			_this.edit.id = row.id;
			_this.edit.name = row.name;
			_this.edit.icon = row.icon;
			_this.edit.url = row.url;
			_this.edit.level = row.level;
			_this.edit.min_gold = row.min_gold;
			_this.edit.is_vertical = row.is_vertical;
			_this.edit.game_type = row.game_type;
			_this.edit.type = row.type;
			_this.edit.is_show = row.is_show;
			_this.edit.sort = row.sort;
			_this.edit.dialogvisible = true;
		},
		editSure(val) {
			var _this = this;
			if(val==0) {
				_this.edit.dialogvisible = false;
			} else if(val==1) {
				let formData = new FormData();
				formData.append('id', _this.edit.id);
				formData.append('name', _this.edit.name);
				formData.append('icon', _this.edit.icon);
				formData.append('url', _this.edit.url);
				formData.append('level', _this.edit.level);
				formData.append('min_gold', _this.edit.min_gold);
				formData.append('is_vertical', _this.edit.is_vertical);
				formData.append('game_type', _this.edit.game_type);
				formData.append('type', _this.edit.type);
				formData.append('is_show', _this.edit.is_show);
				formData.append('sort', _this.edit.sort);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};
				axios.post(allget+'/NewActivity/editGame', formData, config)
					.then((res) => {
						_this.edit.dialogvisible = false;
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
		// 房间游戏搜索时间
		searchConditionManage() {
			var _this = this;
			var obj = {};
			obj.room_id = _this.formTwo.room_id;
			obj.page = _this.formTwo.page;
			return obj;
		},
		// 房间游戏
		getTableManage() {
			var _this = this ;
			var url = '/NewActivity/getRoomGameList';
			var params = _this.searchConditionManage();
			axios.get(allget+url, { params: params })
				.then((res) => {
					if(res.data.ret) {
						res.data.data.forEach((item, index) => {
							res.data.data[index].gambling_name = '';
							item.gambling_array.forEach((subItem, subIndex) => {
								if(subIndex==0) {
									res.data.data[index].gambling_name += subItem.name;
								} else {
									res.data.data[index].gambling_name += '、'+subItem.name;
								}
							});
						});
						_this.formTwo.tabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		editRoomGame(row) {
			var _this = this;
			_this.editGame.yun_xin_room_id = row.yun_xin_room_id;
			_this.editGame.dialogvisible = true;
		},
		editRoomGameSure(val) {
			var _this = this;
			if(val==0) {
				_this.editGame.dialogvisible = false;
			} else if(val==1) {
				console.log(_this.editGame.yun_xin_room_id);
				console.log(_this.editGame.gambling_list);
				let formData = new FormData();
				formData.append('yun_xin_room_id', _this.editGame.yun_xin_room_id);
				formData.append('gambling_list', _this.editGame.gambling_list);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};
				axios.post(allget+'/NewActivity/editRoomGame', formData, config)
					.then((res) => {
						_this.editGame.dialogvisible = false;
						if(res.data.ret) {
							baseConfig.successTipMsg(_this, '新增成功！');
							_this.getTableManage();
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
		tableRowClassName({row, rowIndex}) {
			if(rowIndex===0) {
				return 'warning-row';
			} 
			return '';
		},
		tableRowClassNameElse({row, rowIndex}) {
			if(rowIndex===0) {
				return 'warning-row';
			} 
			return '';
		},
		allRoomGame() {
			var _this = this;
			var url = '/NewActivity/getGameList';
			var params = {
				type: 1,
			};
			axios.get(allget+url, { params: params })
				.then((res) => {
					if(res.data.ret) {
						var obj = {};
						res.data.data.forEach((item, index) => {
							obj = {
								label: item.name,
								value: item.id,
							};
							_this.editGame.all_gambling_list.push(obj);
						});
						console.log(_this.editGame.all_gambling_list);
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		sendPhoneCode(val) {
			var _this = this;
			if(val==0) {
				_this.phoneCode.dialogvisible = false;
			} else if(val==1) {
				var url = '/HAccount/getVerifyCode';
				var params = {
					type: 1,
					phone: _this.phoneCode.phone,
				};
				axios.get(khserverget+url, { params: params })
					.then((res) => {
						if(res.data.code) {
							baseConfig.successTipMsg(_this, '已发送成功~');
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
			_this.tabHeight = baseConfig.lineNumber(tabHeight);
			_this.tableOneHeight = baseConfig.lineNumber(tabPageHeight);
			_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight);
			_this.tableTwoHeight = baseConfig.lineNumber(tabSearchPageHeight);
			_this.getTableFind();
			_this.getTableManage();
			_this.allRoomGame();
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
.el-table .warning-row {
	background: oldlace;
}
</style>
