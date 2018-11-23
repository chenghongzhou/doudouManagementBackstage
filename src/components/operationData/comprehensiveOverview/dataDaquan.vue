<template>
    <!-- 数据大全 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" style="overflow: hidden;">
                <el-form-item>
                    <div class="block">
                        <span class="registerTime">日期</span>
                        <el-date-picker 
                        v-model="formOne.params.startDate" 
                        type="daterange" 
                        range-separator=" 至 " 
                        start-placeholder="开始日期" 
                        end-placeholder="结束日期"></el-date-picker>
                    </div>
                </el-form-item>
                <el-form-item style="margin-left: 100px;">
                    <span>渠道</span>
                    <el-select 
                    v-model="formOne.params.channelId" 
                    multiple 
                    collapse-tags 
                    style="margin-left:20px;" 
                    placeholder="请选择">
                        <el-option 
                        v-for="(item, key) of formOne.params.channelData" 
                        :key="key" 
                        :label="item" 
                        :value="key"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item class="search-span" style="float:right;">
                    <el-button 
                    id="searchBtn" 
                    type="primary" 
                    @click="getData()">查询</el-button>
                </el-form-item>
            </el-form>
        </el-col>
        <template>
            <el-table 
            :data="onePageTabData" 
            border fit highlight-current-row 
            style="width:100%;" 
            :height="searchPageHeight">
                <el-table-column prop="time" label="时间"></el-table-column>
                <el-table-column prop="device" label="激活"></el-table-column>
                <el-table-column prop="active" label="DAU"></el-table-column>
                <el-table-column prop="ACCU" label="ACCU"></el-table-column>
                <el-table-column prop="consume_user" label="付费人数"></el-table-column>
                <el-table-column prop="total_fee" label="付费金额"></el-table-column>
                <el-table-column prop="avg_online_time" label="平均在线时长"></el-table-column>
                <el-table-column prop="consume_rate" label="付费率"></el-table-column>
                <el-table-column prop="consume_arpu" label="付费ARPU值"></el-table-column>
                <el-table-column prop="active_arpu" label="活跃ARPU值"></el-table-column>
                <el-table-column prop="first_time_user" label="新增付费人数"></el-table-column>
                <el-table-column prop="first_time_amount" label="新增充值金额"></el-table-column>
                <el-table-column prop="new_consume_rate" label="新增付费率"></el-table-column>
                <el-table-column prop="new_consume_arpu" label="新增付费ARPU"></el-table-column>
                <el-table-column prop="new_active_arpu" label="新增活跃ARPU值"></el-table-column>
                <el-table-column prop="online_total_best" label="最高在线人数"></el-table-column>
                <el-table-column prop="online_avg_man" label="平均在线人数"></el-table-column>
                <el-table-column prop="ret_one_rate" label="次日留存"></el-table-column>
                <el-table-column prop="ret_seven_rate" label="七日留存"></el-table-column>
            </el-table>
            <el-col :span="24" class="toolbar">
				<el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="handleCurrentChange" 
                :page-size="20" 
                :total="formOne.totalpage" 
                style="float:right;"></el-pagination>
			</el-col>
        </template>
    </section>
</template>

<script>
import { allget } from "../../../api/api";
import store from "../../../vuex/store";
import axios from 'axios';
export default {
    data() {
        return {
            searchPageHeight: null,
            formOne: {
                params: {
                    startDate: [new Date()-20*24*60*60*1000, new Date()],
                    channelData: {},
                    channelId: null,
                },
                tabData: [],
                totalpage: null,
                page: 1,
                star: '0',
                end: '20',
            },
        };
    },
    computed:{
        /*
            1、onePageTabData：计算的属性
        */ 
		onePageTabData() {
			var _this = this;
			return _this.formOne.tabData.slice(_this.formOne.star, _this.formOne.end);
		},
	},
    methods: {
        /*
            1、handleCurrentChange：每一次进行翻页时候触发的方法
            2、getData：获取到全部的数据
        */ 
        handleCurrentChange(val) {
			var _this = this;
			_this.formOne.page = val;
			_this.formOne.star = (_this.formOne.page-1)*20;
			_this.formOne.end = _this.formOne.star+20;
		},
        getData() {
            var _this = this;
            let url = allget+"/Base/getAllDataAnother";
            let params = {
                date_s: baseConfig.changeDateTime(this.formOne.params.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.params.startDate[1], 0),
                channel: _this.formOne.params.channelId.join(','),
            };
            axios.get(url, { params: params })
                .then((res) => {
                    if(res.data.ret) {
                        _this.formOne.tabData = res.data.data;
                        _this.formOne.totalpage = res.data.data.length;
                    } else {
                        baseConfig.warningTipMsg(_this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
        },
    },
    mounted() {
        var _this = this;
        _this.searchPageHeight = searchPageHeight;
        _this.getData();
        var id = store.state.user.channelid.split(",");
        var name = store.state.user.channelname.split(","); 
        for(var i=0; i<id.length; i++){
            _this.formOne.params.channelData[id[i]] = name[i];
        }
    }
};
</script>

<style lang="css" scoped>
.search-span {
    float: right;
}
#searchBtn {
    margin-right: 50px;
}
</style>
