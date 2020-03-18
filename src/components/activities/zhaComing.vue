<template>
	<!-- 六一 -->
	<section>
		<template>
			<el-form :inline="true" style="overflow:hidden;">
                <el-form-item>
                    <el-button 
                    type="primary" 
                    @click="addNewloading.dialogFormVisible = true" style="margin:50px 200px">设置</el-button>
                </el-form-item>
            </el-form>
		</template>
        <el-dialog title="设置" :visible.sync="addNewloading.dialogFormVisible">
			<el-form :model="addNewloading.params"> 
				<el-form-item label="礼物id">
					<el-input 
					style="width:250px;margin-bottom:20px"
					v-model="addNewloading.params.gift_id" 
					auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item label="开始时间">
                        <el-date-picker 
                        v-model="addNewloading.params.start_time" 
                        type="datetime" 
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="选择日期范围" style="width:30%"></el-date-picker>
                </el-form-item>
                <el-form-item label="结束时间">
                        <el-date-picker 
                        v-model="addNewloading.params.end_time" 
                        type="datetime" 
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="选择日期范围" style="width:30%"></el-date-picker>
                </el-form-item>
                <el-form-item label="物品说明" style="margin-top:20px">
					<el-input 
					type="textarea"
					style="width:350px"
					v-model="addNewloading.params.rule" 
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
            formLabelWidth: "120px",
            addNewloading: {
                dialogFormVisible: false,
                params: {
                    gift_id:'',
                    rule:'',
                    start_time:'',
                    end_time:''
                }
            }
		};
    },
	methods: {
		// 获取数据列表
        addBannerSure(num){
			var _this = this;
				if(num == 0){
					_this.addNewloading.dialogFormVisible = false;
				}else{
                    if(_this.addNewloading.params.gift_id == ''){
                        baseConfig.successTipMsg(_this, '请填写礼物id');
                        return false;
                    };
                    if(_this.addNewloading.params.rule == ''){
                        baseConfig.successTipMsg(_this, '请填写活动规则');
                        return false;
                    };
                    if(_this.addNewloading.params.start_time == '' || _this.addNewloading.params.end_time == ''){
                        baseConfig.successTipMsg(_this, '请设置时间');
                        return false;
                    };
					let params = {
						start_time: _this.addNewloading.params.start_time,
						end_time: _this.addNewloading.params.end_time,
						gift_id: _this.addNewloading.params.gift_id,
						rule: _this.addNewloading.params.rule,
					};
					axios.get(allget+'/NewActivity/setZhaData', {params: params})
					.then((res) => {
						if(res.data.ret == 1) {
							baseConfig.successTipMsg(_this, '设置成功');
							_this.addNewloading.dialogFormVisible = false;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					})
				}
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
