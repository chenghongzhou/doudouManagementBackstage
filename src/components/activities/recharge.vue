<template>
	<!-- 六一 -->
	<section>
		<template>
			<el-form :inline="true" style="overflow:hidden;">
                <el-form-item>
                    <el-button 
                    type="primary" 
                    @click="clearData" style="margin:50px 200px">删除数据</el-button>
                </el-form-item>
            </el-form>
		</template>
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
			listLoading: false,
		};
    },
	methods: {
		// 获取数据列表
		clearData() {
			var _this = this ;
            _this.listLoading = true;
				this.$confirm('此操作将永久删除, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					var url = '/NewActivity/deleteRechargeBagData';
					let params = {
						
					};
					axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if (res.data.ret) {
							this.$message({
								type: 'success',
								message: '操作成功!'
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

	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = searchPageHeight+60;
		});
	}
};
</script>

<style lang="css" scoped>
</style>
