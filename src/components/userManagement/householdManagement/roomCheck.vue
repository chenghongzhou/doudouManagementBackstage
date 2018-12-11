<template>
    <!-- 房间考核日志 -->
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
                    <span>房间类型</span>
                    <el-select style="width: 120px;" v-model="room_type">
                        <el-option label="全部" value=""></el-option>
                        <el-option label="家族房间" value="0"></el-option>
                        <el-option label="个人房间" value="1"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <span>UID</span>
                    <el-input 
                    style="width:120px;" 
                    clearable 
                    placeholder="请输入uid" 
                    v-model="uid"></el-input>
                </el-form-item>
                <el-form-item>
                    <span>房间ID</span>
                    <el-input 
                    style="width:120px;" 
                    clearable 
                    placeholder="房间ID" 
                    v-model="room_id"></el-input>
                </el-form-item>
                <el-form-item>
                    <span>房间名称</span>
                    <el-input 
                    style="width:120px;" 
                    clearable 
                    placeholder="房间名称" 
                    v-model="room_name"></el-input>
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
                <el-table-column prop="create_time" label="时间"></el-table-column>
                <el-table-column prop="room_id" label="房间ID"></el-table-column>
                <el-table-column prop="room_name" label="房间名称"></el-table-column>
                <el-table-column prop="room_type" :formatter="judgeRoom" label="房间类型"></el-table-column>
                <el-table-column prop="owner_uid" label="所有者UID"></el-table-column>
                <el-table-column prop="room_level" label="当前等级"></el-table-column>
                <el-table-column prop="result_status" :formatter="judgeEvent" label="事件"></el-table-column>
                <el-table-column prop="days" label="考核周期"></el-table-column>
                <el-table-column prop="chat_ticket" label="周期内礼物豆票"></el-table-column>
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
            page: 0,
            totalpage: 1000,
            uid: "",
            room_id: "",
            room_type: "",
            room_name: "",
        };
    },
    methods: {
        handleCurrentChange(val) {
            this.page = val - 1;
            this.getData();
        },
        judgeRoom(row) {
            if(row.room_type == 0){
                return "家族房间";
            }else if(row.room_type == 1){
                return "个人房间";
            }
        },
        judgeEvent(row) {
            if(row.result_status == 0){
                return "申请考核中";
            }else if(row.result_status == 1){
                return "考核成功,升级";
            }else if(row.result_status == 2){
                return "考核失败";
            }else if(row.result_status == 3){
                return "未达标，降级";
            }
        },
        // 获取数据
        getData(type) {
            var _this = this;
            _this.listLoading = true;
            if (type == 0) {
                _this.page = 0;
            }
            let url = "/NewFamily/getRoomCheckRecord";
            let params = {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                room_type: this.room_type,
                owner_uid: this.uid,
                family_id: this.room_id,
                page: this.page,
                room_name: this.room_name,
            };
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
