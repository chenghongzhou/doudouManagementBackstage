<template>
	<!-- 录音投诉管理 -->
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
                    <span>UID:</span>
                    <el-input 
                    style="width:120px;" 
                    v-model="uid" 
                    placeholder="请输入UID" 
                    clearable></el-input>
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
            :data="onePageTabData" 
            v-loading="listLoading" 
            border fit highlight-current-row 
            style="width:100%;" 
            :height="tableHeight">
				<el-table-column prop="date" label="日期"></el-table-column>
				<el-table-column prop="num" label="被投诉次数"></el-table-column>
				<el-table-column prop="complain_uid" label="投诉人"></el-table-column>
				<el-table-column prop="reason" label="理由"></el-table-column>
				<el-table-column prop="voice_id" label="录音编码"></el-table-column>
				<el-table-column prop="uid" label="录音发布者"></el-table-column>
				<el-table-column prop="voice_url" width="300" label="录音内容">
                    <template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<audio 
                            controls="controls" 
                            :src="scope.row.voice_url"></audio>
						</div>
					</template>
                </el-table-column>
				<el-table-column prop="desc" label="描述"></el-table-column>
				<el-table-column label="操作">
					<template slot-scope="scope">
                        <el-button 
                        size="mini" 
                        type="success" 
                        @click="ignore(scope.$index, scope.row)">忽视</el-button>
                        <el-button 
                        size="mini" 
                        type="warning" 
                        @click="deleteFun(scope.$index, scope.row)">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
            <el-col :span="24" class="toolbar">
				<el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="handleCurrentChange" 
                :page-size="20" 
                :total="totalpage" 
                style="float:right;"></el-pagination>
			</el-col>
		</template>
		<el-dialog title="提示" :visible.sync="ignoreDialog" width="30%">
			<span>确定要忽视吗？</span>
			<span slot="footer" class="dialog-footer">
				<el-button 
                @click="ignoreDialog=false">取 消</el-button>
				<el-button 
                type="primary" 
                @click="ignoreDialog=false,sureIgnore()">确 定</el-button>
			</span>
		</el-dialog>
        <el-dialog title="提示" :visible.sync="deleteDialog" width="30%">
			<span>确定要删除吗？</span>
			<span slot="footer" class="dialog-footer">
				<el-button 
                @click="deleteDialog=false">取 消</el-button>
				<el-button 
                type="primary" 
                @click="deleteDialog=false,sureDeleteFun()">确 定</el-button>
			</span>
		</el-dialog>
	</section>
</template>

<script>
import { allget } from "../../../api/api";
import axios from "axios";
import store from "../../../vuex/store";
export default {
    data() {
        return {
            tableHeight: null,
            formOne: {
                startDate: [new Date()-7*24*60*60*1000, new Date()]
            },
            recordStatus: "",
            listData: [],
            formLabelWidth: "120px",
            listLoading: false,
            formLabelWidth: "120px",
            uid: "",
            operate_user: null,
            ignoreDialog: false,
            ignoreId: null,
            deleteDialog: false,
            deleteId: null,
            page: 0,
			totalpage: null,
			star: '0',
			end: '20',
        };
    },
	computed: {
		onePageTabData() {
			var _this = this;
			return _this.listData.slice(_this.star, _this.end);
		},
	},
    methods: {
		handleCurrentChange(val) {
			var _this = this;
			_this.page = val;
			_this.star = (_this.page-1)*20;
			_this.end = _this.star+20;
        },
        // 获取数据
        getData() {
            var _this = this;
            _this.listLoading = true;
            let url = "/NewVoice/getVoiceComplain";
            let params = {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                uid: _this.uid,
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if(res.data.ret){
                        this.listData = res.data.data;
                        _this.totalpage = res.data.data.length;
                    }else{
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    baseConfig.warningTipMsg(_this, err.data.msg);
                });
        },
        // 忽视
        ignore(index, row) {
            this.ignoreId = row.voice_id;
            this.ignoreDialog = true;
            
        },
        sureIgnore() {
            var _this = this;
            let url = "/Voice/ignoreVoiceComplain";
            let params = {
                id: this.ignoreId
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        baseConfig.successTipMsg(_this, res.data.msg);
                        _this.getData();
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    baseConfig.warningTipMsg(_this, err.data.msg);
                });
        },
        // 删除
        deleteFun(index, row) {
            this.deleteId = row.voice_id;
            this.deleteDialog = true;
        },
        sureDeleteFun() {
            var _this = this;
            let url = "/Voice/deleteVoiceComplain";
            let params = {
                id: this.deleteId
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if (res.data.ret) {
                        baseConfig.successTipMsg(_this, res.data.msg);
                        _this.getData();
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
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
        _this.operate_user = store.state.user.name;
    },
};
</script>

<style lang="css" scoped>

</style>
