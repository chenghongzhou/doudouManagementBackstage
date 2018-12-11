<template>
    <!-- 渠道排行 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;">
            <el-form :inline="true" style="overflow:hidden;">
                <el-form-item>
                    <div class="block">
                        <span class="registerTime">日期</span>
                        <el-date-picker 
                        v-model="formOne.params.startDate" 
                        type="date" 
                        placeholder="选择日期"></el-date-picker>
                    </div>
                </el-form-item>
                <el-form-item style="margin-left:100px;">
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
            border fit highlight-current-row 
            style="width:100%;" 
            :height="searchPageHeight">
                <el-table-column prop="channel" label="渠道号"></el-table-column>
                <el-table-column prop="device" label="激活"></el-table-column>
                <el-table-column 
                prop="device_up_down" 
                label="激活环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.device_up_down==0">{{Math.abs(scope.row.device_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.device_up_down<0">↓ {{Math.abs(scope.row.device_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.device_up_down>0">↑ {{Math.abs(scope.row.device_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="register" label="注册"></el-table-column>
                <el-table-column 
                prop="register_up_down" 
                label="注册环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.register_up_down==0">{{Math.abs(scope.row.register_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.register_up_down<0">↓ {{Math.abs(scope.row.register_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.register_up_down>0">↑ {{Math.abs(scope.row.register_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="active" label="DAU"></el-table-column>
                <el-table-column 
                prop="active_up_down" 
                label="DAU环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.active_up_down==0">{{Math.abs(scope.row.active_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.active_up_down<0">↓ {{Math.abs(scope.row.active_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.active_up_down>0">↑ {{Math.abs(scope.row.active_up_down)}}</p>
                        </div>
                    </template>    
                </el-table-column>
                <el-table-column prop="consume_user" label="付费人数"></el-table-column>
                <el-table-column 
                prop="consume_user_up_down" 
                label="付费人数环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.consume_user_up_down==0">{{Math.abs(scope.row.consume_user_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.consume_user_up_down<0">↓ {{Math.abs(scope.row.consume_user_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.consume_user_up_down>0">↑ {{Math.abs(scope.row.consume_user_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="total_fee" label="付费金额"></el-table-column>
                <el-table-column 
                prop="total_fee_up_down" 
                label="付费金额环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.total_fee_up_down==0">{{Math.abs(scope.row.total_fee_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.total_fee_up_down<0">↓ {{Math.abs(scope.row.total_fee_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.total_fee_up_down>0">↑ {{Math.abs(scope.row.total_fee_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="consume_rate" label="付费率"></el-table-column>
                <el-table-column 
                prop="consume_rate_up_down" 
                label="付费率环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.consume_rate_up_down==0">{{Math.abs(scope.row.consume_rate_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.consume_rate_up_down<0">↓ {{Math.abs(scope.row.consume_rate_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.consume_rate_up_down>0">↑ {{Math.abs(scope.row.consume_rate_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="consume_arpu" label="付费ARPU值"></el-table-column>
                <el-table-column 
                prop="consume_arpu_up_down" 
                label="付费ARPU值环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.consume_arpu_up_down==0">{{Math.abs(scope.row.consume_arpu_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.consume_arpu_up_down<0">↓ {{Math.abs(scope.row.consume_arpu_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.consume_arpu_up_down>0">↑ {{Math.abs(scope.row.consume_arpu_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="first_time_user" label="新增付费人数"></el-table-column>
                <el-table-column 
                prop="first_time_user_up_down" 
                label="新增付费人数环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.first_time_user_up_down==0">{{Math.abs(scope.row.first_time_user_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.first_time_user_up_down<0">↓ {{Math.abs(scope.row.first_time_user_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.first_time_user_up_down>0">↑ {{Math.abs(scope.row.first_time_user_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="first_time_amount" label="新增充值金额"></el-table-column>
                <el-table-column 
                prop="first_time_amount_up_down" 
                label="新增充值金额环比">
                     <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.first_time_amount_up_down==0">{{Math.abs(scope.row.first_time_amount_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.first_time_amount_up_down<0">↓ {{Math.abs(scope.row.first_time_amount_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.first_time_amount_up_down>0">↑ {{Math.abs(scope.row.first_time_amount_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="new_consume_rate" label="新增付费率"></el-table-column>
                <el-table-column 
                prop="new_consume_rate_up_down" 
                label="新增付费率环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.new_consume_rate_up_down==0">{{Math.abs(scope.row.new_consume_rate_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.new_consume_rate_up_down<0">↓ {{Math.abs(scope.row.new_consume_rate_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.new_consume_rate_up_down>0">↑ {{Math.abs(scope.row.new_consume_rate_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="new_consume_arpu" label="新增付费ARPU"></el-table-column>
                <el-table-column 
                prop="new_consume_arpu_up_down" 
                label="新增付费ARPU环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.new_consume_arpu_up_down==0">{{Math.abs(scope.row.new_consume_arpu_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.new_consume_arpu_up_down<0">↓ {{Math.abs(scope.row.new_consume_arpu_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.new_consume_arpu_up_down>0">↑ {{Math.abs(scope.row.new_consume_arpu_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="ret_one_rate" label="次日留存"></el-table-column>
                <el-table-column 
                prop="ret_one_rate_up_down" 
                label="次日留存环比">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p style="background-color:#aaa;color:#fff;" v-if="scope.row.ret_one_rate_up_down==0">{{Math.abs(scope.row.ret_one_rate_up_down)}}</p>
                            <p style="background-color:green;color:#fff;" v-else-if="scope.row.ret_one_rate_up_down<0">↓ {{Math.abs(scope.row.ret_one_rate_up_down)}}</p>
                            <p style="background-color:red;color:#fff;" v-else-if="scope.row.ret_one_rate_up_down>0">↑ {{Math.abs(scope.row.ret_one_rate_up_down)}}</p>
                        </div>
                    </template>
                </el-table-column>
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
                    startDate: new Date(),
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
            let url = allget+"/Base/getChannelDataRank";
            let params = {
                // date: baseConfig.changeDateTime(this.formOne.params.startDate, 0),
                // channel: _this.formOne.params.channelId.join(','),
                date: '2018-08-01',
                channel: '13',
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
        // _this.getData();
        baseConfig.warningTipMsg(_this, '请选择日期、渠道号进行搜索~');
        var id = store.state.user.channelid.split(",");
        var name = store.state.user.channelname.split(","); 
        for(var i=0; i<id.length; i++){
            _this.formOne.params.channelData[id[i]] = name[i];
        }
    }
};
</script>

<style lang="css" scoped>

</style>
