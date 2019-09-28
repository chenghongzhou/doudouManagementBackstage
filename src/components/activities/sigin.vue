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
                <el-table-column prop="uid_count" label="任意任务的完成人数"></el-table-column>
				<el-table-column prop="task_1" label="任务一的完成人数"></el-table-column>
                <el-table-column prop="task_2" label="任务二的完成人数"></el-table-column>
                <el-table-column prop="task_3" label="任务三的完成人数"></el-table-column>
                <el-table-column prop="task_4" label="任务四的完成人数"></el-table-column>
                <el-table-column prop="task_5" label="任务五的完成人数"></el-table-column>
                <el-table-column prop="task_6" label="任务六的完成人数"></el-table-column>
                <el-table-column prop="task_7" label="任务七的完成人数"></el-table-column>
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
            var url = '/NewActivity/getWebCelebrityResultData';
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
