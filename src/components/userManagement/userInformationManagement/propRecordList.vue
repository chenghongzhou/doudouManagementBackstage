<template>
    <!-- 道具流水查询 -->
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
                        placeholder="选择日期范围"></el-date-picker>
                    </div>
                </el-form-item>
                <el-form-item>
					<span>类型</span>
					<el-select v-model="type">
						<el-option label="全部" value=""></el-option>
						<el-option label="获得" value="56"></el-option>
						<el-option label="使用" value="55"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item>
                    <span>UID</span>
                    <el-input 
                    style="width:200px;" 
                    placeholder="请输入uid" 
                    v-model="uid" 
                    clearable></el-input>
                </el-form-item>
                <el-form-item>
                    <span>道具名称</span>
                    <el-input 
                    style="width:200px;" 
                    v-model="name" 
                    clearable></el-input>
                </el-form-item>
                <el-form-item style="float:right;">
                    <el-button 
                    type="primary" 
                    @click="getData(0)">查询</el-button>
                </el-form-item>
            </el-form>
        </el-col>
        <template>
            <el-table 
            :data="listData" 
            v-loading="listLoading" 
            border fit highlight-current-row 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="uid" label="UID"></el-table-column>
                <el-table-column prop="nickname" label="昵称"></el-table-column>
                <el-table-column prop="name" label="道具名称"></el-table-column>
                <el-table-column prop="action_type" label="类型">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
							<p v-if="scope.row.action_type==55">使用道具</p>
							<p v-else-if="scope.row.action_type==56">获得道具</p>
						</div>
                    </template>
                </el-table-column>
                <el-table-column prop="create_time" label="时间"  sortable></el-table-column>
            </el-table>
            <el-col :span="24" class="toolbar">
                <el-pagination 
                layout="total,prev,pager,next,jumper" 
                :page-size="20"  
                @current-change="handleCurrentChange" 
                :total="totalpage" 
                style="float:right;">
                </el-pagination>
            </el-col>
        </template>
    </section>
</template>

<script>
import { allget } from "../../../api/api";
import axios from 'axios';
export default {
    data() {
        return {
            tableHeight: null, 
            formOne: {
                startDate: [new Date()-14*24*60*60*1000, new Date()]
            },
            listData: [],
            formLabelWidth: "120px",
            listLoading: false,
            uid: null,
            name: null,
            type: null,
            page: 0,
            totalpage: 1000,
        };
    },
    methods: {
        handleCurrentChange(val) {
            this.page = val-1;
            this.getData();
        },
        // 获取数据
        getData(type) {
            var _this = this;
            _this.listLoading = true;
            if(type==0){
                _this.page = 0;
            }
            let url = "/NewUser/getPropRecord";
            let params = {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                page: this.page,
                uid: this.uid,
                name: this.name,
                type: this.type,
            };
            if(params.uid == null){
                delete params.uid;
            }
            if(params.name == null){
                delete params.name;
            }
            if(params.type == null){
                delete params.type;
            }
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if(res.data.ret){
                        _this.listData = res.data.data;
                    }else{
                        baseConfig.successTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
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
