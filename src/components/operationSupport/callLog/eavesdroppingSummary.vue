<template>
	<!-- 偷听汇总 -->
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
            style="width:100%;" 
            v-loading="listLoading" 
            :height="tableHeight">
				<el-table-column prop="date" label="日期"></el-table-column>
				<el-table-column prop="listen_times" label="偷听发起次数"></el-table-column>
				<el-table-column prop="woman_listen_times" label="女方偷听发起次数"></el-table-column>
				<el-table-column prop="man_listen_times" label="男方偷听发起次数"></el-table-column>
				<el-table-column prop="new_listen_times" label="新用户偷听发起次数"></el-table-column>
				<el-table-column prop="old_listen_times" label="老用户偷听发起次数"></el-table-column>
				<el-table-column prop="listen_nums" label="偷听发起人数"></el-table-column>
				<el-table-column prop="listen_success_times" label="偷听成功次数"></el-table-column>
				<el-table-column prop="listen_success_nums" label="偷听成功人数"></el-table-column>
				<el-table-column prop="listen_valid_times" label="有效偷听次数"></el-table-column>
				<el-table-column prop="listen_valid_nums" label="有效偷听人数"></el-table-column>
				<el-table-column prop="listen_long" label="总偷听时长">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.listen_long)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="avg" label="总偷听平均时长"></el-table-column>
				<el-table-column prop="listen_valid_long" label="有效偷听时长">
                    <template slot-scope="scope">
                        <div slot="reference" class="name-wrapper">
                            <P>{{timeTransform(scope.row.listen_valid_long)}}</P>
                        </div>
                    </template>
                </el-table-column>
				<el-table-column prop="listen_money" label="纯偷听消费"></el-table-column>
				<el-table-column prop="gift_money" label="偷听者礼物消费"></el-table-column>
				<el-table-column prop="one_num" label="0~1分钟"></el-table-column>
				<el-table-column prop="three_num" label="1~3分钟"></el-table-column>
				<el-table-column prop="five_num" label="3~5分钟"></el-table-column>
				<el-table-column prop="ten_num" label="5~10分钟"></el-table-column>
				<el-table-column prop="thirty_num" label="10~30分钟"></el-table-column>
				<el-table-column prop="sixty_num" label="30~60分钟"></el-table-column>
				<el-table-column prop="sixtymore_num" label="大于60分钟   "></el-table-column>
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
            let url = "/Record/getListenData";
            axios.get(allget+url, {params: params})
                .then((res) => {
                    _this.listLoading = false;
                    if (res.data.ret) {
                            for(var i = 0;i<res.data.data.length;i++){
                                res.data.data[i].avg = baseConfig.changeTime(res.data.data[i].listen_long/res.data.data[i].listen_success_times);
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
