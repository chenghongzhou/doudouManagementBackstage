<template>
	<!-- 星星流水记录 -->
	<section>
		<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
			<el-form :inline="true" :model="formOne">
                <el-form-item>
					<div class="block">
						<span class="registerTime">日期</span>
						<el-date-picker 
                        v-model="formOne.choiceDate" 
                        type="daterange" 
                        range-separator=" 至 " 
                        placeholder="选择日期范围"></el-date-picker>
					</div>
				</el-form-item>
                <el-form-item>
					<el-input 
                    placeholder="UID" 
                    style="width:120px;" 
                    v-model="formOne.uid" 
                    auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item>
					<el-button 
                    type="primary" 
                    @click.native.prevent="getTableData">查询</el-button>
                    <!-- <el-button 
                    type="primary" 
                    @click.native.prevent="handleDownload">导出</el-button> -->
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
            ref="tableHeight" 
            :data="onePageTabData" 
            border fit highlight-current-row 
            v-loading="listLoading" 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="time" label="时间"></el-table-column>
                <el-table-column prop="uid" label="UID"></el-table-column>
                <el-table-column prop="trade_type" label="流水类型">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<p v-if="scope.row.trade_type == 103">完成任务</p>
							<p v-else-if="scope.row.trade_type==104">抽奖</p>
							<p v-else-if="scope.row.trade_type==102">购物</p>
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="change_num" label="数额"></el-table-column>
				<el-table-column prop="name" label="兑换物品"></el-table-column>
				<el-table-column prop="buyPrve" label="交易前数额"></el-table-column>
				<el-table-column prop="num" label="交易后数额"></el-table-column>
			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="oneHandleCurrentChange" 
                :page-size="20" 
                :total="formOne.totalPage" 
                style="float:right;"></el-pagination>
			</el-col>
		</template>
	</section>
</template>

<script>
import Event from './../../../public_js/event.js';
import { allget } from '../../../api/api';
import store from '../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null,
			formOne: {
                choiceDate: [new Date()-30*24*60*60*1000, new Date()],
                uid : '',
                tabData: [],
                totalPage: null,
                star: '0',
                end: '20',
                page:0
            },
			listLoading: false,
		};
    },
    computed: {
        onePageTabData() {
            var _this = this;
            return _this.formOne.tabData.slice(_this.formOne.star, _this.formOne.end);
        },
    },
	methods: {
        oneHandleCurrentChange(val) {
			var _this = this;
            val = val-1;
            _this.formOne.page = val;
            _this.formOne.star = (_this.formOne.page)*20;
            _this.formOne.end = _this.formOne.star+20;
        },
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
            var url = '/NewTreasureBox/getStarList';
			var params = {
               // type: _this.formOne.type,
               start_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0):'',
               end_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0):'',
               user_id : _this.formOne.uid,
               page:_this.formOne.page,
            };
            axios.get(allget+url, { params: params })
            .then((res) => {
                _this.listLoading = false;
                 var arr = [];
                if(res.data.ret) {
                    for(var i=0; i<res.data.data.length; i++) {
                        var obj = {};
                        for(var key in res.data.data[i]) {
                            obj.uid = res.data.data[i].uid;
                            obj.time = res.data.data[i].time;
                            obj.change_num = res.data.data[i].change_num;

                            obj.buyPrve = Number(res.data.data[i].num) - Number(res.data.data[i].change_num);
                            obj.num = res.data.data[i].num;
                            obj.trade_type = res.data.data[i].trade_type;
                            obj.name = res.data.data[i].name;
                        }
                        arr.push(obj);
                    }
                    _this.formOne.totalPage = arr.length;
                    _this.formOne.tabData = arr;
                }
                else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }
            })
            .catch((err) => {
                console.log(err);
            });
        },
        // //导出
        // handleDownload() {
		// 	require.ensure([], () => {
		// 	const { export_json_to_excel } = require('../vendor/Export2Excel');
		// 	const tHeader = ['uid','昵称','抽奖剩余次数','已抽奖次数','iPhone Xs','8888 豆币','888豆币','168豆币','88豆币','8豆币','获取抽奖次数','砸蛋获取抽奖次数','送礼物获取抽奖次数','好友注册数量','好友充值金额'];
        //     const filterVal = ['uid','nickname','left_chance','count_total','count_100','count_0','count_1','count_2','count_3','count_4','chance','chance_gold_egg','chance_gift','chance_invite','recharge'];
		// 	const data = this.formatJson(filterVal, this.formOne.tabData);
		// 	export_json_to_excel(tHeader, data, 'excel表');
		// 	})
		// },
		// formatJson(filterVal, jsonData) {
		// 	return jsonData.map(v => filterVal.map(j => {
		// 	if (j === 'timestamp') {
		// 		return parseTime(v[j])
		// 	} else {
		// 		return v[j]
		// 	}
		// 	}))
		// },

	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = searchPageHeight;
			_this.getTableData();
		});
	}
};
</script>

<style lang="css" scoped>
</style>
