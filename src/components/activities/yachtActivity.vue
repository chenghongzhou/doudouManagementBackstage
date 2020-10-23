<template>
	<!-- 六一 -->
	<section>
		<template>
            
            <div style="overflow:hidden">
                <div class="infos">开始时间：{{formOne.start_time}}</div>
                <div class="infos">结束时间：{{formOne.end_time}}</div>
                <div class="infos">礼物id：{{formOne.gift_id}}</div>
                 <el-button 
                type="primary" 
                @click="setNewloading.dialogFormVisible = true" style="margin:10px 200px">设置</el-button>
            </div>
            
            <div class="infos" style="margin-top:0;">活动规则：{{formOne.rule}}</div>
			<el-table 
            ref="tableHeight" 
            :data="tablList" 
            border fit highlight-current-row 
            v-loading="listLoading" 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="uid" label="	用户 id"></el-table-column>
				<el-table-column prop="nickname" label="用户昵称"></el-table-column>
                <el-table-column prop="total" label="礼物数量"></el-table-column>
                <el-table-column prop="img" label="图片">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
							<img 
							:src="scope.row.gift_icon"
							style="width:100px;height:100px;">
						</div>
                    </template>
                </el-table-column>

			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="oneHandleCurrentChange" 
                :page-size="20" 
                :total="formOne.totalPage" 
                style="float:right;"></el-pagination>
			</el-col>
		</template>
        <el-dialog title="设置" :visible.sync="setNewloading.dialogFormVisible">
			<el-form :model="setNewloading.params"> 
				<el-form-item label="礼物id">
					<el-input 
					style="width:250px;margin-bottom:20px"
					v-model="setNewloading.params.gift_id" 
					auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item label="开始时间">
                        <el-date-picker 
                        v-model="setNewloading.params.start_time" 
                        type="datetime" 
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="选择日期范围" style="width:30%"></el-date-picker>
                </el-form-item>
                <el-form-item label="结束时间">
                        <el-date-picker 
                        v-model="setNewloading.params.end_time" 
                        type="datetime" 
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="选择日期范围" style="width:30%"></el-date-picker>
                </el-form-item>
                <el-form-item label="活动规则" style="margin-top:20px">
					<el-input 
					type="textarea"
					style="width:350px"
					v-model="setNewloading.params.rule" 
					auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click.native.prevent="setBannerSure(0)">取 消</el-button>
				<el-button 
				type="primary" 
				@click.native.prevent="setBannerSure(1)">确 定</el-button>
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
			formOne: {
                choiceDate: [new Date()-30*24*60*60*1000, new Date()],
                uid : '',
                tabData: [],
                totalPage: 1000,
                star: '0',
                end: '20',
                page:1,
                start_time:'',
                end_time:'',
                rule:'',
                gift_id:''
            },
			listLoading: false,
             setNewloading: {
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
    computed: {
        tablList(){
            var _this = this;
            return _this.formOne.tabData.slice(_this.formOne.star,_this.formOne.end);
        }
    },
	methods: {
        oneHandleCurrentChange(val) {
			var _this = this;
            val = val-1;
            _this.formOne.page = val;
            _this.formOne.star = (_this.formOne.page)*20;
            _this.formOne.end = (_this.formOne.star)+20;
        },
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
            var url = '/NewActivity/getYachtData';
			var params = {
               page:_this.formOne.page,
            };
            axios.get(allget+url, { params: params })
            .then((res) => {
                _this.listLoading = false;
                if(res.data.ret) {
                    _this.formOne.start_time = res.data.data.start_time;
                    _this.formOne.end_time = res.data.data.end_time;
                    _this.formOne.rule = res.data.data.rule;
                    _this.formOne.gift_id = res.data.data.gift_id;

                    _this.formOne.tabData = res.data.data.ranking_list;
                }
                else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.log(err);
            });
        },
        setBannerSure(num){
			var _this = this;
				if(num == 0){
					_this.setNewloading.dialogFormVisible = false;
				}else{
                    if(_this.setNewloading.params.gift_id == ''){
                        baseConfig.successTipMsg(_this, '请填写礼物id');
                        return false;
                    };
                    if(_this.setNewloading.params.rule == ''){
                        baseConfig.successTipMsg(_this, '请填写活动规则');
                        return false;
                    };
                    if(_this.setNewloading.params.start_time == '' || _this.setNewloading.params.end_time == ''){
                        baseConfig.successTipMsg(_this, '请设置时间');
                        return false;
                    };
					let params = {
						start_time: _this.setNewloading.params.start_time,
						end_time: _this.setNewloading.params.end_time,
						gift_id: _this.setNewloading.params.gift_id,
						rule: _this.setNewloading.params.rule,
					};
					axios.get(allget+'/NewActivity/setYachtData', {params: params})
					.then((res) => {
						if(res.data.ret == 1) {
                            _this.getTableData();
							baseConfig.successTipMsg(_this, '设置成功');
							_this.setNewloading.dialogFormVisible = false;
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
			_this.tableHeight = searchPageHeight-40;
			_this.getTableData();
		});
	}
};
</script>

<style lang="css" scoped>
.infos{
    float: left;
    font-size: 14px;
    color: #333333;
    margin:20px;
}
</style>
