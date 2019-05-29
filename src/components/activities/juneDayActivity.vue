<template>
	<!-- 六一 -->
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
				</el-form-item>
			</el-form>
		</el-col>
		<template>
			<el-table 
            ref="tableHeight" 
            :data="formOne.tabData" 
            border fit highlight-current-row 
            v-loading="listLoading" 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="uid" label="用户ID"></el-table-column>
				<el-table-column prop="prop_1" label="图像框领取数量"></el-table-column>
                <el-table-column prop="prop_2" label="座驾领取数量"></el-table-column>
				<el-table-column prop="money_30" label="30元充值笔数"></el-table-column>
                <el-table-column prop="money_100" label="100元充值笔数"></el-table-column>
				<el-table-column prop="money_200" label="200元充值笔数"></el-table-column>
                <el-table-column prop="money_500" label="500元充值笔数"></el-table-column>
				<el-table-column prop="money_1000" label="1000元充值笔数"></el-table-column>
                <el-table-column prop="money_3000" label="3000元充值笔数"></el-table-column>
				<el-table-column prop="money_5000" label="5000元充值笔数"></el-table-column>
                <el-table-column prop="money_10000" label="10000元充值笔数"></el-table-column>
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
                totalPage: 1000,
                star: '0',
                end: '20',
                page:1,
            },
			listLoading: false,
		};
    },
	methods: {
        oneHandleCurrentChange(val) {
			var _this = this;
            _this.formOne.page = val;
            _this.getTableData();
        },
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
            var url = '/NewActivity/getChildrenDayData';
			var params = {
               start_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0):'',
               end_date : _this.formOne.choiceDate?baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0):'',
               uid : _this.formOne.uid,
               page:_this.formOne.page,
            };
            axios.get(allget+url, { params: params })
            .then((res) => {
                _this.listLoading = false;
                if(res.data.ret) {
                    if(res.data.data.total){
                        res.data.data.total.uid = '总计';
                        res.data.data.list.unshift(res.data.data.total);
                    };
                    _this.formOne.tabData = res.data.data.list;
                }
                else {
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
		this.$nextTick(function() {
			_this.tableHeight = searchPageHeight;
			_this.getTableData();
		});
	}
};
</script>

<style lang="css" scoped>
</style>
