<template>
	<!-- 实时通话次数 -->
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
				<el-form-item style="margin-left:100px;">
					<span>通话类型</span>
					<el-select style="width: 150px;" v-model="type">
						<el-option label="全部" value="0"></el-option>
						<el-option label="随机" value="1"></el-option>
						<el-option label="好友" value="2"></el-option>
						<el-option label="网红" value="3"></el-option>
						<el-option label="抢聊" value="4"></el-option>
						<el-option label="组组" value="5"></el-option>
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
			v-loading="listLoading" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="time" label="日期"></el-table-column>
				<el-table-column prop="day_best" label="当天最高峰"></el-table-column>
				<el-table-column prop="a" label="00:00~00:59"></el-table-column>
				<el-table-column prop="b" label="01:00~01:59"></el-table-column>
				<el-table-column prop="c" label="02:00~02:59"></el-table-column>
				<el-table-column prop="d" label="03:00~03:59"></el-table-column>
				<el-table-column prop="e" label="04:00~04:59"></el-table-column>
				<el-table-column prop="f" label="05:00~05:59"></el-table-column>
				<el-table-column prop="g" label="06:00~06:59"></el-table-column>
				<el-table-column prop="h" label="07:00~07:59"></el-table-column>
				<el-table-column prop="i" label="08:00~08:59"></el-table-column>
				<el-table-column prop="j" label="09:00~09:59"></el-table-column>
				<el-table-column prop="k" label="10:00~10:59"></el-table-column>
				<el-table-column prop="l" label="11:00~11:59"></el-table-column>
				<el-table-column prop="m" label="12:00~12:59"></el-table-column>
				<el-table-column prop="n" label="13:00~13:59"></el-table-column>
				<el-table-column prop="o" label="14:00~14:59"></el-table-column>
				<el-table-column prop="p" label="15:00~15:59"></el-table-column>
				<el-table-column prop="q" label="16:00~16:59"></el-table-column>
				<el-table-column prop="r" label="17:00~17:59"></el-table-column>
				<el-table-column prop="s" label="18:00~18:59"></el-table-column>
				<el-table-column prop="t" label="19:00~19:59"></el-table-column>
				<el-table-column prop="u" label="20:00~20:59"></el-table-column>
				<el-table-column prop="v" label="21:00~21:59"></el-table-column>
				<el-table-column prop="w" label="22:00~22:59"></el-table-column>
				<el-table-column prop="x" label="23:00~23:59"></el-table-column>
			</el-table>
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
			type: "",
			page: 0,
			totalpage: null,
			star: '0',
			end: '20',
        };
	},
	computed: {
		onePageTabData() {
			var _this = this;
			return _this.listData.slice(_this.star, _this.end);
		},
	},
    methods: {
		handleCurrentChange(val) {
			var _this = this;
			_this.page = val-1;
			_this.star = (_this.page)*20;
			_this.end = _this.star+20;
		},
        // 获取数据
        getData() {
			var _this = this;
			_this.listLoading = true;
            let url = "/Base/getCallNum";
			let params = _this.condition();
			axios.get(allget+url, {params: params})
                .then((res) => {
					_this.listLoading = false;
                    if (res.data.ret) {
						this.listData = res.data.data;
						_this.totalpage = res.data.data.length;
                    } else {
                        baseConfig.warningTipMsg(this, res.data.msg);
                    }
                })
                .catch((err) => {
                    console.log(err);
                });
		},
		condition() {
			if(this.type !== "" || this.type !== null){
				return {
                	date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
                	date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
                	type: this.type
            	};
			}else {
				return{
					date_ms: baseConfig.changeDateTime(this.formOne.month, 2),
					date_me: baseConfig.changeDateTime(this.formOne.month, 2),
				}
			}
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
