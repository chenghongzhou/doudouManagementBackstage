<template>
<!-- 修改密码 -->
<section>
    <template>
        <el-table 
        ref="tableHeight" 
        :data="tablList" 
        border fit highlight-current-row 
        style="width:100%;" 
        :height="tableHeight">
            <el-table-column prop="code" label="代号" width="300"></el-table-column>
            <el-table-column prop="name" label="名称" width="300"></el-table-column>
            <el-table-column prop="start_time" label="开始时间"></el-table-column>
            <el-table-column prop="end_time" label="结束时间"></el-table-column>
            <el-table-column label="操作" width="300">
                <template slot-scope="scope">
                    <el-button 
                    type="primary" 
                    @click.native.prevent="setTime(scope.row)" 
                    size="small">设置时间</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-dialog 
        title="时间设置"
        :visible.sync="dialogShow">
             <el-form label-position="right" :model="formOne">
                <el-form-item label="发布时间" :label-width="formLabelWidth">
                        <el-date-picker 
                        v-model="formOne.start_time" 
                        type="datetime" 
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="选择日期范围" style="width:30%"></el-date-picker>
                </el-form-item>
                <el-form-item label="发布时间" :label-width="formLabelWidth">
                        <el-date-picker 
                        v-model="formOne.end_time" 
                        type="datetime" 
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="选择日期范围" style="width:30%"></el-date-picker>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer" style="margin-top:50px;margin-left:200px;">
                 <el-button 
                    @click="editorSure(0)">取 消</el-button>
                <el-button 
                type="primary" 
                @click="editorSure(1)">确 定</el-button>
            </div>       
        </el-dialog>
        <el-col :span="24" class="toolbar">
            <el-pagination 
            layout="total,prev,pager,next,jumper" 	
            @current-change="oneHandleCurrentChange" 
            :page-size="20" 
            :total="totalPage" 
            style="float:right;"></el-pagination>
        </el-col>
    </template>
</section>
</template>
<script>
import Event from './../../../public_js/event.js';
import { allget, wechatget} from '../../../api/api';
import store from '../../../vuex/store';
import axios from 'axios';
export default {
	data() {
		return {
			tableHeight: null, 
            formLabelWidth: "120px",
            list:[],
			formOne: { // 编辑
				start_time: '',
                end_time: '',
                code:''
			},
			listLoading: false,
            dialogShow:false,
            totalPage: 1000,
            star: '0',
            end: '20',
            page:1,
		};
	},
    computed: {
        tablList(){
            var _this = this;
            return _this.list.slice(_this.star,_this.end);
        },
    },
	methods: {
        oneHandleCurrentChange(val) {
			var _this = this;
            val = val-1;
            _this.page = val;
            _this.star = (_this.page)*20;
            _this.end = (_this.star)+20;
        },
        getData(){
             var _this = this;
            var params={
                page:_this.page,
            };
            axios.get(allget+'/NewActivity/getActivityTimeList', {params: params})
            .then((res) => {
                _this.listLoading = false;
                if(res.data.ret) {
                    _this.list = res.data.data;
                } else {
                    baseConfig.warningTipMsg(_this, res.data.msg);
                }

            })
            .catch((err) => {
                console.log(err);
            });
        },
        setTime(row){
            var _this = this;
            _this.dialogShow = true;
            _this.formOne.start_time = row.start_time;
            _this.formOne.end_time = row.end_time;
            _this.formOne.code = row.code;
        },
        editorSure(type) {
            var _this = this;
            if(type == 0){
                _this.dialogShow = false;
                return false;
            };
            var params={
                code:_this.formOne.code,
                start_time:_this.formOne.start_time,
                end_time:_this.formOne.end_time,
            };
            axios.get(allget+'/NewActivity/setActivityTime', {params: params})
            .then((res) => {
                _this.listLoading = false;
                if(res.data.ret) {
                     _this.dialogShow = false;
                    baseConfig.successTipMsg(_this, '设置成功');
                    _this.getData();
                } else {
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
		_this.$nextTick(function() {
            _this.getData();
            _this.tableHeight = searchPageHeight+60; 
		});
	},
};
</script>
<style lang="css" scoped>
/* 页面样式css内容 */
.excelBox{
	width: 500px; height: 190px; margin-left: -150px; background: #f1f7ff;
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
.content-wrapper{
    background: #fdfdfd !important;
}
</style>