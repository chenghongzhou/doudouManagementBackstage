<template>
    <!-- 录音精彩内容 -->
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
				<el-form-item style="margin-left: 50px;">
					<span>推荐类别：</span>
					<el-select style="width:100px;" v-model="recordStatus">
						<el-option label="人工推荐" value="1"></el-option>
						<el-option label="系统推荐" value="2"></el-option>
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
			:data="listData" 
			border fit highlight-current-row 
			v-loading="listLoading" 
			element-loading-text="拼命加载中" 
			element-loading-spinner="el-icon-loading" 
			element-loading-background="rgba(0, 0, 0, 0.8)" 
			style="width:100%;" 
			:height="tableHeight">
				<el-table-column prop="recommend_time" label="推荐时间" width="180"></el-table-column>
				<el-table-column prop="id"  label="录音编码" width="100"></el-table-column>
				<el-table-column label="录音音频" width="300">
					<template slot-scope="scope">
						<div slot="reference" class="name-wrapper">
							<audio 
							controls="controls" 
							:src="scope.row.voice_url"></audio>
						</div>
					</template>
				</el-table-column>
				<el-table-column prop="duration" label="录音时长"  min-width="50" ></el-table-column>
				<el-table-column prop="uid" label="发布者ID" min-width="50" ></el-table-column>
				<el-table-column prop="content" label="录音描述"></el-table-column>
				<el-table-column prop="price" :formatter="price" label="形式" min-width="50" sortable></el-table-column>
				<el-table-column prop="listen" label="播放次数" width="50"></el-table-column>
				<el-table-column prop="praise" label="点赞数" width="50"></el-table-column>
				<el-table-column prop="relay" label="转播数" width="50"></el-table-column>
				<el-table-column prop="award" label="打赏数" width="50"></el-table-column>
				<el-table-column prop="share" label="分享数" width="50"></el-table-column>
				<el-table-column prop="complain" label="举报数" width="50"></el-table-column>
				<el-table-column prop="recommend" label="推荐次数" width="50"></el-table-column>
				<el-table-column label="操作" min-width="120">
					<template slot-scope="scope">
                        <el-col :span="12">
							<el-button 
							size="small" 
							type="primary" 
							@click="recommendAgent(scope.$index, scope.row)">在推荐</el-button>
						</el-col>
                        <el-col :span="12">
							<el-button 
							size="small" 
							type="danger" 
							@click="cancleAgent(scope.$index, scope.row)">取消推荐</el-button>
						</el-col>
					</template>
				</el-table-column>
			</el-table>
			<el-col :span="24" class="toolbar">
				<el-pagination 
				layout="total,prev,pager,next,jumper" 
				@current-change="handleCurrentChange" 
				:page-size="20" 
				:total=1000 
				style="float:right;"></el-pagination>
			</el-col>
		</template>
    </section>
</template>

<script>
	import { allget } from '../../../api/api';
	import axios from 'axios';
    export default {
        data() {
            return {
				tableHeight: null, 
                formOne: {
                    startDate: [new Date()-100*24*60*60*1000, new Date()], 
				},
				value6: '',
				value10: 'text',
				recordStatus: '',
				page: 0,
				listData: [],
				totalpage:null,
				formLabelWidth: '120px',
				listLoading: false,
            }
        },
		methods: {
			handleCurrentChange(val) {
				this.page = val-1;
				this.getData();
			},
			// 获取数据
			getData() {
				var _this = this;
				_this.listLoading = true;
				let url = '/Voice/getHotVoiceList';
				let params ={
					page: this.page,
					date_s: baseConfig.changeDateTime(this.formOne.startDate[0], 0),
					date_e: baseConfig.changeDateTime(this.formOne.startDate[1], 0),
					is_up_list: this.recordStatus,
					price: this.recordStyle,
					uid: this.uid,
					id: this.audio_uid,
				}
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret){
							this.listData = res.data.data;
						}else{
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			},
			// 提示已审核
			tipInfo() {
				var _this = this;
				baseConfig.warningTipMsg(_this, '已审核了！');
			},
			// 录音形式
			price(row){
				return row.price > 0 ? "付费" : "免费";
			},
            // 再推荐
			recommendAgent(index, row) {
				var _this = this;
				var id = row.id;
				var url = '/Voice/recommendVoiceToHot';
				var params = {
					id: id,
				};
				_this.listLoading = true;				
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							baseConfig.successTipMsg(_this, '推荐成功');						
							_this.getData();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
						baseConfig.warningTipMsg(_this, 'error');
					});
            },
			// 取消推荐
			cancleAgent(index, row) {
				var _this = this;
				var id = row.id;
				var url = '/Voice/cancelRecommendVoiceToHot';
				var params = {
					id: id,
				};
				_this.listLoading = true;				
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							baseConfig.successTipMsg(_this, '已取消');						
							_this.getData();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
						baseConfig.warningTipMsg(_this, 'error');
					});
			},
		},
		mounted() {
			var _this = this;
			_this.tableHeight = baseConfig.lineNumber(searchPageHeight);
			_this.getData();
		},
		updated() {
			var _this = this;
			//同一时间段，只能播放一条音频
			var audio = document.getElementsByTagName("audio");
			for(var i=0; i<audio.length; i++){
				audio[i].addEventListener("play", function() {
					for(var j=0; j<audio.length; j++) {
						if(audio[j]!=this) { //这里的this指向的是监听音频对象
							audio[j].pause();
						}
					}
				});
			}
		}
    }

</script>

<style lang="css" scoped>

</style>
