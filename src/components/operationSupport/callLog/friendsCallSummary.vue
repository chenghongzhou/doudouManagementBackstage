<template>
	<!-- 好友通话汇总 -->
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
            style="width:100%;" 
            :height="tableHeight">
				<el-table-column prop="time" label="日期"></el-table-column>
				<el-table-column prop="manredclick" label="发起次数（男发起）"></el-table-column>
				<el-table-column prop="man_num" label="接通次数（男发起）"></el-table-column>
				<el-table-column prop="man_rate" label="接通率（男发起）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{(scope.row.man_rate*100).toFixed(2)+"%"}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="man_time" label="通话时长（男发起）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.man_time)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="womanredclick" label="发起次数（女发起）"></el-table-column>
				<el-table-column prop="woman_num" label="接通次数（女发起）"></el-table-column>
				<el-table-column prop="woman_rate" label="接通率（女发起）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{(scope.row.woman_rate*100).toFixed(2)+"%"}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="woman_time" label="通话时长（女发起）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.woman_time)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="freeredclick" label="发起次数（免费）"></el-table-column>
				<el-table-column prop="free_num" label="接通次数（免费）"></el-table-column>
				<el-table-column prop="free_rate" label="接通率（免费）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{(scope.row.free_rate*100).toFixed(2)+"%"}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="free_time" label="通话时长（免费）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.free_time)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="moneyredclick" label="发起次数（付费）"></el-table-column>
				<el-table-column prop="money_num" label="接通次数（付费）"></el-table-column>
				<el-table-column prop="money_rate" label="接通率（付费）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{(scope.row.money_rate*100).toFixed(2)+"%"}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="money_time" label="通话时长（付费）">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.money_time)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="totalredclick" label="总发起次数"></el-table-column>
				<el-table-column prop="total_num" label="总接通次数"></el-table-column>
				<el-table-column prop="total_rate" label="总接通率">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{(scope.row.total_rate*100).toFixed(2)+"%"}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="total_time" label="总通话时长">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.total_time)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="call_money" label="总通话消费"></el-table-column>
				<el-table-column prop="listen_valid_times" label="有效通话次数"></el-table-column>
				<el-table-column prop="listen_valid_long" label="有效通话时长">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.listen_valid_long)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="avg" label="总通话平均时长"></el-table-column>
				<el-table-column prop="gift_money" label="礼物消费"></el-table-column>
				<el-table-column prop="one_num" label="0~1分钟"></el-table-column>
				<el-table-column prop="three_num" label="1~3分钟"></el-table-column>
				<el-table-column prop="five_num" label="3~5分钟"></el-table-column>
				<el-table-column prop="ten_num" label="5~10分钟"></el-table-column>
				<el-table-column prop="thirty_num" label="10~30分钟"></el-table-column>
				<el-table-column prop="sixty_num" label="30~60分钟"></el-table-column>
				<el-table-column prop="sixtymore_num" label="大于60分钟"></el-table-column>
			</el-table>
		</template>
	</section>
</template>

<script>
import { allget } from "../../../api/api";
import axios from "axios";
export default {
    data() {
        return {
            tableHeight: null,
            formOne: {
				startDate: [new Date()-7*24*60*60*1000, new Date()],
            },
            listData: [],
            formLabelWidth: "120px",
            listLoading: false,
        };
    },
    methods: {
        // 获取数据
        getData() {
            var _this = this;
            _this.listLoading = true;
			let params = _this.condition();
            let url = "/Record/getFriendData";
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                            for(var i = 0;i<res.data.data.length;i++){
                                res.data.data[i].avg = baseConfig.changeTime(res.data.data[i].total_time/res.data.data[i].total_num);
                            }
                        this.listData = res.data.data;
                    } else {
                        baseConfig.warningTipMsg(this, res.data.msg);
                    }
                })
                .catch((err) => {
                    baseConfig.warningTipMsg(this, "error"); 
                });
		},
		condition() {
            return {
                date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
            };
        },
        // 时间转换
        timeTransform(oldValue){
            return baseConfig.changeTime(oldValue);
        }
    },
    mounted() {
        var _this = this;
        _this.tableHeight = baseConfig.lineNumber(searchHeight);
        _this.getData();
    }
};
</script>

<style lang="css" scoped>

</style>
