<template>
	<!-- 动态统计 -->
	<section>
		<el-col 
            :span="24" 
            class="toolbar" 
            style="padding-bottom:0px;">
                <el-form 
                :inline="true" 
                style="overflow:hidden;" 
                :model="formSearch">
                    <el-form-item>
                        <div class="block">
                            <span class="registerTime">日期</span>
                            <el-date-picker 
                            v-model="formSearch.choiceDate" 
                            type="daterange" 
                            range-separator=" 至 " 
                            placeholder="选择日期范围"></el-date-picker>
                        </div>
                    </el-form-item>
                    <el-form-item>
					<el-button 
					type="primary" 
					@click="getData">查询</el-button>
				</el-form-item>
                </el-form>
            </el-col>
			<!--数据展示列表-->
			<template>
				<el-table 
				ref="tableHeight" 
				:data="tabData" 
				border fit highlight-current-row 
				style="width:100%;" 
				:height="searchPageHeight">
					<el-table-column prop="date" label="日期" width="300"></el-table-column>
					<el-table-column prop="content_record" label="动态发布数量"></el-table-column>
                    <el-table-column prop="content_praise" label="合计点赞数量"></el-table-column>
                    <el-table-column prop="content_review" label="合计评论数"></el-table-column>
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
            formSearch : {
				choiceDate: [new Date()-180*24*60*60*1000, new Date()], 
				key_word : '',
			},
			searchPageHeight: null,
			formOne: {
				tabData: [],
				totalPage: 1000, 
                page: 0,
                star: '0',
                end: '20',
			},
			numDialog: {
				dialogShow: false,
				content: '',
                id: '',
                sort:'',
            },
            addDialog: {
               dialogShow: false,
				content: '',
                id: '',
                sort:'',
            },
			formLabelWidth: '130px',
		};
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
            _this.formOne.page = val;
            _this.formOne.star = (_this.formOne.page)*20;
            _this.formOne.end = _this.formOne.star+20;
		},
		// 获取家族数据统计的数据
		getData() {
			var _this = this ;
			var url = '/NewContent/getContentStatList';
			var params = {
				start_date : baseConfig.changeDateTime(_this.formSearch.choiceDate[0], 0),
                end_date : baseConfig.changeDateTime(_this.formSearch.choiceDate[1], 0),
                page: _this.formOne.page
			};
			axios.get(allget+url, { params: params })
			.then((res) => {
				if(res.data.ret) {
					_this.formOne.tabData = res.data.data;
				} else {
					baseConfig.warningTipMsg(_this, res.data.msg);
				}
			}).catch((error) => {
				console.log(error);
			});
        },
    },
    computed:{
        tabData(){
            var _this = this;
            return _this.formOne.tabData.slice(_this.formOne.star,_this.formOne.end);
        }
    },
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.searchPageHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getData();
		})
	}
};
</script>

<style lang="css" scoped>
/* 引入的图片展示样式 */
.fileinput{
	float: left;
	margin-top: 8px;
}
/* 页面样式css内容 */
.excelBox{
	width: 500px; height: 270px; margin-left: -150px; background: #f1f7ff;
	position: absolute; left: 50%; top: 15%; z-index: 1000;
}
p{ margin: 0; }
.excelBox>p{
	width:100%; height: 50px; line-height: 50px; font-weight:bold;
	background: #e3efff; text-align:center;
}
.excelBox .excelInput{
	width:100%; height: 60px;
}
.excelBox .select{
	width:100%; height: 80px;
}
.excelBox .excelInput p,
.excelBox .select p{
	width:100%; height: 36px; text-indent: 20px; line-height: 36px;
}
.excelBox .excelInput input{
    width:300px; display:block; margin: 0 auto;
}
.excelBox .select>div{
	width:300px; display:block; margin: 0 auto;
}
.btns{
    width:100%; height: 50px;
}
.btns button{
    width: 80px; height: 40px; text-align:center; line-height: 40px;
    border: none; border-radius: 5px;
    background-color: #78B2FF; margin-top:20px; color: #fff;
}
.btns button:nth-of-type(1){
    margin-left: 150px; cursor: pointer;
}
.btns button:nth-of-type(2){
    margin-left: 50px; cursor: pointer;
}
.demo-ruleForm{
	width: 500px;
}
.el-tab-pane{
	height: 800px;
}
</style>
