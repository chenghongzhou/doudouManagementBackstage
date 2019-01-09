<template>
	<!-- 砍价 -->
	<section>
		<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
			<el-form :inline="true" :model="formOne">
				<el-form-item>
					<span>类型</span>
					<el-select style="width: 120px;" v-model="formOne.type">
						<el-option label="全部" value="0"></el-option>
						<el-option label="座驾" value="1"></el-option>
                        <el-option label="图像框" value="2"></el-option>
						<el-option label="道具" value="3"></el-option>
						<el-option label="礼物" value="4"></el-option>
					</el-select>
				</el-form-item>
                <el-form-item>
					<el-input 
                    placeholder="UID" 
                    style="width:120px;" 
                    v-model="formOne.uid" 
                    auto-complete="off"></el-input>
				</el-form-item>
                <el-form-item>
					<el-input 
                    placeholder="礼物ID" 
                    style="width:120px;" 
                    v-model="formOne.goodsuid" 
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
            style="width:100%;" :height="tableHeight">
				<el-table-column prop="uid" label="用户ID"></el-table-column>
				<el-table-column label="发起砍价道具名称">
                    <template slot-scope="scope">{{ scope.row.goods_name || '--'}}</template>
                </el-table-column>
				<el-table-column prop="goods_id" label="发起砍价道具名称ID"></el-table-column>
				<el-table-column label="已砍金额">
                     <template slot-scope="scope">{{ scope.row.discount || '--'}}</template>
                </el-table-column>
                <el-table-column label="剩余金额">
                    <template slot-scope="scope">{{ scope.row.price || '--'}}</template>
                </el-table-column>
                <el-table-column prop="status" label="是否砍价成功"></el-table-column>
				<el-table-column label="发起时间">
                    <template slot-scope="scope">{{ scope.row.date || '--'}}</template>
                </el-table-column>
				<el-table-column prop="count_old" label="帮砍老用户数"></el-table-column>
				<el-table-column prop="count_new" label="帮砍新注册数"></el-table-column>
				<el-table-column label="新用户ID">
                    <template slot-scope="scope">{{ scope.row.new_assist_list || '--'}}</template>
                </el-table-column>
			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
                layout="total,prev,pager,next,jumper" 
                @current-change="oneHandleCurrentChange" 
                :page-size="20" :total="formOne.totalPage" 
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
                type: "0",
                uid:null,
                goodsuid:null,
                tabData: [],
                totalPage: null,
                page: 1,
                star: '0',
                end: '20',
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
            var url = '/NewActivity/getBargainOrderData';
			var params = {
                type: _this.formOne.type,
                goods_id: _this.formOne.goodsuid,
                uid: _this.formOne.uid,
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
                            obj.goods_name = res.data.data[i].goods_name;
                            obj.goods_id = res.data.data[i].goods_id;
                            obj.discount = res.data.data[i].discount;
                            obj.price = res.data.data[i].price;
                            obj.status = res.data.data[i].status == 0?'否':'是';
                            obj.date = res.data.data[i].date;
                            obj.count_old = res.data.data[i].count_old;
                            obj.count_new = res.data.data[i].count_new;
                            obj.new_assist_list = res.data.data[i].new_assist_list;
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
			const tHeader = ['uid','发起砍价道具名称','发起砍价道具名称ID','已砍金额','剩余金额','是否砍价成功','发起时间','帮砍老用户数','帮砍新注册数','新用户ID'];
            const filterVal = ['uid','goods_name','goods_id','discount','price','status','date','count_old','count_new','new_assist_list'];
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
