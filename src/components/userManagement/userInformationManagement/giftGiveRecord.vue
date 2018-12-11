<template>
    <!-- 礼物赠送查询 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;">
            <el-form :inline="true" style="overflow:hidden;">
                <el-form-item>
                    <div class="block">
                        <span class="registerTime">日期</span>
                        <el-date-picker 
                        v-model="formOne.startDate" 
                        style="width:250px;" 
                        type="daterange" 
                        range-separator=" 至 " start-placeholder="开始日期" 
                        end-placeholder="结束日期" 
                        :default-time="['00:00:00', '23:59:59']">
                        </el-date-picker>
                    </div>
                </el-form-item>
                <el-form-item>
                    <span>赠送类型：</span>
                    <el-select v-model="type">
                        <el-option label="全部" value=""></el-option>
                        <el-option label="赠送他人" value="1"></el-option>
                        <el-option label="被赠送" value="2"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <span>UID：</span>
                    <el-input 
                    style="width:150px;" 
                    clearable 
                    placeholder="请输入uid" 
                    v-model="uid"></el-input>
                </el-form-item>
                <el-form-item>
                    <span>礼品名称：</span>
                    <el-input 
                    style="width:150px;" 
                    clearable 
                    placeholder="请输入物品名称" 
                    v-model="gift_name"></el-input>
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
            element-loading-text="拼命加载中" 
            element-loading-spinner="el-icon-loading" 
            element-loading-background="rgba(0, 0, 0, 0.8)" 
            border fit highlight-current-row 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="time" label="时间"></el-table-column>
                <el-table-column prop="uid" label="UID"></el-table-column>
                <el-table-column prop="nickname" label="昵称"></el-table-column>
                <el-table-column prop="name" label="礼物名称"></el-table-column>
                <el-table-column prop="price" label="价格"></el-table-column>
                <el-table-column prop="give_uid" label="对方UID"></el-table-column>
                <el-table-column prop="give_nickname" label="对方昵称"></el-table-column>
                <el-table-column prop="gift_num" label="数量"></el-table-column>
                <el-table-column prop="chat_gold" label="总价值"></el-table-column>
            </el-table>
            <el-col :span="24" class="toolbar">
                <el-pagination 
                layout="total,prev,pager,next,jumper" 
                :page-size="20" 
                @current-change="handleCurrentChange" 
                :total="totalpage" 
                style="float:right;"></el-pagination>
            </el-col>
        </template>
    </section>
</template>

<script>
import { allget } from "../../../api/api";
import store from '../../../vuex/store';
import axios from 'axios';
export default {
    data() {
        return {
            tableHeight: null, 
            formOne: {
                startDate: [new Date()-7*24*60*60*1000, new Date()] 
            },
            listData: [],
            type: "",
            gift_name: "",
            uid: "",
            formLabelWidth: "120px",
            listLoading: false,
            page: 0,
            totalpage: 1000,
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
            _this.listLoading = true;
            if (type == 0) {
                _this.page = 0;
            }
            let url = "/NewGift/getGiftGiveRecord";
            let params = {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                uid: this.uid,
                gift_name: this.gift_name,
                type: this.type,
                page: this.page,
            };
    		if( params.uid!== null && params.uid !== ''){
			//  判断搜索有没有uid,如果有就将开始时间和结束时间设置为null
            	params.date_s=null;
            	params.date_e=null;
            }
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                        _this.listData = res.data.data;
                    } else {
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
