<template>
	<!-- 首页标签管理 -->
	<section>
		<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
			<el-form :inline="true" style="overflow:hidden;">
                <el-form-item>
					<el-button 
                    type="primary" 
                    @click="dialogFormVisible=true;">新增标签</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
            ref="tableHeight" 
            :data="tabData" 
            border fit highlight-current-row 
            v-loading="listLoading" 
            style="width:100%;" 
            :height="tableHeight">
				<el-table-column prop="tab_name" label="标签名称"  sortable ></el-table-column>
				<el-table-column prop="create_time" label="添加日期" sortable ></el-table-column>
				<el-table-column prop="sort" label="当前排序" sortable ></el-table-column>
				<el-table-column label="操作">
					<template slot-scope="scope">
						<el-button 
                        type="danger" 
                        @click.native.prevent="deleteOneUserData(scope.$index, tabData)" 
                        size="small">删除</el-button>
						<el-button 
                        type="primary" 
                        @click.native.prevent="editOneUserData(scope.$index, tabData)" 
                        size="small">编辑</el-button>
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
            <el-dialog title="新增标签" :visible.sync="dialogFormVisible">
                <el-form :model="form">
                    <el-form-item label="标签名称：" :label-width="formLabelWidth">
                        <el-input 
                        v-model="form.tab_name" 
                        auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="标签sort：" placeholder="标签名称" :label-width="formLabelWidth">
                        <el-input 
                        v-model="form.sort" 
                        placeholder="sort"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button 
                    @click="dialogFormVisible=false">取 消</el-button>
                    <el-button 
                    type="primary" 
                    @click="addHomeTab()">确 定</el-button>
                </div>
            </el-dialog>
            <el-dialog title="编辑标签" :visible.sync="formTwo.dialogFormVisible">
                <el-form :model="formTwo">
                    <el-form-item label="标签名称：" :label-width="formLabelWidth">
                        <el-input 
                        v-model="formTwo.tab_name" 
                        auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="标签sort：" placeholder="标签名称" :label-width="formLabelWidth">
                        <el-input 
                        v-model="formTwo.sort" 
                        placeholder="sort"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button 
                    @click="formTwo.dialogFormVisible=false">取 消</el-button>
                    <el-button 
                    type="primary" 
                    @click="sureEditUser()">确 定</el-button>
                </div>
            </el-dialog>
		</template>
	</section>
</template>

<script>
import Event from './../../../public_js/event.js';
import { allget } from '../../../api/api';
import store from '../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null,
			form: {
				tab_name: '',
				sort: '',
            },
            formTwo: {
                dialogFormVisible: false,
                tab_name: "",
                sort: "",
                position: "",
            },
			listLoading: false, 
			tabData: [],
			totalpage: null,
			page: 1,
			star: '0',
			end: '20',
			dialogFormVisible: false,
			formLabelWidth: '120px',
		};
	},
	methods: {
		handleCurrentChange(val) {
			var _this = this;
			_this.page = val-1;
			_this.star = (_this.page)*20;
			_this.end = _this.star+20;
		},
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
            var url = '/User/get_home_tab_name';
            var params = {};
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if(res.data.ret){
                        _this.tabData = res.data.data;
                        _this.totalpage = res.data.data.length;
                    }else{
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
		},
        // 新增标签页
        addHomeTab() {
            var _this = this;
            var url = '/User/add_home_name';
            var params = {
                tab_name: _this.form.tab_name,
                sort: _this.form.sort,
            }
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if(res.data.ret) {
                        baseConfig.normalTipMsg(_this, res.data.msg);
                        _this.dialogFormVisible=false;
                        _this.getTableData();
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        // 编辑标签页
        editOneUserData(index, row){
            this.formTwo.tab_name = row[index].tab_name;
            this.formTwo.sort = row[index].sort;
            this.formTwo.position = row[index].position;
            this.formTwo.dialogFormVisible=true;
        },
        sureEditUser(){
            var _this = this;
            var url = "/User/edit_home_tab_name";
            var params = {
                tab_name: this.formTwo.tab_name,
                sort: this.formTwo.sort,
                position: this.formTwo.position,
            }
            axios.get(allget+url, {params: params})
                .then((res)=>{
                    if(res.data.ret) {
                        _this.formTwo.dialogFormVisible=false;
                        baseConfig.normalTipMsg(_this, res.data.msg);
                        _this.getTableData();
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        // 删除
        deleteOneUserData() {
            baseConfig.warningTipMsg(this, "请联系服务端进行删除！谢谢。。。")
        }
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
</style>
