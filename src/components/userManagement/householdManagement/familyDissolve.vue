<template>
    <!-- 解散家族 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;">
            <el-form :inline="true" style="overflow:hidden;">
                <el-form-item>
                    <el-button 
                    class="dissolve" 
                    type="primary" 
                    @click="dialogFormVisible=true">解散家族</el-button>
                </el-form-item>
            </el-form>
        </el-col>
        <template>
            <el-table 
            :data="listData" 
            border fit highlight-current-row 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="dismiss_time" label="解散时间"></el-table-column>
                <el-table-column prop="family_id" label="家族ID"></el-table-column>
                <el-table-column prop="family_name" label="家族名称"></el-table-column>
                <el-table-column prop="owner_uid" label="创建者UID"></el-table-column>
                <el-table-column prop="level" label="家族等级"></el-table-column>
                <el-table-column prop="reason" label="解散理由"></el-table-column>
                <el-table-column prop="operate_user" label="操作人"></el-table-column>
            </el-table>
            <el-dialog title="解散家族" :visible.sync="dialogFormVisible">
                <el-form :model="form">
                    <el-form-item label="家族ID" :label-width="formLabelWidth">
                        <el-input 
                        v-model="form.family_id" 
                        auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="解散理由" :label-width="formLabelWidth">
                        <el-input 
                        v-model="form.reason" 
                        auto-complete="off"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button 
                    @click="dialogFormVisible=false">取 消</el-button>
                    <el-button 
                    type="primary" 
                    @click="dissolve()">确 定</el-button>
                </div>
            </el-dialog>
            <el-col :span="24" class="toolbar">
                <el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="handleCurrentChange" 
                :page-size="20" 
                :total=1000 
                style="float:right;"></el-pagination>
            </el-col>
        </template>
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
            page: 0,
            listData: [],
            totalpage: null,
            formLabelWidth: "120px",
            listLoading: false,
            operate_user: null,
            form: {
                family_id: null,
                reason: null
            },
            dialogFormVisible: false,
        };
    },
    methods: {
        handleCurrentChange(val) {
            this.page = val - 1;
            this.getData();
        },
        // 获取数据
        getData(type) {
            var _this = this;
            if (type == 0) {
                _this.page = 0;
            }
            let url = "/NewFamily/getDismissFamilyRecord";
            let params = {
                page: this.page,
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    this.listData = res.data.data;
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        // 解散家族
        dissolve() {
            var _this = this;
            let url = "/NewFamily/dismissFamily";
            let params = {
                family_id: this.form.family_id,
                reason: this.form.reason,
                operate_user: this.operate_user
            };
            axios.get(allget+url, {params: params})
                .then((res) => {
                    if(res.data.ret){
                        _this.dialogFormVisible=false;
                        baseConfig.successTipMsg(_this, res.data.msg);
                    }else{
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        }
    },
    mounted() {
        var _this = this;
        _this.tableHeight = baseConfig.lineNumber(searchPageHeight);
        _this.getData();
        this.operate_user = store.state.user.name;
    }
};
</script>

<style lang="css" scoped>
.dissolve {
    margin-left:20px;
}
</style>
