<template>
	<!-- banner管理->table为指的是数据table展示页面 -->
	<section>
		<el-tabs 
		v-model="tabActiveName" 
		type="border-card" @tab-click="handleClick">
			<!-- Banner条记录查询 -->
			<el-tab-pane 
			label="Banner条记录查询" 
			name="first" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formOne">
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
							<span>位置</span>
							<el-select v-model="formOne.position">
								<el-option label="全部" value=""></el-option>
								<el-option label="首页广告弹窗（安卓）" value="0"></el-option>
								<el-option label="首页广告弹窗（ios）" value="1"></el-option>
								<el-option label="充值页（安卓）" value="2"></el-option>
								<el-option label="充值页（ios）" value="3"></el-option>
								<el-option label="设置收费页（安卓）" value="4"></el-option>
								<el-option label="设置收费页（ios）" value="5"></el-option>
								<el-option label="安卓消息界面广告" value="9"></el-option>
								<el-option label="IOS消息界面广告" value="10"></el-option>
								<el-option label="安卓首页界面广告" value="11"></el-option>
								<el-option label="IOS首页界面广告" value="12"></el-option>
							</el-select>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="getTableFind">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="onePageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="add_time" label="注册时间" width="200"></el-table-column>
						<el-table-column label="banner条位置" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.position==0">安卓首页广告</p>
									<p v-else-if="scope.row.position==1">IOS首页广告</p>
									<p v-else-if="scope.row.position==2">安卓钱包界面广告</p>									
									<p v-else-if="scope.row.position==3">IOS钱包界面广告</p>									
									<p v-else-if="scope.row.position==4">安卓设置通话价格界面广告</p>									
									<p v-else-if="scope.row.position==5">IOS设置通话价格界面广告</p>									
									<p v-else-if="scope.row.position==9">安卓消息界面广告</p>									
									<p v-else-if="scope.row.position==10">IOS消息界面广告</p>									
									<p v-else-if="scope.row.position==11">安卓首页界面广告</p>									
									<p v-else-if="scope.row.position==12">IOS首页界面广告</p>									
									<p v-else-if="scope.row.position==13">安卓首页顶部界面广告</p>									
									<p v-else-if="scope.row.position==14">IOS首页顶部界面广告</p>									
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="title" label="标题" width="100"></el-table-column>
						<el-table-column label="banner条图片" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img 
									:src="scope.row.image_url" 
									style="width:100px;height:auto;">
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="jump_url" label="跳转地址" min-width="300"></el-table-column>
						<el-table-column prop="page_param" label="跳转参数" width="80"></el-table-column>
						<el-table-column label="跳转类型" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.type==1">应用</p>
									<p v-else-if="scope.row.type==2">H5页面</p>
									<p v-else-if="scope.row.type==6">偷听界面</p>
									<p v-else-if="scope.row.type==0">不跳转</p>
									<p v-else-if="scope.row.type==12">跳转到游戏</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="展示频率" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.show_type==0">每次都显示</p>
									<p v-else-if="scope.row.show_type==1">一天显示一次</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="展示状态" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.is_show==0">不展示</p>
									<p v-else-if="scope.row.is_show==1">展示</p>
								</div>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="oneHandleCurrentChange" 
						:page-size="20" 
						:total="formOne.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- Banner条记录管理 -->
			<el-tab-pane 
			label="Banner条记录管理" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item>
							<el-button 
							type="primary" 
							@click="bannerNewloading.dialogShow=true;">新增Banner条</el-button>							
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="twoPageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column label="Banner条位置" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.position==0">安卓首页广告</p>
									<p v-else-if="scope.row.position==1">IOS首页广告</p>
									<p v-else-if="scope.row.position==2">安卓钱包界面广告</p>									
									<p v-else-if="scope.row.position==3">IOS钱包界面广告</p>									
									<p v-else-if="scope.row.position==4">安卓设置通话价格界面广告</p>									
									<p v-else-if="scope.row.position==5">IOS设置通话价格界面广告</p>									
									<p v-else-if="scope.row.position==9">安卓消息界面广告</p>									
									<p v-else-if="scope.row.position==10">IOS消息界面广告</p>									
									<p v-else-if="scope.row.position==11">安卓首页界面广告</p>									
									<p v-else-if="scope.row.position==12">IOS首页界面广告</p>									
									<p v-else-if="scope.row.position==13">安卓首页顶部界面广告</p>									
									<p v-else-if="scope.row.position==14">IOS首页顶部界面广告</p>									
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="title" label="标题" width="80"></el-table-column>
						<el-table-column prop="sort" label="排序" width="50"></el-table-column>
						<el-table-column label="Banner条图片" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img 
									:src="scope.row.image_url" 
									style="width:100px;height:auto;">
								</div>
							</template>
						</el-table-column>
						<el-table-column label="新版本Banner条图片" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img 
									:src="scope.row.new_image_url" 
									style="width:100px;height:auto;">
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="jump_url" label="跳转地址" min-width="150"></el-table-column>
						<el-table-column prop="page_param" label="跳转参数" width="50"></el-table-column>
						<el-table-column label="跳转类型" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.type==1">应用</p>
									<p v-else-if="scope.row.type==2">H5页面</p>
									<p v-else-if="scope.row.type==6">偷听界面</p>
									<p v-else-if="scope.row.type==0">不跳转</p>
									<p v-else-if="scope.row.type==12">跳转到游戏</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="展示频率" width="50">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.show_type==0">每次都显示</p>
									<p v-else-if="scope.row.show_type==1">一天显示一次</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="展示状态" width="50">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.is_show==0">不展示</p>
									<p v-else-if="scope.row.is_show==1">展示</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="show_s_time" label="显示开始时间" width="80"></el-table-column>
						<el-table-column prop="show_e_time" label="显示结束时间" width="80"></el-table-column>
						<el-table-column prop="req_uid" label="拨打电话Uid" width="50"></el-table-column>
						<el-table-column prop="res_uid" label="接听电话Uid" width="50"></el-table-column>
						<el-table-column prop="start_time" label="直播通话开始时间" width="80"></el-table-column>
						<el-table-column prop="end_time" label="直播通话结束时间" width="80"></el-table-column>
						<el-table-column label="游戏类型" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.game_type==0">非游戏</p>
									<p v-else-if="scope.row.game_type==1">猫猫</p>
									<p v-else-if="scope.row.game_type==6">牛牛</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column label="操作" width="150">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								@click.native.prevent="changeOneUserData(scope.$index, scope.row)" 
								size="small">编辑</el-button>								
								<el-button 
								type="primary" 
								@click.native.prevent="deleteOneUserData(scope.$index, scope.row)" 
								size="small">删除</el-button>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="twoHandleCurrentChange" 
						:page-size="20" 
						:total="formTwo.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			<!-- banner条新增 -->
			<el-dialog title="banner条新增" :visible.sync="bannerNewloading.dialogShow">
				<el-form :model="bannerNewloading.params">
					<el-form-item label="banner位置" :label-width="formLabelWidth">
						<el-select v-model="bannerNewloading.params.position">
							<el-option label="安卓首页广告" value="0"></el-option>
							<el-option label="IOS首页广告" value="1"></el-option>
							<el-option label="安卓钱包界面广告" value="2"></el-option>
							<el-option label="IOS钱包界面广告" value="3"></el-option>
							<el-option label="安卓设置通话价格界面广告" value="4"></el-option>
							<el-option label="IOS设置通话价格界面广告" value="5"></el-option>
							<el-option label="安卓消息界面广告" value="9"></el-option>
							<el-option label="IOS消息界面广告" value="10"></el-option>
							<!-- <el-option label="安卓首页界面广告" value="11"></el-option>
							<el-option label="IOS首页界面广告" value="12"></el-option> -->
							<el-option label="安卓首页顶部界面广告" value="13"></el-option>
							<el-option label="IOS首页顶部界面广告" value="14"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="banner排序" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.sort" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="banner标题" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.title" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filepic fileinput" 
						@change="uploading($event, 0)" 
						type="file">
						<span class="showname">{{bannerNewloading.pic_name}}</span>
				        <img 
						:src="bannerNewloading.src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="跳转类型" :label-width="formLabelWidth">
						<el-select v-model="bannerNewloading.params.type">
							<el-option label="不跳转" value="0"></el-option>
							<el-option label="H5跳转" value="2"></el-option>
							<el-option label="应用内" value="1"></el-option>
							<el-option label="应用外" value="3"></el-option>
							<el-option label="偷听界面" value="6"></el-option>
							<el-option label="跳转到游戏" value="12"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="拨打电话uid" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.req_uid" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="接听电话uid" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.res_uid" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="显示开始时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.show_s_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="显示结束时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.show_e_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="直播通话开始时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.start_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="直播通话结束时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.end_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="跳转链接" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.jump_url" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="跳转参数" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.page_param" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="展示频率" :label-width="formLabelWidth">
						<el-select v-model="bannerNewloading.params.show_type">
							<el-option label="每次都显示" value="0"></el-option>
							<el-option label="一天显示一次" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="上线设置" :label-width="formLabelWidth">
						<el-select v-model="bannerNewloading.params.is_show">
							<el-option label="不展示" value="0"></el-option>
							<el-option label="展示" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="游戏类型" :label-width="formLabelWidth">
						<el-select v-model="bannerNewloading.params.game_type">
							<el-option label="非游戏" value="0"></el-option>
							<el-option label="猫猫" value="1"></el-option>
							<el-option label="牛牛" value="6"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="屏幕展示方式" :label-width="formLabelWidth">
						<el-select v-model="bannerNewloading.params.is_vertical">
							<el-option label="横屏" value="0"></el-option>
							<el-option label="竖屏" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="最少金币" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerNewloading.params.min_gold" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="新版本图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filepic fileinput" 
						@change="newUploading($event, 0)" 
						type="file">
						<span class="showname">{{bannerNewloading.new_pic_name}}</span>
				        <img 
						:src="bannerNewloading.new_src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="addBannerSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="addBannerSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<!-- banner条编辑修改 -->
			<el-dialog title="banner条编辑" :visible.sync="bannerEditorloading.dialogShow">
				<el-form :model="bannerEditorloading.params">
					<el-form-item label="banner位置" :label-width="formLabelWidth">
						<el-select v-model="bannerEditorloading.params.position" disabled>
							<el-option label="安卓首页广告" value="0"></el-option>
							<el-option label="IOS首页广告" value="1"></el-option>
							<el-option label="安卓钱包界面广告" value="2"></el-option>
							<el-option label="IOS钱包界面广告" value="3"></el-option>
							<el-option label="安卓设置通话价格界面广告" value="4"></el-option>
							<el-option label="IOS设置通话价格界面广告" value="5"></el-option>
							<el-option label="安卓消息界面广告" value="9"></el-option>
							<el-option label="IOS消息界面广告" value="10"></el-option>
							<!-- <el-option label="安卓首页界面广告" value="11"></el-option>
							<el-option label="IOS首页界面广告" value="12"></el-option> -->
							<el-option label="安卓首页顶部界面广告" value="13"></el-option>
							<el-option label="IOS首页顶部界面广告" value="14"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="banner排序" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.sort" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="banner标题" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.title" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filepic fileinput" 
						@change="uploading($event, 1)" 
						type="file">
						<span class="showname">{{bannerEditorloading.pic_name}}</span>
				        <img 
						:src="bannerEditorloading.src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="跳转类型" :label-width="formLabelWidth">
						<el-select v-model="bannerEditorloading.params.type">
							<el-option label="不跳转" value="0"></el-option>
							<el-option label="H5跳转" value="2"></el-option>
							<el-option label="应用内" value="1"></el-option>
							<el-option label="应用外" value="3"></el-option>
							<el-option label="偷听界面" value="6"></el-option>
							<el-option label="跳转到游戏" value="12"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="拨打电话uid" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.req_uid" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="接听电话uid" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.res_uid" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="显示开始时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.show_s_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="显示结束时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.show_e_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="直播通话开始时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.start_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="直播通话结束时间" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.end_time" 
						placeholder="2018-01-01 01:01:01" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="跳转链接" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.jump_url" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="跳转参数" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.page_param" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="展示频率" :label-width="formLabelWidth">
						<el-select v-model="bannerEditorloading.params.show_type">
							<el-option label="每次都显示" value="0"></el-option>
							<el-option label="一天显示一次" value="1"></el-option>
						</el-select>
					</el-form-item>

					<el-form-item label="上线设置" :label-width="formLabelWidth">
						<el-select v-model="bannerEditorloading.params.is_show">
							<el-option label="不展示" value="0"></el-option>
							<el-option label="展示" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="游戏类型" :label-width="formLabelWidth">
						<el-select v-model="bannerEditorloading.params.game_type">
							<el-option label="非游戏" value="0"></el-option>
							<el-option label="猫猫" value="1"></el-option>
							<el-option label="牛牛" value="6"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="屏幕展示方式" :label-width="formLabelWidth">
						<el-select v-model="bannerEditorloading.params.is_vertical">
							<el-option label="横屏" value="0"></el-option>
							<el-option label="竖屏" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="最少金币" :label-width="formLabelWidth">
						<el-input 
						v-model="bannerEditorloading.params.min_gold" 
						auto-complete="off"></el-input>
					</el-form-item>

					<el-form-item label="新版本图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filepic fileinput" 
						@change="newUploading($event, 1)" 
						type="file">
						<span class="showname">{{bannerEditorloading.new_pic_name}}</span>
				        <img 
						:src="bannerEditorloading.new_src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="editorBannerSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="editorBannerSure(1)">确 定</el-button>
				</div>
			</el-dialog>
		</el-tabs>
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
			tabSearchHeight: null,
			formOne: {
				choiceDate: [new Date()-180*24*60*60*1000, new Date()], 
				position: '',
				TabData: [], 
				TotalPage: null, 
				Page: 1,
				star: '0',
				end: '20', 
			},
			formTwo: {
				choiceDate: [new Date()-180*24*60*60*1000, new Date()], 
				TabData: [],
				TotalPage: null,
				Page: 1, 
				star: '0',
				end: '20', 
			},
			bannerNewloading: {
				dialogShow: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '0',
					sort: '',
					title: '',
					type: '2',
					req_uid: '',
					res_uid: '',
					start_time: '',
					end_time: '',
					jump_url: '',
					page_param: '',
					show_type: '0',
					is_show: '1',
					show_s_time: '',
					show_e_time: '',
					game_type: '',
					is_vertical: '',
					min_gold: '',
				},
			},
			bannerEditorloading: {
				dialogShow: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '',
					id: '',
					sort: '',
					title: '',
					type: '',
					req_uid: '',
					res_uid: '',
					start_time: '',
					end_time: '',
					jump_url: '',
					page_param: '',
					show_type: '',
					is_show: '',
					show_s_time: '',
					show_e_time: '',
					game_type: '',
					is_vertical: '',
					min_gold: '',
				},
			},
			listLoading: false, 
			tabActiveName: 'second', // 设置为tab切换栏的选中不同的状态(first、second)
			formLabelWidth: '130px', 
		};
	},
	computed:{
		onePageData() {
			var _this = this;
			return _this.formOne.TabData.slice(_this.formOne.star, _this.formOne.end);
		},
		twoPageData() {
			var _this = this;
			return _this.formTwo.TabData.slice(_this.formTwo.star, _this.formTwo.end);
		},
	},
	methods: {
		oneHandleCurrentChange(val) {
			var _this = this;
			_this.formOne.Page = val-1;
			_this.formOne.star = (_this.formOne.Page)*20;
			_this.formOne.end = _this.formOne.star+20;
		},
		twoHandleCurrentChange(val) {
			var _this = this;
			_this.formTwo.Page = val-1;
			_this.formTwo.star = (_this.formTwo.Page)*20;
			_this.formTwo.end = _this.formTwo.star+20;
		},
		searchConditionFind() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.position = _this.formOne.position;
			for(var key in obj) { 
				if(key=='position') {
				} else {
					if(obj[key]=='') {
						baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
						return null;
					}
				}
			}
			return obj; 
		},
		// 获取banner查询的数据
		getTableFind() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Banner/getBanner';
			var params = _this.searchConditionFind();
			if(params==null) { 
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, {params: params})
					.then((res) => {
						_this.listLoading = false;
						if(res.data.ret) {
							_this.formOne.TotalPage = res.data.data.length; 
							_this.formOne.TabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg); 
						}
					})
					.catch((err) => {
						console.log(err);
						baseConfig.warningTipMsg(_this, 'error');
					});
			}
		},
		// 获取banner管理的数据
		getTableManage() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Banner/getBanner';
			var params = {};
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						_this.formTwo.TotalPage = res.data.data.length; 
						_this.formTwo.TabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
					baseConfig.warningTipMsg(_this, 'error');
				});
		},
		// 得到上传文件type(0->新增banner，1->编辑banner)
        uploading(event, type) {
			var _this = this;
			if(type==0) {
				_this.bannerNewloading.file_pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.bannerNewloading.src_pic = windowURL.createObjectURL(event.target.files[0]);
				_this.bannerNewloading.pic_name = event.target.files[0].name;
			} else if(type==1) {
				_this.bannerEditorloading.file_pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.bannerEditorloading.src_pic = windowURL.createObjectURL(event.target.files[0]);
				_this.bannerEditorloading.pic_name = event.target.files[0].name;
			}
      	}, 
    // 得到上传文件type(0->新增banner，1->编辑banner) 新版图片地址
        newUploading(event, type) {
			var _this = this;
			if(type==0) {
				_this.bannerNewloading.new_file_pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.bannerNewloading.new_src_pic = windowURL.createObjectURL(event.target.files[0]);
				_this.bannerNewloading.new_pic_name = event.target.files[0].name;
			} else if(type==1) {
				_this.bannerEditorloading.new_file_pic = event.target.files[0];
		  	    var windowURL = window.URL || window.webkitURL;
				_this.bannerEditorloading.new_src_pic = windowURL.createObjectURL(event.target.files[0]);
				_this.bannerEditorloading.new_pic_name = event.target.files[0].name;
			}
      	},
		// banner条新增
		addBannerSure(type) {
			var _this = this;
			if(type==0) {
				_this.bannerNewloading.dialogShow = false;
				_this.resetForm();
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
				formData.append('position', _this.bannerNewloading.params.position);
				formData.append('sort', _this.bannerNewloading.params.sort);
				formData.append('title', _this.bannerNewloading.params.title);
				formData.append('type', _this.bannerNewloading.params.type);
				formData.append('req_uid', _this.bannerNewloading.params.req_uid);
				formData.append('res_uid', _this.bannerNewloading.params.res_uid);
				formData.append('start_time', _this.bannerNewloading.params.start_time);
				formData.append('end_time', _this.bannerNewloading.params.end_time);
				formData.append('jump_url', _this.bannerNewloading.params.jump_url);
				formData.append('page_param', _this.bannerNewloading.params.page_param);
				formData.append('show_type', _this.bannerNewloading.params.show_type);
				formData.append('is_show', _this.bannerNewloading.params.is_show);
			  	formData.append('pic', _this.bannerNewloading.file_pic); //提交的新增图标的文件
			  	formData.append('show_s_time', _this.bannerNewloading.params.show_s_time);
			  	formData.append('show_e_time', _this.bannerNewloading.params.show_e_time);
				formData.append('game_type', _this.bannerNewloading.params.game_type);
				formData.append('is_vertical', _this.bannerNewloading.params.is_vertical);
				formData.append('min_gold', _this.bannerNewloading.params.min_gold);
				formData.append('new_pic', _this.bannerNewloading.new_file_pic); //提交的新增游戏图标的文件
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/Banner/addBanner', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.bannerNewloading.dialogShow = false;								
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							_this.getTableFind();
							_this.getTableManage();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
						_this.resetForm();
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 对应的banner条的删除功能
		deleteOneUserData(index, rows) {
			var _this = this;
			var id = rows.id;
			_this.listLoading = true;
			var url = 'Banner/deleteBanner';
			var params = {
				id: id,
			};
			axios.get(allget+url, {params: params})
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '删除成功');
						_this.getTableFind();
						_this.getTableManage();
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 编辑进行修改
		changeOneUserData(index, rows) {
			var _this = this;
			_this.bannerEditorloading.params.position = rows.position;
			_this.bannerEditorloading.params.id = rows.id;
			_this.bannerEditorloading.params.sort = rows.sort;
			_this.bannerEditorloading.params.title = rows.title;
			_this.bannerEditorloading.params.type = rows.type;
			_this.bannerEditorloading.params.req_uid = rows.req_uid;
			_this.bannerEditorloading.params.res_uid = rows.res_uid;
			_this.bannerEditorloading.params.start_time = rows.start_time;
			_this.bannerEditorloading.params.end_time = rows.end_time;
			_this.bannerEditorloading.params.jump_url = rows.jump_url;
			_this.bannerEditorloading.params.page_param = rows.page_param;
			_this.bannerEditorloading.params.show_type = rows.show_type;
			_this.bannerEditorloading.params.is_show = rows.is_show;
			_this.bannerEditorloading.src_pic = rows.image_url;
			_this.bannerEditorloading.new_src_pic = rows.new_image_url;
			_this.bannerEditorloading.params.game_type = rows.game_type;
			_this.bannerEditorloading.params.is_vertical = rows.is_vertical;
			_this.bannerEditorloading.params.min_gold = rows.min_gold;
			if(rows.show_s_time==null||rows.show_s_time=='null') {//为null时特殊处理
				_this.bannerEditorloading.params.show_s_time = '';
			} else {
				_this.bannerEditorloading.params.show_s_time = rows.show_s_time;
			}
			if(rows.show_e_time==null||rows.show_e_time=='null') {//为null时特殊处理
				_this.bannerEditorloading.params.show_e_time = '';
			} else {
				_this.bannerEditorloading.params.show_e_time = rows.show_e_time;
			}
			_this.bannerEditorloading.dialogShow=true;
		},
		// 确定进行修改的
		editorBannerSure(type) {
			var _this = this;
			if(type==0) {
				_this.bannerEditorloading.dialogShow = false;
				_this.resetForm();
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
				formData.append('position', _this.bannerEditorloading.params.position);
				formData.append('id', _this.bannerEditorloading.params.id);
				formData.append('sort', _this.bannerEditorloading.params.sort);
				formData.append('title', _this.bannerEditorloading.params.title);
				formData.append('type', _this.bannerEditorloading.params.type);
				formData.append('req_uid', _this.bannerEditorloading.params.req_uid);
				formData.append('res_uid', _this.bannerEditorloading.params.res_uid);
				formData.append('start_time', _this.bannerEditorloading.params.start_time);
				formData.append('end_time', _this.bannerEditorloading.params.end_time);
				formData.append('jump_url', _this.bannerEditorloading.params.jump_url);
				formData.append('page_param', _this.bannerEditorloading.params.page_param);
				formData.append('show_type', _this.bannerEditorloading.params.show_type);
				formData.append('is_show', _this.bannerEditorloading.params.is_show);
			  	formData.append('pic', _this.bannerEditorloading.file_pic); //提交的新增图标的文件
			  	formData.append('show_s_time', _this.bannerEditorloading.params.show_s_time);
			  	formData.append('show_e_time', _this.bannerEditorloading.params.show_e_time);
				formData.append('game_type', _this.bannerEditorloading.params.game_type);
			  	formData.append('new_pic', _this.bannerEditorloading.new_file_pic); //提交的新增游戏图标的文件
			  	formData.append('is_vertical', _this.bannerEditorloading.params.is_vertical);
			  	formData.append('min_gold', _this.bannerEditorloading.params.min_gold);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};	
				axios.post(allget+'/Banner/editBanner', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.bannerEditorloading.dialogShow = false;								
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '编辑修改成功！');
							_this.getTableFind();
							_this.getTableManage();
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);						
						}
						_this.resetForm();
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 顶部tab进行页面的切换
		handleClick(tab, event) {
			var _this = this;
			// console.log(tab.label);
		},
		resetForm() {
			// 进行对应的重置
			var _this = this;
			_this.bannerNewloading = {
				dialogShow: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '0',
					sort: '',
					title: '',
					type: '2',
					req_uid: '',
					res_uid: '',
					start_time: '',
					end_time: '',
					jump_url: '',
					page_param: '',
					show_type: '0',
					is_show: '1',
					show_s_time: '',
					show_e_time: '',
					game_type: '',
					is_vertical: '',
					min_gold: '',
				},
			};
			_this.bannerEditorloading = {
				dialogShow: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				new_file_pic: '',
				new_src_pic: '',
				new_pic_name: '',
				params: {
					position: '',
					id: '',
					sort: '',
					title: '',
					type: '',
					req_uid: '',
					res_uid: '',
					start_time: '',
					end_time: '',
					jump_url: '',
					page_param: '',
					show_type: '',
					is_show: '',
					show_s_time: '',
					show_e_time: '',
					game_type: '',
					is_vertical: '',
					min_gold: '',
				},
			};
		},
	},
	mounted() {
		var _this = this;
		this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(tabSearchPageHeight);
			_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight);			
			_this.getTableFind();
			_this.getTableManage();
		})
	}
};
</script>

<style lang="css" scoped>
.fileinput{
	float: left;
	margin-top: 8px;
}
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
.showbtn{
	position: absolute;
	left: 0;
	top: 0;
	width: 150px;
	height: 30px;
	line-height: 30px;
	text-align:center;
	font-size: 12px;
	color: #fff;
	background-color: #00b3ee;
	border: 2px solid #00b3ee;
}
.fileinput{
	position: absolute;
	left: 0;
	top: 0;
	width: 150px;
	height: 30px;
	opacity: 0;
	border: 2px solid red;
}
.showname{
	position: absolute;
	left: 0;
	top: 38px;
	font-size: 14px;
	width: 150px;
	height: 16px;
	line-height: 16px;
	overflow:hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
</style>