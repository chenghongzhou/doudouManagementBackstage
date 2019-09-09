<template>
	<!-- 六一 -->
	<section>
		<template>
			<el-table 
            ref="tableHeight" 
            :data="tablList" 
            border fit highlight-current-row 
            v-loading="listLoading" 
            style="width:100%;" 
            :height="tableHeight">
                <el-table-column prop="date" label="日期"></el-table-column>
				<el-table-column prop="count" label="当日参与人数"></el-table-column>
                <el-table-column prop="score" label="日榜奔月值总数"></el-table-column>
				<el-table-column prop="uv" label="页面访问人数"></el-table-column>
                <el-table-column prop="pv" label="	页面访问次数"></el-table-column>
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
    computed: {
        tablList(){
            var _this = this;
            return _this.formOne.tabData.slice(_this.formOne.star,_this.formOne.end);
        }
    },
	methods: {
        oneHandleCurrentChange(val) {
			var _this = this;
            val = val-1;
            _this.formOne.page = val;
            _this.formOne.star = (_this.formOne.page)*20;
            _this.formOne.end = (_this.formOne.star)+20;
        },
		// 获取数据列表
		getTableData() {
			var _this = this ;
			_this.listLoading = true;
            var url = '/NewActivity/getMidAutumnData';
			var params = {
               page:_this.formOne.page,
            };
            axios.get(allget+url, { params: params })
            .then((res) => {
                _this.listLoading = false;
                if(res.data.ret) {
                    _this.formOne.tabData = res.data.data;
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
			_this.tableHeight = searchPageHeight+60;
			_this.getTableData();
		});
	}
};
</script>

<style lang="css" scoped>
</style>
