<template>
	<!-- 主播个人数据汇总 -->
	<section>
		<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
			<el-form :inline="true" style="overflow:hidden;">
				<el-form-item>
					<div class="block">
						<span class="registerTime">日期</span>
						<el-date-picker 
						v-model="formOne.startDate" 
						type="daterange" 
						range-separator=" 至 " 
						start-placeholder="开始日期" 
						end-placeholder="结束日期"></el-date-picker>
					</div>
				</el-form-item>
				<el-form-item>
					<span>uid</span>
					<el-input 
					style="width:200px;" 
					placeholder="请输入uid" 
					v-model="uid" 
					clearable></el-input>
				</el-form-item>
				<el-form-item>
					<span>操作人</span>
					<el-input 
					style="width:200px;" 
					placeholder="请输入操作人" 
					v-model="operate_user" 
					clearable></el-input>
				</el-form-item>
				<el-form-item style="margin-left: 50px;">
					<el-button 
					id="addAuthor" 
					type="primary" 
					@click="dialogFormVisible=true">新增主播</el-button>
				</el-form-item>
				<el-form-item style="float:right;">
					<el-button 
					type="primary" 
					@click="getData()">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
			:data="listData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			element-loading-text="拼命加载中" 
			element-loading-spinner="el-icon-loading" 
			element-loading-background="rgba(0, 0, 0, 0.8)"  
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="uid" label="用户UID"></el-table-column>
				<el-table-column prop="register_time" label="注册时间"></el-table-column>
				<el-table-column prop="addtime" label="添加时间"></el-table-column>
				<el-table-column prop="nickname" label="昵称"></el-table-column>
				<el-table-column prop="lasttime" label="最后登录时间"></el-table-column>
				<el-table-column prop="accumulate_time" label="累积通话时长" sortable></el-table-column>
				<el-table-column prop="total_chat_ticket" label="累积豆票数"></el-table-column>
				<el-table-column prop="total_money" label="累积收入"></el-table-column>
				<el-table-column prop="follow_num" label="关注数"></el-table-column>
				<el-table-column prop="fans_num" label="粉丝数"></el-table-column>
				<el-table-column prop="rand_call_num" label="累积随机通话次数"></el-table-column>
				<el-table-column prop="operate_user" label="操作人"></el-table-column>
			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
				layout="total,prev,pager,next,jumper" 
				@current-change="handleCurrentChange" 
				:page-size="20" 
				:total=1000 
				style="float:right;"></el-pagination>
			</el-col>
		</template>
		<el-dialog title="新增主播" :visible.sync="dialogFormVisible" center>
			<el-form>
				<el-form-item label="请输入uid" :label-width="formLabelWidth">
					<el-input 
					v-model="addUid" 
					placeholder="请输入uid" 
					auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button 
				@click="dialogFormVisible=false">取 消</el-button>
				<el-button 
				type="primary" 
				@click="dialogFormVisible=false,addNewAuthor()">确 定</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
import { allget } from "../../../api/api";
import store from '../../../vuex/store.js'
import axios from 'axios';
export default {
    data() {
        return {
            tableHeight: null, 
            formOne: {
                startDate: [new Date()-7*24*60*60*1000, new Date()] 
            },
            listData: [],
            formLabelWidth: "120px",
            listLoading: false,
            uid: null,
            operate_user: null,
            dialogFormVisible: false,
			addUid: null,
			listLoading: false,
			page: 0,
        };
    },
    methods: {
		handleCurrentChange(val) {
            this.page = val - 1;
            this.getData();
        },
        // 获取数据
        getData() {
			var _this = this;
			_this.listLoading = true;
            let url = "/Anchor/getAnchorOwnData";
            let params = {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                uid: this.uid,
                operate_user: this.operate_user,
				page: this.page,
            };
			axios.get(allget+url, {params: params})
                .then((res) => {
					_this.listLoading = false;
					if(res.data.ret){
						this.listData = res.data.data;
					}else{
						baseConfig.warningTipMsg(res.data.msg);
					}
                })
                .catch((err) => {
                    console.log(err);
                });
        },
		addNewAuthor() {
			var _this = this;
			let url = '/Anchor/registerAnchorByUid';
			let params = {
				uid: this.addUid,
				operate_user: store.state.user.name,
			}
			axios.get(allget+url, {params: params})
				.then((res) => {
					if(res.data.ret){
						baseConfig.successTipMsg(_this,res.data.msg)
					}else{
						baseConfig.warningTipMsg(_this,res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
					baseConfig.warningTipMsg(_this, 'error');
				});
		}
    },
    mounted() {
        var _this = this;
        _this.tableHeight = baseConfig.lineNumber(searchPageHeight);
        _this.getData();
    }
};
</script>

<style lang="css" scoped>

</style>
