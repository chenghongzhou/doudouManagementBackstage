<template>
	<section>
		<el-tabs 
		v-model="activeName" 
		type="border-card"
		@tab-click="handleClick">
		    <el-tab-pane 
			label="收益数据" 
			name="first" 
			:style="{height:tabHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formOne">
						<el-form-item label="uid">
							<el-input 
							v-model="formOne.uid" 
							auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getOneData">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<el-table 
				:data="formOne.tabData" 
				style="width:100%" 
				ref="tabSearchPageHeight" 
				:height="tabSearchPageHeight">
				  	<el-table-column prop="goods_id" label="商品 id"></el-table-column>
			  		<el-table-column prop="uid" label="卖家用户 id"></el-table-column>
			  		<el-table-column prop="type" label="商品类型" min-width="200">
						  <template slot-scope="scope">
							<p v-if="scope.row.type == 1">分手</p>
							<p v-else-if="scope.row.type == 2">表白</p>
							<p v-else-if="scope.row.type == 3">挽留</p>
							<p v-else-if="scope.row.type == 4">求婚</p>
						</template>
					  </el-table-column>
			  		<el-table-column prop="voice" label="商品声音" min-width="200">
						<template slot-scope="scope">
							<audio 
							:src="scope.row.voice" 
							controls="controls" 
							preload="none"></audio>
						</template>
					</el-table-column>
					<el-table-column label="操作" width="150">
						<template slot-scope="scope">
							<el-button
							type="danger" 
							size="small" 
							@click="detale1(scope.row.goods_id)">删除</el-button>
						</template>
					</el-table-column>
				</el-table>
			    <el-col :span="24" class="toolbar">
					<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="oneHandleCurrentChange" 
						:page-size="20" 
						:total="formOne.totalPage" 
						style="float:right;">
					</el-pagination>
				</el-col>
		    </el-tab-pane>
			<el-tab-pane 
			label="通话数据" 
			name="second" 
			:style="{height:tabHeight+'px'}">
				<el-col 
				:span="24" 
				class="toolbar" 
				style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item label="商品id">
							<el-input 
							v-model="formTwo.goods_id" 
							auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item label="卖家用户id">
							<el-input 
							v-model="formTwo.seller_uid" 
							auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item label="买家用户id">
							<el-input 
							v-model="formTwo.buyer_uid" 
							auto-compvare="off"></el-input>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTwoData">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<el-table 
				:data="formTwo.tabData" 
				style="width:100%" 
				ref="tabSearchPageHeight" 
				:height="tabSearchPageHeight">
				  	<el-table-column prop="order_id" label="订单 id"></el-table-column>
			  		<el-table-column prop="goods_id" label="商品 id"></el-table-column>
			  		<el-table-column prop="buyer_uid" label="买家用户 id"></el-table-column>
			  		<el-table-column prop="remarks" label="订单备注"></el-table-column>
					<el-table-column prop="deliver_time" label="订单交付时间"></el-table-column>
			  		<el-table-column prop="order_voice" label="订单声音" min-width="200">
						<template slot-scope="scope">
							<audio 
							:src="scope.row.order_voice" 
							controls="controls" 
							preload="none"></audio>
						</template>
					</el-table-column>
			  		<el-table-column prop="seller_uid" label="卖家用户 id" min-width="200"></el-table-column>
					<el-table-column prop="type" label="商品类型" min-width="200">
						<template slot-scope="scope">
						<p v-if="scope.row.type == 1">分手</p>
						<p v-else-if="scope.row.type == 2">表白</p>
						<p v-else-if="scope.row.type == 3">挽留</p>
						<p v-else-if="scope.row.type == 4">求婚</p>
						</template>
					</el-table-column>
					<el-table-column prop="goods_voice" label="商品声音" min-width="200">
						<template slot-scope="scope">
							<audio 
							:src="scope.row.goods_voice" 
							controls="controls" 
							preload="none"></audio>
						</template>
					</el-table-column>
					<el-table-column label="操作" width="150">
						<template slot-scope="scope">
							<el-button
							type="danger" 
							size="small" 
							@click="detale2(scope.row.order_id)">删除</el-button>
						</template>
					</el-table-column>
				</el-table>
			    <el-col :span="24" class="toolbar">
					<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="twoHandleCurrentChange" 
						:page-size="20" 
						:total="formTwo.totalPage" 
						style="float:right;">
					</el-pagination>
				</el-col>
			</el-tab-pane>		
		</el-tabs>
	</section>
</template>

<script>
	import Event from './../../public_js/event.js';
	import { allget } from '../../api/api';
	import store from '../../vuex/store';
	import axios from 'axios';
	export default{
		data(){
			return{
				tabHeight: null,
				tabPageHeight: null,
				tabSearchPageHeight: null,	
				activeName:'first',
				formOne:{
					uid:'',
					totalPage:1000,
					tabData:[],
					page:1
				},
				formTwo:{
					goods_id:'',
					seller_uid:'',
					buyer_uid:'',
					totalPage:1000,
					tabData:[],
					page:1
				}
			}
		},
		mounted(){
			this.$nextTick(function() { 
				this.tabHeight = baseConfig.lineNumber(tabHeight);
				this.tabPageHeight = baseConfig.lineNumber(tabPageHeight);
				this.tabSearchPageHeight = baseConfig.lineNumber(tabSearchPageHeight);
				this.getOneData();
			})			
		},
		methods:{
			// 搜索条件
			searchConditionEarning() {
				var _this = this;
				var obj = {};
				obj.uid = _this.formOne.uid;
				obj.page=_this.formOne.page;
				return obj; 
			},
			searchConditionCall() {
				var _this = this;
				var obj = {};
				obj.goods_id = _this.formTwo.goods_id;
				obj.seller_uid = _this.formTwo.seller_uid;
				obj.buyer_uid = _this.formTwo.buyer_uid;
				obj.page=_this.formTwo.page;
				return obj; 
			},
			oneHandleCurrentChange(val){
				this.formOne.page=val;
				this.getOneData();
			},
			twoHandleCurrentChange(val){
				this.formTwo.page=val;
				this.getTwoData();
			},
	        handleClick(tab, event) {
				if(this.formTwo.tabData.length==0&&tab.name=='second') {
					this.getTwoData();
				} else {}
				if(this.formOne.tabData.length==0&&tab.name=='first') {
					this.getOneData();
				} else {}
       		},
       		getOneData(){
       			var _this = this;
				var params=_this.searchConditionEarning();
				axios.get(
					allget+'/NewActivity/getStoreGoodsList', 
					{params: params}
				)
					.then((res) => {
						if(res.data.ret==1) {
							_this.formOne.tabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			},
			detale1(id){
				var _this = this;
				_this.listLoading = true;
				this.$confirm('此操作将永久删除, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					var url = '/NewActivity/deleteStoreGoods';
					let params = {
						goods_id : id
					};
					axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if (res.data.ret) {
							_this.getOneData();
							this.$message({
								type: 'success',
								message: '删除成功!'
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
						message: '已取消删除'
					});          
				});	
			},
			getTwoData(){
       			var _this = this;
				var params=_this.searchConditionCall(); 
				axios.get(allget+'/NewActivity/getStoreOrderList', {params: params})
					.then((res) => {
						if(res.data.ret==1) {
							_this.formTwo.tabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			},
			detale2(id){
				var _this = this;
				_this.listLoading = true;
				this.$confirm('此操作将永久删除, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					var url = '/NewActivity/deleteStoreOrder';
					let params = {
						order_id : id
					};
					axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if (res.data.ret) {
							_this.getTwoData();
							this.$message({
								type: 'success',
								message: '删除成功!'
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
						message: '已取消删除'
					});          
				});	
			},
		}
	}
</script>

<style scoped="scoped">
</style>