<template>
	<!-- 红包雨 -->
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
					<el-button 
                    type="primary" 
                    @click.native.prevent="getTableData">查询</el-button>
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
                <el-table-column prop="date" label="日期"></el-table-column>
				<el-table-column prop="people" label="活动参与人数"></el-table-column>
				<el-table-column prop="down_rain" label="准点红包数量"></el-table-column>
				<el-table-column prop="req_rain" label="求雨红包数量"></el-table-column>
				<el-table-column prop="count_0" label="有钱装扮"></el-table-column>
				<el-table-column prop="count_1" label="灯笼礼物"></el-table-column>
				<el-table-column prop="count_2" label="爆竹装扮"></el-table-column>
				<el-table-column prop="count_3" label="猪宝宝礼物"></el-table-column>
				<el-table-column prop="count_4" label="猪年大吉座驾"></el-table-column>
				<el-table-column prop="count_5" label="猪猪装扮"></el-table-column>
				<el-table-column prop="count_6" label="猪福装扮"></el-table-column>
				<el-table-column prop="count_7" label="花旦装扮"></el-table-column>
                <el-table-column prop="count_8" label="豆币"></el-table-column>
				<el-table-column prop="count_9" label="2019礼物"></el-table-column>
				<el-table-column prop="count_10" label="财神到礼物"></el-table-column>
                <el-table-column prop="count_11" label="财神帽装扮"></el-table-column>
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
				page:0
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
            var url = '/NewActivity/getRedPackageRainData';
			var params = {
               // type: _this.formOne.type,
               start_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0):'',
			   end_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0):'',
			   page : _this.formOne.page
            };
            axios.get(allget+url, { params: params })
            .then((res) => {
                _this.listLoading = false;
                var arr = [];
                if(res.data.ret) {
                    for(var i=0; i<res.data.data.length; i++) {
                        var obj = {};
                        for(var key in res.data.data[i]) {
                            obj.date = res.data.data[i].date;
                            obj.people = res.data.data[i].people;
                            obj.down_rain = res.data.data[i].down_rain;
                            obj.req_rain = res.data.data[i].req_rain;
                            obj.count_0 = res.data.data[i].有钱装扮;
                            obj.count_1 = res.data.data[i].灯笼礼物;
                            obj.count_2 = res.data.data[i].爆竹装扮;
                            obj.count_3 = res.data.data[i].猪宝宝礼物;
                            obj.count_4 = res.data.data[i].猪年大吉座驾;
                            obj.count_5 = res.data.data[i].猪猪装扮;
                            obj.count_6 = res.data.data[i].猪福装扮;
                            obj.count_7 = res.data.data[i].花旦装扮;
                            obj.count_8 = res.data.data[i].豆币;
                            obj.count_9 = res.data.data[i]['2019礼物'];
                            obj.count_10 = res.data.data[i].财神到礼物;
							obj.count_11 = res.data.data[i].财神帽装扮;
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
