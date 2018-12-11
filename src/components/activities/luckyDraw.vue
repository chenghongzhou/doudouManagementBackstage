<template>
	<!-- 抽奖活动 -->
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
                    <el-button 
                    type="primary" 
                    @click.native.prevent="handleDownload">导出</el-button>
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
                <el-table-column prop="uid" label="UID"></el-table-column>
                <el-table-column label="用户昵称">
                    <template slot-scope="scope">{{ scope.row.nickname || '--'}}</template>
                </el-table-column>
				<el-table-column prop="left_chance" label="抽奖剩余次数"></el-table-column>
				<el-table-column prop="count_total" label="已抽奖次数"></el-table-column>
				<el-table-column prop="count_100" label="iPhone Xs "></el-table-column>
				<el-table-column prop="count_0" label="8888 豆币"></el-table-column>
				<el-table-column prop="count_1" label="888 豆币"></el-table-column>
				<el-table-column prop="count_2" label="168 豆币"></el-table-column>
				<el-table-column prop="count_3" label="88 豆币"></el-table-column>
				<el-table-column prop="count_4" label="8 豆币"></el-table-column>
				<el-table-column prop="chance" label="获取抽奖次数"></el-table-column>
				<el-table-column prop="chance_gold_egg" label="砸蛋获取抽奖次数"></el-table-column>
				<el-table-column prop="chance_gift" label="送礼物获取抽奖次数"></el-table-column>
                <el-table-column prop="chance_invite" label="好友注册数量"></el-table-column>
                <el-table-column label="好友id">
                    <template slot-scope="scope">{{ scope.row.friend_list || '--'}}</template>
                </el-table-column>
                <el-table-column prop="recharge" label="好友充值金额"></el-table-column>
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
import Event from './../../public_js/event.js';
import { allget } from '../../api/api';
import store from '../../vuex/store';
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
            },
			listLoading: false,
		};
    },
    computed: {
        onePageTabData() {
            var _this = this;
            console.log(_this.formOne.tabData)
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
            var url = '/NewActivity/getLotteryStat';
			var params = {
               // type: _this.formOne.type,
               start_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0):'',
               end_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0):'',
               user_id : _this.formOne.uid
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
                            obj.nickname = res.data.data[i].nickname;
                            obj.left_chance = res.data.data[i].left_chance;
                            obj.count_total = res.data.data[i].count_total;
                            obj.count_100 = res.data.data[i].count_100;
                            obj.count_0 = res.data.data[i].count_0;
                            obj.count_1 = res.data.data[i].count_1;
                            obj.count_2 = res.data.data[i].count_2;
                            obj.count_3 = res.data.data[i].count_3;
                            obj.count_4 = res.data.data[i].count_4;
                            obj.chance = res.data.data[i].chance;
                            obj.chance_gold_egg = res.data.data[i].chance_gold_egg;
                            obj.chance_gift = res.data.data[i].chance_gift;
                            obj.chance_invite = res.data.data[i].chance_invite;
                            obj.recharge = res.data.data[i].recharge;
                            obj.friend_list = res.data.data[i].friend_list;
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
        //导出
        handleDownload() {
			require.ensure([], () => {
			const { export_json_to_excel } = require('../vendor/Export2Excel');
			const tHeader = ['uid','昵称','抽奖剩余次数','已抽奖次数','iPhone Xs','8888 豆币','888豆币','168豆币','88豆币','8豆币','获取抽奖次数','砸蛋获取抽奖次数','送礼物获取抽奖次数','好友注册数量','好友充值金额'];
            const filterVal = ['uid','nickname','left_chance','count_total','count_100','count_0','count_1','count_2','count_3','count_4','chance','chance_gold_egg','chance_gift','chance_invite','recharge'];
			const data = this.formatJson(filterVal, this.formOne.tabData);
			export_json_to_excel(tHeader, data, 'excel表');
			})
		},
		formatJson(filterVal, jsonData) {
			return jsonData.map(v => filterVal.map(j => {
			if (j === 'timestamp') {
				return parseTime(v[j])
			} else {
				return v[j]
			}
			}))
		},

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
