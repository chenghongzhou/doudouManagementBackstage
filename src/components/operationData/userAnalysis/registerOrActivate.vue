<template>
    <!-- 激活与注册 -->
    <section>
        <el-col :span="24" class="toolbar" style="padding-bottom:0px;over-flow:hidden;">
            <el-form :inline="true" style="overflow:hidden;" :model="formOne">
                <el-form-item style="z-index:999;">
                    <div class="block">
                        <span class="registerTime">日期</span>
                        <el-date-picker 
                        v-model="formOne.choiceDate" 
                        type="daterange" 
                        range-separator=" 至 " 
                        placeholder="选择日期范围"></el-date-picker>
                    </div>
                </el-form-item>
                <el-form-item style="margin-left:100px;">
                    <span>渠道</span>
                    <el-select 
                    v-model="channelId" 
                    multiple 
                    collapse-tags 
                    style="margin-left:20px;" 
                    placeholder="请选择">
                        <el-option 
                        v-for="(item, key) of channelData" 
                        :key="key" 
                        :label="item" 
                        :value="key"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <el-button 
                    type="primary" 
                    @click="chartLineShow">折线图</el-button>
                    <el-button 
                    type="primary" 
                    @click="getTableData">查询</el-button>
                </el-form-item>
            </el-form>
        </el-col>
        <div class="total_data" id="total_data">
            <el-row>
                <el-col :span="4">
                    <div class="grid-content bg-purple">累计</div>
                </el-col>
                <el-col :span="9">
                    <div class="grid-content bg-purple">设备激活：{{totleTableData.total_device}}</div>
                </el-col>
                <el-col :span="9">
                    <div class="grid-content bg-purple-light">新增注册：{{totleTableData.total_register}}</div>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="4">
                    <div class="grid-content bg-purple">合计</div>
                </el-col>
                <el-col :span="9">
                    <div class="grid-content bg-purple">设备激活：{{totleTableData.total_date_device}}</div>
                </el-col>
                <el-col :span="9">
                    <div class="grid-content bg-purple-light">新增注册：{{totleTableData.total_date_register}}</div>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="4">
                    <div class="grid-content bg-purple">平均</div>
                </el-col>
                <el-col :span="9">
                    <div class="grid-content bg-purple">设备激活：{{totleTableData.avg_device}}</div>
                </el-col>
                <el-col :span="9">
                    <div class="grid-content bg-purple-light">新增注册：{{totleTableData.avg_register}}</div>
                </el-col>
            </el-row>
        </div>
        <template>
            <el-table 
            ref="tableHeight" 
            :data="onePageTabData" 
            border 
            v-loading="listLoading" 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="time" label="日期" sortable></el-table-column>
                <el-table-column prop="device" label="设备激活" sortable></el-table-column>
                <el-table-column prop="register" label="新增注册" sortable></el-table-column>
                <el-table-column prop="rate" label="转化率" sortable>
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <p v-if="scope.row.rate==0">--</p>
                            <p v-else>{{(scope.row.rate*100).toFixed(2)+'%'}}</p>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column prop="man_register" label="男用户注册数" sortable></el-table-column>
                <el-table-column prop="woman_register" label="女用户注册数" sortable></el-table-column>
            </el-table>
            <!-- 折线图组建 -->
            <el-dialog 
            title="折线图" 
            :visible.sync="dialogVisible" 
            @open="show">
                <div 
                class="chartLine" 
                style="width:100%;height:600px;"></div>
            </el-dialog>
            <el-col :span="24" class="toolbar">
                <el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="handleCurrentChange" 
                :page-size="20" 
                :total="totalpage" 
                style="float:right;"></el-pagination>
            </el-col>
        </template>
    </section>
</template>

<script>
import Event from "./../../../public_js/event.js";
import { allget } from "../../../api/api";
import store from "../../../vuex/store";
import axios from "axios";
import echarts from "echarts";
export default {
    data() {
        return {
            tableHeight: null, 
            formOne: {
                choiceDate: [new Date()-15*24*60*60*1000, new Date()], 
                channel: [],
                options: []
            },
            listLoading: true, 
            tabData: [], 
            totalpage: null,
            page: 1, 
            star: "0",
            end: "20",
            channelId: null,
            channelData: {},
            dialogVisible: false,
            formLabelWidth: "120px", 
            chartLineData: {
                //用于折线图绘制的数据
                title: {
                    text: "注册与激活"
                },
                tooltip: {
                    trigger: "axis"
                },
                toolbox: {
                    show: true,
                    feature: {
                        mark: { show: true },
                        dataView: { show: true, readOnly: false },
                        magicType: { show: true, type: ["line", "bar"] },
                        restore: { show: true },
                        saveAsImage: { show: true }
                    }
                },
                legend: {
                    data: ["设备激活", "新增注册", "转换率"]
                },
                xAxis: [
                    {
                        type: "category",
                        boundaryGap: false,
                        data: []
                    }
                ],
                yAxis: [
                    {
                        type: "value",
                        name: "设备/注册",
                        axisLabel: {
                            formatter: "{value} "
                        }
                    },
                    {
                        type: "value",
                        name: "付费率",
                        axisLabel: {
                            formatter: "{value} %"
                        }
                    }
                ],
                series: [
                    {
                        name: "设备激活",
                        type: "line",
                        data: []
                    },
                    {
                        name: "新增注册",
                        type: "line",
                        data: []
                    },
                    {
                        name: "转换率",
                        type: "line",
                        yAxisIndex: 1,
                        data: []
                    }
                ]
            },
            totleTableData: {}
        };
    },
    computed: {
        onePageTabData() {
            var _this = this;
            return _this.tabData.slice(_this.star, _this.end);
        }
    },
    methods: {
        handleCurrentChange(val) {
            var _this = this;
            _this.page = val-1;
            _this.star = (_this.page) * 20;
            _this.end = _this.star + 20;
        },
        // 搜索条件
        searchCondition() {
            var _this = this;
            var obj = {};
            obj.date_s = baseConfig.changeDateTime(
                _this.formOne.choiceDate[0],
                0
            );
            obj.date_e = baseConfig.changeDateTime(
                _this.formOne.choiceDate[1],
                0
            );
            (obj.channel = this.channelId.join(",")),
                (obj.sex = _this.formOne.sex);
            return obj;
        },
        // 获取数据列表
        getTableData() {
            var _this = this;
            _this.listLoading = true;
            var url = "/Base/getDeviceActive";
            var params = _this.searchCondition();
            if (params == null) {
                _this.listLoading = false;
            } else {
                axios.get(allget+url, {params: params})
                    .then((res) => {
                        _this.listLoading = false;
                        if (res.data.ret) {
                            _this.totleTableData = res.data.total;
                            _this.tabData = res.data.data;
                            _this.totalpage = res.data.data.length;

                            for (var i = res.data.data.length-1; i >= 0; i--) {
                                _this.chartLineData.xAxis[0].data.push(res.data.data[i].time.slice(5,10));
                                _this.chartLineData.series[0].data.push(this.tabData[i].device);
                                _this.chartLineData.series[1].data.push(this.tabData[i].register);
                                _this.chartLineData.series[2].data.push(this.tabData[i].rate);
                            }
                        } else {
                            baseConfig.warningTipMsg(_this, res.data.msg);
                        }
                    })
                    .catch((err) => {
                        console.log(err);
                    });
            }
        },
        show() {
            this.$nextTick(function() {
                this.chartLine = echarts.init(
                    document.querySelector(".chartLine")
                );
                document.querySelector(".chartLine").style.border =
                    "1px solid #4488BB";
                this.chartLine.setOption(this.chartLineData);
            });
        },
        // 折线图展示
        chartLineShow() {
            this.dialogVisible=true;
        }
    },
    mounted() {
        var _this = this;
        var lookHeight =
            document.documentElement.clientHeight || document.body.clientHeight;
        var windowHeight = lookHeight - 286;
        _this.tableHeight = baseConfig.lineNumber(windowHeight);
        _this.getTableData();
        var id = store.state.user.channelid.split(",");
        var name = store.state.user.channelname.split(",");
        for (var i = 0; i < id.length; i++) {
            this.channelData[id[i]] = name[i];
        }
    }
};
</script>

<style lang="css" scoped>
/* 页面样式css内容 */
#total_data {
    width: 50%;
    height: 30px;
}
.el-row {
    /* margin-bottom: 20px; */
    z-index: 999;
    /* border: 1px solid red; */
    height: 0px;
}
/* .el-col {
    border-radius: 4px;
} */
.bg-purple-dark {
    background: #99a9bf;
}
.bg-purple {
    background: #d3dce6;
}
.bg-purple-light {
    background: #e5e9f2;
}
.grid-content {
    /* border-radius: 4px; */
    border: 1px solid #fff;
    min-height: 36px;
    line-height: 36px;
    text-align:center;
}
.row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
}
</style>
