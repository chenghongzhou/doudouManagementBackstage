<template>
	<!-- 礼物管理->table为指的是数据table展示页面 -->
	<section>
		<el-tabs 
		v-model="tabActiveName" type="border-card" @tab-click="handleClick">
			<el-tab-pane 
			label="黄金宝箱" 
			name="first" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formOne">
						<el-form-item>
							<div class="block">
								<span class="registerTime">注册时间</span>
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
							@click="giftUploading.dialogShow=true;">新增礼物</el-button>
							<el-button 
							type="primary" 
							@click="getTableNormalGiftData">查询</el-button>
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
						<el-table-column prop="id" label="礼物ID" width="50"></el-table-column>
						<el-table-column prop="name" label="礼物名称" width="100"></el-table-column>
						<el-table-column label="礼物图标" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img 
									:src="scope.row.icon" 
									style="width:100px;height:auto;">
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="price" label="礼物价格" width="50"></el-table-column>
						<el-table-column prop="on_sale_time" label="上架时间" width="80"></el-table-column>
						<el-table-column prop="create_time" label="创建时间" width="80"></el-table-column>
						<el-table-column label="上架状态" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.status==1">上架了</p>
									<p v-else-if="scope.row.status==0">已下架</p>
									<p v-else-if="scope.row.status==2">等待上架</p>									
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="probability" label="概率" width="80"></el-table-column>
						<el-table-column prop="week_hot" label="爆款时间(0代表周日)">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									{{scope.row.week_hot}}
									<!-- {{handleWeek(scope.row.week_hot)}} -->
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="sort" label="排序" width="50"></el-table-column>
						<el-table-column prop="down_sale_time" label="礼物下架时间" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.down_sale_time==null">无</p>
									<p v-else>{{scope.row.down_sale_time}}</p>
								</div>
							</template>
						</el-table-column>											
						<el-table-column label="操作" fixed="right" min-width="200">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								@click.native.prevent="changeOneUserData(scope.$index, scope.row, '1')" 
								size="mini">编辑</el-button>								
								<el-button 
								type="primary" 
								@click.native.prevent="specialData(scope.$index, scope.row, '1')" 
								size="mini">特效</el-button>								
								<el-button 
								v-if="scope.row.status=='0'" 
								plain 
								size="mini" 
								@click.native.prevent="tipUndercarriage(scope.$index, scope.row, '1')">下架</el-button>
								<el-button 
								v-else 
								type="primary" 
								@click.native.prevent="undercarriage(scope.$index, scope.row, '1')" 
								size="mini">下架</el-button>
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
			<!-- 活动礼物 -->
			<el-tab-pane 
			label="铂金宝箱" 
			name="second" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formTwo">
						<el-form-item>
							<div class="block">
								<span class="registerTime">注册时间</span>
								<el-date-picker 
								v-model="formTwo.choiceDate" 
								type="daterange" 
								range-separator=" 至 " 
								placeholder="选择日期范围"></el-date-picker>
							</div>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="giftUploading.dialogShow=true;">新增礼物</el-button>							
							<el-button 
							type="primary" 
							@click="getTableActivityGiftData">查询</el-button>
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
						<el-table-column prop="id" label="礼物ID" width="50"></el-table-column>
						<el-table-column prop="name" label="礼物名称" width="100"></el-table-column>
						<el-table-column label="礼物图标" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img 
									:src="scope.row.icon" 
									style="width:100px;height:auto;">
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="price" label="礼物价格" width="50"></el-table-column>
						<el-table-column prop="on_sale_time" label="上架时间" width="80"></el-table-column>
						<el-table-column prop="create_time" label="创建时间" width="80"></el-table-column>
						<el-table-column label="上架状态" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.status==1">上架了</p>
									<p v-else-if="scope.row.status==0">已下架</p>
									<p v-else-if="scope.row.status==2">等待上架</p>									
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="probability" label="概率" width="80"></el-table-column>
						<el-table-column prop="week_hot" label="爆款时间(0代表周日)">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									{{scope.row.week_hot}}
									<!-- {{handleWeek(scope.row.week_hot)}} -->
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="sort" label="排序" width="50"></el-table-column>
						<el-table-column prop="down_sale_time" label="礼物下架时间" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.down_sale_time==null">无</p>
									<p v-else>{{scope.row.down_sale_time}}</p>
								</div>
							</template>
						</el-table-column>											
						<el-table-column label="操作" fixed="right" min-width="200">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								@click.native.prevent="changeOneUserData(scope.$index, scope.row, '2')" 
								size="mini">编辑</el-button>								
								<el-button 
								type="primary" 
								@click.native.prevent="specialData(scope.$index, scope.row, '2')" 
								size="mini">特效</el-button>								
								<el-button 
								v-if="scope.row.status=='0'" 
								plain 
								size="mini" 
								@click.native.prevent="tipUndercarriage(scope.$index, scope.row, '2')">下架</el-button>
								<el-button 
								v-else 
								type="primary" 
								@click.native.prevent="undercarriage(scope.$index, scope.row, '2')" 
								size="mini">下架</el-button>
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
			<!-- 钻石宝箱 -->
			<el-tab-pane 
			label="钻石宝箱" 
			name="three" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form 
					:inline="true" 
					style="overflow:hidden;" 
					:model="formThree">
						<el-form-item>
							<div class="block">
								<span class="registerTime">注册时间</span>
								<el-date-picker 
								v-model="formThree.choiceDate" 
								type="daterange" 
								range-separator=" 至 " 
								placeholder="选择日期范围"></el-date-picker>
							</div>
						</el-form-item>
						<el-form-item>
							<el-button 
							type="primary" 
							@click="giftUploading.dialogShow=true;">新增礼物</el-button>							
							<el-button 
							type="primary" 
							@click="getTableRoomGiftData">查询</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="threePageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="id" label="礼物ID" width="50"></el-table-column>
						<el-table-column prop="name" label="礼物名称" width="100"></el-table-column>
						<el-table-column label="礼物图标" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<img 
									:src="scope.row.icon" 
									style="width:100px;height:auto;">
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="price" label="礼物价格" width="50"></el-table-column>
						<el-table-column prop="on_sale_time" label="上架时间" width="80"></el-table-column>
						<el-table-column prop="create_time" label="创建时间" width="80"></el-table-column>
						<el-table-column label="上架状态" width="100">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.status==1">上架了</p>
									<p v-else-if="scope.row.status==0">已下架</p>
									<p v-else-if="scope.row.status==2">等待上架</p>									
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="probability" label="概率" width="80"></el-table-column>
						<el-table-column prop="week_hot" label="爆款时间(0代表周日)">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									{{scope.row.week_hot}}
									<!-- {{handleWeek(scope.row.week_hot)}} -->
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="sort" label="排序" width="50"></el-table-column>
						<el-table-column prop="down_sale_time" label="礼物下架时间" width="80">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.down_sale_time==null">无</p>
									<p v-else>{{scope.row.down_sale_time}}</p>
								</div>
							</template>
						</el-table-column>											
						<el-table-column label="操作" fixed="right" min-width="200">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								@click.native.prevent="changeOneUserData(scope.$index, scope.row, '3')" 
								size="mini">编辑</el-button>								
								<el-button 
								type="primary" 
								@click.native.prevent="specialData(scope.$index, scope.row, '3')" 
								size="mini">特效</el-button>								
								<el-button 
								v-if="scope.row.status=='0'" 
								plain 
								size="mini" 
								@click.native.prevent="tipUndercarriage(scope.$index, scope.row, '3')">下架</el-button>
								<el-button 
								v-else 
								type="primary" 
								@click.native.prevent="undercarriage(scope.$index, scope.row, '3')" 
								size="mini">下架</el-button>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="threeHandleCurrentChange" 
						:page-size="20" 
						:total="formThree.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
				</template>
			</el-tab-pane>
			
			
			<!-- 标签管理 -->
			<!-- <el-tab-pane 
			label="标签管理" 
			name="four" 
			:style="{height:tabSearchHeight+'px'}">
				<el-col :span="24" class="toolbar" style="padding-bottom:0px;">
					<el-form :inline="true" style="overflow:hidden;" :model="formFour">
						<el-form-item>
							<el-button 
							type="primary" 
							@click="formFour.addDialogShow=true;">新增标签</el-button>
						</el-form-item>
					</el-form>
				</el-col>
				<template>
					<el-table 
					ref="tableHeight" 
					:data="fourPageData" 
					border fit highlight-current-row 
					v-loading="listLoading" 
					style="width:100%;" 
					:height="tableHeight">
						<el-table-column prop="id" label="ID" width="150"></el-table-column>
						<el-table-column prop="tab_name" label="礼物标签名" width="150"></el-table-column>
						<el-table-column prop="position" label="位置(值)" width="150"></el-table-column>
						<el-table-column prop="create_time" label="创建时间" width="200"></el-table-column>
						<el-table-column label="状态" width="150">
							<template slot-scope="scope">
								<div slot="reference" class="name-wrapper">
									<p v-if="scope.row.status==0">下架</p>
									<p v-else-if="scope.row.status==1">上架</p>
								</div>
							</template>
						</el-table-column>
						<el-table-column prop="sort" label="排序" width="150"></el-table-column>
						<el-table-column label="操作" min-width="100">
							<template slot-scope="scope">
								<el-button 
								type="primary" 
								@click.native.prevent="changeLabelUserData(scope.$index, scope.row)" 
								size="mini">编辑</el-button>								
								<el-button 
								type="primary" 
								@click.native.prevent="deleteLabel(scope.$index, scope.row)" 
								size="mini">删除</el-button>
							</template>
						</el-table-column>
					</el-table>
					<el-col :span="24" class="toolbar">
						<el-pagination 
						layout="total,prev,pager,next,jumper" 
						@current-change="fourHandleCurrentChange" 
						:page-size="20" 
						:total="formFour.TotalPage" 
						style="float:right;"></el-pagination>
					</el-col>
					<el-dialog 
					title="修改礼物标签" 
					:visible.sync="formFour.editorDialogShow" 
					width="80%">
						<el-form :model="formFour.editor">
							<el-form-item label="标签ID" :label-width="formLabelWidth">
								<el-input 
								v-model="formFour.editor.id" 
								auto-complete="off" 
								disabled></el-input>
							</el-form-item>
							<el-form-item label="标签名称" :label-width="formLabelWidth">
								<el-input 
								v-model="formFour.editor.tab_name" 
								auto-complete="off"></el-input>
							</el-form-item>
							<el-form-item label="标签排序" :label-width="formLabelWidth">
								<el-input 
								v-model="formFour.editor.sort" 
								auto-complete="off"></el-input>
							</el-form-item>
						</el-form>
						<div slot="footer" class="dialog-footer">
							<el-button 
							@click.native.prevent="editorLabelSure(0)">取 消</el-button>
							<el-button 
							type="primary" 
							@click.native.prevent="editorLabelSure(1)">确 定</el-button>
						</div>
					</el-dialog> -->
					<!-- 新增修改--对应的dialog -->
					<!-- <el-dialog 
					title="新增礼物标签" 
					:visible.sync="formFour.addDialogShow">
						<el-form :model="formFour.add">
							<el-form-item label="标签名称" :label-width="formLabelWidth">
								<el-input 
								v-model="formFour.add.tab_name" 
								auto-complete="off"></el-input>
							</el-form-item>
							<el-form-item label="标签排序" :label-width="formLabelWidth">
								<el-input 
								v-model="formFour.add.sort" 
								auto-complete="off"></el-input>
							</el-form-item>
						</el-form>
						<div slot="footer" class="dialog-footer">
							<el-button 
							@click.native.prevent="addLabelSure(0)">取 消</el-button>
							<el-button 
							type="primary" 
							@click.native.prevent="addLabelSure(3)">确 定</el-button>
						</div>
					</el-dialog>
				</template>
			</el-tab-pane> -->
			<!-- 礼物上传 -->
			<el-dialog 
			title="礼物上传" 
			:visible.sync="giftUploading.dialogShow" 
			width="80%">
				<el-form :model="giftUploading.params">
					<el-form-item label="礼物排序" :label-width="formLabelWidth">
						<el-input 
						v-model="giftUploading.params.sort" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="礼物名称" :label-width="formLabelWidth">
						<el-input 
						v-model="giftUploading.params.name" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="礼物价格" :label-width="formLabelWidth">
						<el-input 
						v-model="giftUploading.params.price" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						id="fileinput0" 
						class="filepic fileinput" 
						@change="uploading($event, 0, 0)" 
						type="file">
						<span class="showname">{{giftUploading.pic_name}}</span>
				        <img 
						:src="giftUploading.src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="动态图上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						id="fileinput1" 
						class="filegif fileinput" 
						@change="uploading($event, 1, 0)" 
						type="file">
						<span class="showname">{{giftUploading.gif_name}}</span>
				        <img 
						:src="giftUploading.src_gif" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="角标图" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						id="fileinput2" 
						class="filegif fileinput" 
						@change="uploading($event, 2, 0)" 
						type="file">
						<span class="showname">{{giftUploading.corner_name}}</span>
				        <img 
						:src="giftUploading.src_corner" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="礼物位置" :label-width="formLabelWidth">
						<el-select v-model="giftUploading.params.position">
							<el-option label="黄金礼物" value="8"></el-option>
							<el-option label="铂金礼物" value="9"></el-option>
							<el-option label="钻石礼物" value="10"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="礼物上架状态" :label-width="formLabelWidth">
						<el-select v-model="giftUploading.params.status">
							<el-option label="上架" value="1"></el-option>
							<el-option label="下架" value="0"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item style="margin-left:40px;">
                        <span>爆款时间</span>
						<el-checkbox v-model="item.checked" @change="changeAddFun(item)" v-for="(item, index) in weekData" :key="index">
							{{ item.text }}
						</el-checkbox>
                        <!-- filterable:是否进行多选框输入 -->
                        <!-- <el-select 
                        v-model="giftUploading.params.week_hot" 
                        multiple 
                        collapse-tags 
                        style="margin-left:20px;width:240px;" 
                        placeholder="请选择">
                            <el-option 
                            v-for="(item, key) of weekData" 
                            :key="key" 
                            :label="item.text" 
                            :value="item.num"></el-option>
                        </el-select> -->
                    </el-form-item>
					<el-form-item label="是否奖池礼物" :label-width="formLabelWidth">
						<el-select v-model="giftUploading.params.is_pool">
							<el-option label="是" value="1"></el-option>
							<el-option label="否" value="0"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="概率" :label-width="formLabelWidth">
						<el-input 
						v-model="giftUploading.params.probability" 
						auto-complete="off"></el-input>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="addGiftSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="addGiftSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<!-- 礼物编辑 -->
			<el-dialog 
			title="礼物编辑" 
			:visible.sync="giftEditorloading.dialogShow" 
			width="80%">
				<el-form :model="giftEditorloading.params">
					<el-form-item label="礼物ID" :label-width="formLabelWidth">
						<el-input disabled v-model="giftEditorloading.params.id" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="礼物排序" :label-width="formLabelWidth">
						<el-input 
						v-model="giftEditorloading.params.sort" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="礼物名称" :label-width="formLabelWidth">
						<el-input 
						v-model="giftEditorloading.params.name" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="礼物价格" :label-width="formLabelWidth">
						<el-input 
						v-model="giftEditorloading.params.price" 
						auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="图片上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filepic fileinput" 
						id="fileinput3" 
						@change="uploading($event, 0, 1)" 
						type="file">
						<span class="showname">{{giftEditorloading.pic_name}}</span>
				        <img 
						:src="giftEditorloading.src_pic" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="动态图上传" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filegif fileinput" 
						id="fileinput4"
						@change="uploading($event, 1, 1)" 
						type="file">
						<span class="showname">{{giftEditorloading.gif_name}}</span>
				        <img 
						:src="giftEditorloading.src_gif" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="角标图" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filegif fileinput" 
						id="fileinput5" 
						@change="uploading($event, 2, 1)" 
						type="file">
						<span class="showname">{{giftEditorloading.corner_name}}</span>
				        <img 
						:src="giftEditorloading.src_corner" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="礼物位置" :label-width="formLabelWidth">
						<el-select v-model="giftEditorloading.params.position">
							<el-option label="黄金礼物" value="8"></el-option>
							<el-option label="铂金礼物" value="9"></el-option>
							<el-option label="钻石礼物" value="10"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="礼物上架状态" :label-width="formLabelWidth">
						<el-select v-model="giftEditorloading.params.status">
							<el-option label="上架" value="1"></el-option>
							<el-option label="下架" value="0"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item style="margin-left:40px;">
                        <span>爆款时间</span>
						<el-checkbox v-model="item.checked" @change="changeEditFun(item)" v-for="(item, index) in weekData" :key="index">
							{{ item.text }}
						</el-checkbox>
                        <!-- filterable:是否进行多选框输入 -->
                        <!-- <el-select 
                        v-model="giftEditorloading.params.week_hot" 
                        multiple 
                        collapse-tags 
                        style="margin-left:20px;width:240px;" 
                        placeholder="请选择">
                            <el-option 
                            v-for="(item, key) of weekData" 
                            :key="key" 
                            :label="item.text" 
                            :value="item.num"></el-option>
                        </el-select> -->
                    </el-form-item>
					<el-form-item label="是否奖池礼物" :label-width="formLabelWidth">
						<el-select v-model="giftEditorloading.params.is_pool">
							<el-option label="是" value="1"></el-option>
							<el-option label="否" value="0"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="概率" :label-width="formLabelWidth">
						<el-input 
						v-model="giftEditorloading.params.probability" 
						auto-complete="off"></el-input>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="editorGiftSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="editorGiftSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<!-- 礼物特效的弹窗的table表框 -->
			<el-dialog 
			title="礼物数量特效详情" 
			:visible.sync="giftNumTable.dialogShow" 
			width="80%">
				<el-table 
				:data="giftNumTable.giftNumArr">
					<el-table-column property="gift_id" label="礼物ID" width="50"></el-table-column>
					<el-table-column property="num" label="数量" width="50"></el-table-column>
					<el-table-column property="num_name" label="礼物数字描述" min-width="100"></el-table-column>
					<el-table-column property="is_dynamic_effect" label="是否有特效" width="80">
						<template slot-scope="scope">
							<div slot="reference" class="name-wrapper">
								<p v-if="scope.row.is_dynamic_effect==0">无特效</p>
								<p v-else-if="scope.row.is_dynamic_effect==1">有特效</p>
							</div>
						</template>
					</el-table-column>
					<el-table-column property="num_status" label="是否显示" width="80">
						<template slot-scope="scope">
							<div slot="reference" class="name-wrapper">
								<p v-if="scope.row.num_status==0">显示</p>
								<p v-else-if="scope.row.num_status==1">不显示</p>
							</div>
						</template>
					</el-table-column>
					<el-table-column property="num_img_url" label="标识图" width="150">
						<template slot-scope="scope">
							<div slot="reference" class="name-wrapper">
								<p v-if="scope.row.num_img_url==''||scope.row.num_img_url==null">无标识图</p>
								<img 
								v-else 
								:src="scope.row.num_img_url" 
								style="width:100px;height:auto;">
							</div>
						</template>
					</el-table-column>
					<el-table-column property="dynamic_effect_url" label="特效图" width="150">
						<template slot-scope="scope">
							<div slot="reference" class="name-wrapper">
								<p v-if="scope.row.dynamic_effect_url==''||scope.row.dynamic_effect_url==null">无特效图</p>
								<el-button 
								v-else
								type="danger" 
								@click.native.prevent="lookSvga(scope.row.dynamic_effect_url)" 
								size="mini">查看</el-button>
							</div>
						</template>
					</el-table-column>
					<el-table-column label="操作" width="100">
						<template slot-scope="scope">
							<el-button 
							type="primary" 
							@click.native.prevent="changeNumGIft(scope.$index, giftNumTable.giftNumArr)" 
							size="mini">编辑</el-button>								
						</template>
					</el-table-column>
				</el-table>
			</el-dialog>
			<!-- 礼物特效编辑 -->
			<el-dialog 
			title="特效礼物添加编辑" 
			:visible.sync="giftSpecial.dialogShow" 
			width="80%">
				<el-form :model="giftSpecial">
					<el-form-item label="id" :label-width="formLabelWidth">
						<el-input disabled v-model="giftSpecial.id" auto-complete="off"></el-input>
					</el-form-item>
					<el-form-item label="数量" :label-width="formLabelWidth">
						<el-input disabled v-model="giftSpecial.num" auto-complete="off"></el-input>						
					</el-form-item>
					<el-form-item label="数字名称" :label-width="formLabelWidth">
						<el-input disabled v-model="giftSpecial.num_name" auto-complete="off"></el-input>						
					</el-form-item>
					<el-form-item label="是否显示" :label-width="formLabelWidth">
						<el-select v-model="giftSpecial.num_status">
							<el-option label="显示" value="0"></el-option>
							<el-option label="不显示" value="1"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="是否特效" :label-width="formLabelWidth">
						<el-select v-model="giftSpecial.is_dynamic_effect">
							<el-option label="无特效" value="0"></el-option>
							<el-option label="有特效" value="1"></el-option>
						</el-select>				
					</el-form-item>
					<el-form-item label="标识图" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filegif fileinput" 
						id="fileinput6" 
						@change="uploading($event, 0, 2)" 
						type="file">
						<span class="showname">{{giftSpecial.num_file_name}}</span>
				        <img 
						:src="giftSpecial.src_num" 
						style="width:200px;height:auto;margin-left:200px;"/>
					</el-form-item>
					<el-form-item label="特效图" :label-width="formLabelWidth">
						<span class="showbtn">选择文件</span>
						<input 
						class="filegif fileinput" 
						id="fileinput7" 
						@change="uploading($event, 1, 2)" 
						type="file">
						<span class="showname">{{giftSpecial.effect_file_name}}</span>
						<p style="margin-left:200px;width:0px;opacity:0;display:inline-block;"></p>
						<el-button 
						v-if="giftSpecial.src_effect!=''" 
						@click.native.prevent="downDemo(giftSpecial.src_effect)">下载文件</el-button>
						<el-button 
						@click.native.prevent="lookDemo()">特效图页面</el-button>
						<p style="color:red;display:inline-block;">可以打开特效图页面，将文件拖入查看效果~</p>
					</el-form-item>
				</el-form>
				<div slot="footer" class="dialog-footer">
					<el-button 
					@click.native.prevent="specialGiftSure(0)">取 消</el-button>
					<el-button 
					type="primary" 
					@click.native.prevent="specialGiftSure(1)">确 定</el-button>
				</div>
			</el-dialog>
			<el-dialog
			title="特效图展示" 
			:visible.sync="svga.dialogShow" 
			width="50%">
				<el-form :model="svga">
					<div id="demoCanvas"></div>
				</el-form>
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
			svga: {
				dialogShow: false,//控制加上设置svga的显示框
				player: null,
				parser: null,
			},
			tableHeight: null, 
			tabSearchHeight: '',
			formOne: {
				choiceDate: [new Date()-180*24*60*60*1000,new Date()],
				TabData: [], 
				TotalPage: null,  
				Page: 1, 
				star: '0', 
				end: '20', 
			},
			formTwo: {
				choiceDate: [new Date()-180*24*60*60*1000,new Date()], 
				TabData: [], 
				TotalPage: null, 
				Page: 1, 
				star: '0', 
				end: '20', 
			},
			formThree: {
				choiceDate: [new Date()-180*24*60*60*1000,new Date()], 
				TabData: [], 
				TotalPage: null,  
				Page: 1, 
				star: '0', 
				end: '20', 
			},
			
			fourPageData: {
				TabData: [], 
				TotalPage: null, 
				Page: 1, 
				star: '0', 
				end: '20', 
				editorDialogShow: false,
				addDialogShow: false, 
				editor: { // 修改对应标签
					index: '',
					id: '',
					sort: '',
					tab_name: '',
				},
				add: { // 新增对应的标签
					sort: '',
					tab_name: '',
				},
			},
			weekData:[
				{num:0,text:'周日'},
				{num:1,text:'周一'},
				{num:2,text:'周二'},
				{num:3,text:'周三'},
				{num:4,text:'周四'},
				{num:5,text:'周五'},
				{num:6,text:'周六'},
			],
			giftUploading: {
				dialogShow: false,
				file_pic: '',//上传的图片
				src_pic: '',
				pic_name: '',
				file_gif: '',//动态图
				src_gif: '',
				gif_name: '',
				file_corner: '',//角标图
				src_corner: '',
				corner_name: '',
				params: {
					sort: '',
					name: '',
					price: '',
					pic: '',
					gif: '',
					position: '8',
					status: '1',
					type: '1',
					desc: '',
					on_sale_time: '',
					is_vip: '0',
					is_turntable: '0',
					probability: '',
					is_special: '0',
					stock_num: '',
					is_pool:'',
					week_hot:''
				},
			},
			giftEditorloading: {
				index: '',
				type: '',
				dialogShow: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				file_gif: '',
				src_gif: '',
				gif_name: '',
				file_corner: '',//角标图
				src_corner: '',
				corner_name: '',
				params: {
					id: '',
					sort: '',
					name: '',
					price: '',
					pic: '',
					gif: '',
					position: '',
					status: '',
					type: '',
					create_time: '',
					on_sale_time: '',
					is_vip: '',
					is_turntable: '',
					probability: '',
					is_special: '',
					stock_num: '',
					is_pool:'',
					week_hot:''
				},
			},
			giftNumTable: {
				dialogShow: false,
				giftNumArr: [],
			},
			giftSpecial: {
				dialogShow: false,
				id: '',
				num: '',
				is_dynamic_effect: '',
				num_url: '',
				src_num: '',
				num_file_name: '',
				effect_url: '',
				src_effect: '',
				effect_file_name: '',
				num_name: '',
				num_status: '',
			},
			listLoading: false, 
			tabActiveName: 'first', // 设置为tab切换栏的选中不同的状态(first、second、three、four)
			formLabelWidth: '120px', 
			week:null
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
		threePageData() {
			var _this = this;
			return _this.formThree.TabData.slice(_this.formThree.star, _this.formThree.end);
		},
		sixPageData() {
			var _this = this;
			return _this.formSix.TabData.slice(_this.formSix.star, _this.formSix.end);
		},
		handleWeek(){
			return function(data){
				if(data){
					console.log(data)
					// var s = JSON.parse(data).join(',');
					// return s;
				}
			}
		}
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
		threeHandleCurrentChange(val) {
			var _this = this;
			_this.formThree.Page = val-1;
			_this.formThree.star = (_this.formThree.Page)*20;
			_this.formThree.end = _this.formThree.star+20;
		},
		
		sixHandleCurrentChange(val) {
			var _this = this;
			_this.formSix.Page = val-1;
			_this.formSix.star = (_this.formSix.Page)*20;
			_this.formSix.end = _this.formSix.star+20;
		},
		// 提示早已经下架了
		tipUndercarriage() {
			var _this = this;
			baseConfig.warningTipMsg(_this, '早就已经下架了！');
		},
		// 搜索条件
		searchConditionNormalGift() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0);
			obj.position = '8';
			for(var key in obj) { 
				if(obj[key]=='') {
					baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
					return null;
				}
			}
			return obj; 
		},
		searchConditionActiveGift() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formTwo.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formTwo.choiceDate[1], 0);
			obj.position = '9';
			for(var key in obj) { 
				if(obj[key]=='') {
					baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
					return null;
				}
			}
			return obj; 
		},
		searchConditionRoomGift() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formThree.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formThree.choiceDate[1], 0);
			obj.position = '10';
			for(var key in obj) { 
				if(obj[key]=='') {
					baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
					return null;
				}
			}
			return obj; 
		},
		searchBagActiveTurntable() {
			var _this = this;
			var obj = {};
			obj.date_s = baseConfig.changeDateTime(_this.formSix.choiceDate[0], 0);
			obj.date_e = baseConfig.changeDateTime(_this.formSix.choiceDate[1], 0);
			obj.position = '7';
			for(var key in obj) { 
				if(obj[key]=='') {
					baseConfig.warningTipMsg(_this, '搜索条件值不能为空！');
					return null;
				}
			}
			return obj; 
		},
		// 获取黄金宝箱礼物数据列表
		getTableNormalGiftData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Gift/getGift';
			// var params = _this.searchConditionNormalGift();
			var params = {
				date_s: baseConfig.changeDateTime(_this.formOne.choiceDate[0], 0),
				date_e: baseConfig.changeDateTime(_this.formOne.choiceDate[1], 0),
				position: '8',
			};
			if(params==null) { 
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, { params: params })				
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
					});
			}
		},
		getTableActivityGiftData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Gift/getGift';
			var params = _this.searchConditionActiveGift();
			if(params==null) { 
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, { params: params })				
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
					});
			}
		},
		
		getTableRoomGiftData() {
			var _this = this ;
			_this.listLoading = true;
			var url = '/Gift/getGift';
			// var params = _this.searchConditionRoomGift();
			var params = {
				date_s: baseConfig.changeDateTime(_this.formThree.choiceDate[0], 0),
				date_e: baseConfig.changeDateTime(_this.formThree.choiceDate[1], 0),
				position: '10',
			};
			if(params==null) { 
				_this.listLoading = false; 
			} else {
				axios.get(allget+url, { params: params })
					.then((res) => { 
						_this.listLoading = false;
						if(res.data.ret) {
							_this.formThree.TotalPage = res.data.data.length; 
							_this.formThree.TabData = res.data.data;
						} else {
							baseConfig.warningTipMsg(_this, res.data.msg);
						}
					})
					.catch((err) => {
						console.log(err);
					});
			}
		},
		// 下架的操作
		undercarriage(index, rows, type) {
			var _this = this;
			if(type=='1') {
				index = index + (_this.formOne.Page-1)*20; 
			} else if(type=='2') {
				index = index + (_this.formTwo.Page-1)*20;
			} else if(type=='3') {
				index = index + (_this.formThree.Page-1)*20; 
			} else if(type=='6') {
				index = index + (_this.formFive.Page-1)*20; 
			} else if(type=='7') {
				index = index + (_this.formSix.Page-1)*20; 
			}
			var id = rows.id; // 拿出对应内容的中value值	
			var url = 'Gift/downGift';
			var params = {
				id: id,
			};
			axios.get(allget+url, { params: params })
				.then((res) => { 
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '下架成功！');
						if(type=='1') {
							_this.getTableNormalGiftData();
						} else if(type=='2') {
							_this.getTableActivityGiftData();
						} else if(type=='3') {
							_this.getTableRoomGiftData();
						}
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 礼物的编辑操作
		changeOneUserData(index, rows, type) {
			var _this = this;
			if(type=='1') {
				index = index + (_this.formOne.Page-1)*20; 
			} else if(type=='2') {
				index = index + (_this.formTwo.Page-1)*20; 
			} else if(type=='3') {
				index = index + (_this.formThree.Page-1)*20;
			} else if(type=='6') {
				index = index + (_this.formFive.Page-1)*20;
			} else if(type=='7') {
				index = index + (_this.formSix.Page-1)*20;
			}
			_this.giftEditorloading.index = index;
			_this.giftEditorloading.type = type;
			_this.giftEditorloading.params.id = rows.id;
			_this.giftEditorloading.params.sort = rows.sort;
			_this.giftEditorloading.params.name = rows.name;
			_this.giftEditorloading.params.price = rows.price;
			_this.giftEditorloading.src_pic = rows.icon;
			_this.giftEditorloading.src_gif = rows.dynamic_icon;
			_this.giftEditorloading.src_corner = rows.superscript;
			_this.giftEditorloading.params.position = rows.position;
			_this.giftEditorloading.params.status = rows.status;
			_this.giftEditorloading.params.type = rows.type;
			_this.giftEditorloading.params.create_time = rows.create_time;
			_this.giftEditorloading.params.on_sale_time = rows.on_sale_time;
			_this.giftEditorloading.params.is_vip = rows.is_vip;
			_this.giftEditorloading.params.is_turntable = rows.is_turntable;
			_this.giftEditorloading.params.probability = rows.probability;
			_this.giftEditorloading.params.is_special = rows.is_special;
			_this.giftEditorloading.params.stock_num = rows.stock_num;
			_this.giftEditorloading.params.is_pool = rows.is_pool;
			_this.giftEditorloading.params.week_hot = rows.week_hot;
			// 在这里对相应的文件角标图、礼物图片、礼物动图置空
			_this.giftEditorloading.file_pic = '';
			_this.giftEditorloading.file_gif = '';
			_this.giftEditorloading.file_corner = '';
			var arr = _this.giftEditorloading.params.week_hot.split(',');
			console.log(_this.giftEditorloading.params.week_hot.split(','))
			_this.weekData.forEach((item,index) => {
				var num = item.num;
				arr.forEach((rt) => {
					if(num == rt){
						_this.weekData[index].checked = true;
					}
				})
			})
			_this.giftEditorloading.dialogShow=true;
		},
		// 确定礼物的编辑操作
		editorGiftSure(type) {
			var _this = this;
			if(type==0) {
				_this.giftEditorloading.dialogShow = false;
				_this.resetForm();
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('id', _this.giftEditorloading.params.id);
			  	formData.append('sort', _this.giftEditorloading.params.sort);
			  	formData.append('name', _this.giftEditorloading.params.name);
			  	formData.append('price', _this.giftEditorloading.params.price);
			  	formData.append('pic', _this.giftEditorloading.file_pic);
			  	formData.append('gif', _this.giftEditorloading.file_gif);
			  	formData.append('corner', _this.giftEditorloading.file_corner);
			  	formData.append('position', _this.giftEditorloading.params.position);
			  	formData.append('status', _this.giftEditorloading.params.status);
			  	formData.append('type', _this.giftEditorloading.params.type);
			  	formData.append('create_time', _this.giftEditorloading.params.create_time);
			  	formData.append('on_sale_time', baseConfig.changeDateTime(_this.giftEditorloading.params.on_sale_time, 1));
			  	formData.append('is_vip', _this.giftEditorloading.params.is_vip);
			  	formData.append('is_turntable', _this.giftEditorloading.params.is_turntable);
			  	formData.append('probability', _this.giftEditorloading.params.probability);
			  	formData.append('is_special', _this.giftEditorloading.params.is_special);
			  	formData.append('stock_num', _this.giftEditorloading.params.stock_num);
				formData.append('week_hot', _this.giftEditorloading.params.week_hot);
			  	formData.append('is_pool', _this.giftEditorloading.params.is_pool);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/Gift/editGift', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.giftEditorloading.dialogShow = false;									
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '编辑修改成功！');
							// 对相应的table中的内容进行修改替换
							if(_this.giftEditorloading.type==1) {
								_this.getTableNormalGiftData();
							} else if(_this.giftEditorloading.type==2) {
								_this.getTableActivityGiftData();
							} else if(_this.giftEditorloading.type==3) {
								_this.getTableRoomGiftData();
							}
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
		// 获取标签管理页面的数据
		getLabelData() {
			var _this = this;
			_this.listLoading = true;
			var url = '/Gift/getGiftTab';
			var params = {};
			axios.get(allget+url, {params: params})					
				.then((res) => { 
					_this.listLoading = false;
					if(res.data.ret) {
						_this.formFour.TotalPage = res.data.data.length; 
						_this.formFour.TabData = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		// 添加标签的操作
		addLabelSure(type) {
			var _this = this;
			if(type==3) {
				baseConfig.warningTipMsg(_this, '请联系管理员进行添加！');
				_this.formFour.addDialogShow = false;
			} else if(type==0) {
				_this.formFour.addDialogShow = false;
			} else if(type==1) {
				_this.listLoading = true;
				var url = '/Gift/addGiftTab';
				var params = {
					tab_name: _this.formFour.add.tab_name,
					sort: _this.formFour.add.sort,
				};
				if(params.tab_name!=''&&params.sort!='') {
					_this.formFour.addDialogShow = false;
					axios.get(allget+url, { params: params })					
						.then((res) => {
							if(res.data.ret) {
								_this.listLoading = false;
								baseConfig.successTipMsg(_this, '新增成功！');
								_this.getLabelData();
							} else {
								baseConfig.warningTipMsg(_this, res.data.msg);
							}
						})
						.catch((err) => {
							console.log(err);
						});
				} else {
					_this.listLoading = false; 
					baseConfig.warningTipMsg(_this, '新增的标签名称、排序不能为空！');
				}
			}
		},
		// 编辑标签的操作
		changeLabelUserData(index, rows) {
			var _this = this;
			index = index + (_this.formFour.Page-1)*20; 
			_this.formFour.editor.index = index;
			_this.formFour.editor.id = rows.id;
			_this.formFour.editor.tab_name = rows.tab_name;
			_this.formFour.editor.sort = rows.sort;
			_this.formFour.editordialogShow=true; 
		},
		// 编辑标签的确定,0->取消，1->确定
		editorLabelSure(type) {
			var _this = this;
			if(type==0) {
				alert(1)
				_this.formFour.editorDialogShow = false; 
			} else if(type==1){
				alert(2)
				_this.listLoading = true;
				var url = 'Gift/editGiftTab';
				var params = {
					id: _this.formFour.editor.id,
					tab_name: _this.formFour.editor.tab_name,
					sort: _this.formFour.editor.sort,
				};
				if(params.id!=''&&params.tab_name!=''&&params.sort!='') {
					_this.formFour.editorDialogShow = false;
					axios.get(allget+url, { params: params })
						.then((res) => {
							if(res.data.ret) {
								_this.listLoading = false;
								baseConfig.successTipMsg(_this, '编辑修改成功！');
								// 直接对table表格的内容进行修改
								_this.formFour.TabData[_this.formFour.editor.index].id = params.id;
								_this.formFour.TabData[_this.formFour.editor.index].tab_name = params.tab_name;
								_this.formFour.TabData[_this.formFour.editor.index].sort = params.sort;
							} else {
								baseConfig.warningTipMsg(_this, res.data.msg);
							}
						})
						.catch((err) => {
							console.log(err);
						});
				} else {
					_this.listLoading = false; 
					baseConfig.warningTipMsg(_this, '编辑修改的标签名称、排序不能为空！');
				}
			}
		},
		// 删除标签的操作
		deleteLabel(index, rows) {
			var _this = this;
			index = index + (_this.formFour.Page-1)*20;
			var id = rows.id;
			_this.listLoading = true;
			var url = '/Gift/delGiftTab';
			var params = {
				id: id,
			};
			axios.get(allget+url, { params: params })
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						baseConfig.successTipMsg(_this, '删除成功');
						
						_this.formFour.TabData.splice(index, 1);
						_this.formFour.Totalpage--;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg);
					}

				})
				.catch((err) => {
					console.log(err);
				});
		},	
		// 得到上传文件type(0->pic，1->gif),val(0->礼物新增，1->礼物编辑)
        uploading(event, type, val) {
			console.log('进入文件上传的这边');
			var _this = this;
			if(val==0) { // 礼物新增
				if(type==0) { // 静态图icon图标
					_this.giftUploading.file_pic = event.target.files[0];
			  	    var windowURL = window.URL || window.webkitURL;
					_this.giftUploading.src_pic = windowURL.createObjectURL(event.target.files[0]);
					_this.giftUploading.pic_name = event.target.files[0].name;
				} else if(type==1) { // 动态图icon图标
					_this.giftUploading.file_gif = event.target.files[0];
			  	    var windowURL = window.URL || window.webkitURL;
					_this.giftUploading.src_gif = windowURL.createObjectURL(event.target.files[0]);
					_this.giftUploading.gif_name = event.target.files[0].name;
				} else if(type==2) { // 角标图
					_this.giftUploading.file_corner = event.target.files[0];
			  	    var windowURL = window.URL || window.webkitURL;
					_this.giftUploading.src_corner = windowURL.createObjectURL(event.target.files[0]);			
					_this.giftUploading.corner_name = event.target.files[0].name;
				}
			} else if(val==1) { // 礼物编辑
				if(type==0) { // 静态图icon图标
					_this.giftEditorloading.file_pic = event.target.files[0];//获取文件
			  	    var windowURL = window.URL || window.webkitURL;
					_this.giftEditorloading.src_pic = windowURL.createObjectURL(event.target.files[0]);
					_this.giftEditorloading.pic_name = event.target.files[0].name;
				} else if(type==1) { // 动态图icon图标
					_this.giftEditorloading.file_gif = event.target.files[0];//获取文件
			  	    var windowURL = window.URL || window.webkitURL;
					_this.giftEditorloading.src_gif = windowURL.createObjectURL(event.target.files[0]);
					_this.giftEditorloading.gif_name = event.target.files[0].name;
				} else if(type==2) { // 角标图
					_this.giftEditorloading.file_corner = event.target.files[0];
			  	    var windowURL = window.URL || window.webkitURL;
					_this.giftEditorloading.src_corner = windowURL.createObjectURL(event.target.files[0]);					
					_this.giftEditorloading.corner_name = event.target.files[0].name;
				}
			} else if(val==2) { // 礼物特效
				if(type==0) { // 标识图
					_this.giftSpecial.num_url = event.target.files[0];//获取文件
			  	    var windowURL = window.URL || window.webkitURL;
					_this.giftSpecial.src_num = windowURL.createObjectURL(event.target.files[0]);
					_this.giftSpecial.num_file_name = event.target.files[0].name;
				} else if(type==1) { // 特效图
					_this.giftSpecial.effect_url = event.target.files[0];//获取文件
			  	    var windowURL = window.URL || window.webkitURL;
					// 特效图在这里src进行置空处理
					_this.giftSpecial.src_effect = '';
					_this.giftSpecial.effect_file_name = event.target.files[0].name;
				}
			}
      	}, 
		// 添加礼物的操作
		addGiftSure(type) {
			var _this = this;
			if(type==0) {
				_this.giftUploading.dialogShow = false;
				_this.resetForm();
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
			  	formData.append('sort', _this.giftUploading.params.sort);
			  	formData.append('status', _this.giftUploading.params.status);
			  	formData.append('name', _this.giftUploading.params.name);
			  	formData.append('pic', _this.giftUploading.file_pic);
			  	formData.append('gif', _this.giftUploading.file_gif);
			  	formData.append('corner', _this.giftUploading.file_corner);
			  	formData.append('price', _this.giftUploading.params.price);
			  	formData.append('type', _this.giftUploading.params.type);
			  	formData.append('on_sale_time', baseConfig.changeDateTime(_this.giftUploading.params.on_sale_time, 1));
			  	formData.append('is_vip', _this.giftUploading.params.is_vip);
			  	formData.append('position', _this.giftUploading.params.position);
			  	formData.append('is_turntable', _this.giftUploading.params.is_turntable);
			  	formData.append('probability', _this.giftUploading.params.probability);
			  	formData.append('is_special', _this.giftUploading.params.is_special);
			  	formData.append('stock_num', _this.giftUploading.params.stock_num);
			 	formData.append('is_pool', _this.giftUploading.params.is_pool);
			  	formData.append('week_hot', _this.giftUploading.params.week_hot);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};		
				axios.post(allget+'/Gift/addGift', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.giftUploading.dialogShow = false;								
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							if(formData.get('position')==8) {
								_this.getTableNormalGiftData();
							} else if(formData.get('position')==9) {
								_this.getTableActivityGiftData();
							} else if(formData.get('position')==10) {
								_this.getTableRoomGiftData();
							}
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
		// 房间礼物添加特效
		specialData(index, rows, type) {
			var _this = this;
			if(type=='1') {
				index = index + (_this.formOne.Page-1)*20; 
			} else if(type=='2'){
				index = index + (_this.formTwo.Page-1)*20; 
			} else if(type=='3'){
				index = index + (_this.formThree.Page-1)*20; 
			}else {
				baseConfig.warningTipMsg(_this, '只有房间礼物可以添加特效啦~');				
			}
			_this.getNumGift(rows.id);	
		},			
		// 进行房间礼物请求对应数量的详情
		getNumGift(id) {
			var _this = this;
			var url = '/Gift/getGetEffects';
			var params = {
				id: id,
			};
			axios.get(allget+url, { params: params })
				.then((res) => {
					_this.listLoading = false;
					if(res.data.ret) {
						_this.giftNumTable.dialogShow=true;
						_this.giftNumTable.giftNumArr = res.data.data;
					} else {
						baseConfig.warningTipMsg(_this, res.data.msg); 
					}
				})
				.catch((err) => {
					console.log(err);
				});
		},
		changeAddFun(val){
			var arr = [];
			this.weekData.forEach((item) => {
				if(item.checked){
					arr.push(item.num)
				}
			})
			this.giftUploading.params.week_hot = arr;
		},
		changeEditFun(){
			var arr = [];
			this.weekData.forEach((item) => {
				if(item.checked){
					arr.push(item.num)
				}
			})
			this.giftEditorloading.params.week_hot = arr;
		},
		// 礼物数量进行相对应的编辑
		changeNumGIft(index, row) {
			var _this = this;
			_this.giftSpecial.id = row[index].gift_id;
			_this.giftSpecial.num = row[index].num;
			_this.giftSpecial.is_dynamic_effect = row[index].is_dynamic_effect;
			_this.giftSpecial.num_name = row[index].num_name;
			_this.giftSpecial.num_status = row[index].num_status;
			_this.giftSpecial.src_num = row[index].num_img_url ? row[index].num_img_url : '';
			_this.giftSpecial.src_effect = row[index].dynamic_effect_url ? row[index].dynamic_effect_url : '';
			// 在这里进行对应的文件置空，选中的也进行置空
			_this.giftSpecial.num_url = '';
			_this.giftSpecial.effect_url = '';
			_this.giftSpecial.dialogShow=true;
		},
		// 进行特效礼物的上传
		specialGiftSure(type) {
			var _this = this;
			if(type==0) {
				_this.giftSpecial.dialogShow = false;
				_this.resetForm();				
			} else if(type==1) {
				_this.listLoading = true;
				let formData = new FormData();
				formData.append('id', _this.giftSpecial.id);
				formData.append('num', _this.giftSpecial.num);
				formData.append('is_dynamic_effect', _this.giftSpecial.is_dynamic_effect);
				formData.append('num_name', _this.giftSpecial.num_name);
				formData.append('num_status', _this.giftSpecial.num_status);
				formData.append('num_url', _this.giftSpecial.num_url);
				formData.append('effect_url', _this.giftSpecial.effect_url);
				let config = {
					headers: {
						'Content-Type': 'multipart/form-data'
					}
				};
				axios.post(allget+'/Gift/editGetEffects', formData, config)
					.then((res) => {
						_this.listLoading = false;	
						_this.giftSpecial.dialogShow = false;							
						if(res.data.ret) {	
							baseConfig.successTipMsg(_this, '新增成功！');
							// 本身的房间的数据进行更新、然后对相应的弹窗内容进行修改展示
							_this.getTableRoomGiftData();
							_this.getNumGift(_this.giftSpecial.id);
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
			// 进行请求的兼容处理
			if(tab.label=="黄金宝箱"&&_this.formOne.TabData.length==0) {
				_this.getTableNormalGiftData();
			}
			else if(tab.label=="铂金宝箱"&&_this.formTwo.TabData.length==0) {
				_this.getTableActivityGiftData();
			}
			else if(tab.label=="钻石宝箱"&&_this.formThree.TabData.length==0) {
				_this.getTableRoomGiftData();
			}
			else if(tab.label=="标签管理"&&_this.formFour.TabData.length==0) {
				_this.getLabelData();
			}
		},
		resetForm() {
			// 进行对应的重置
			var _this = this;
			_this.giftUploading = {
				dialogShow: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				file_gif: '',
				src_gif: '',
				gif_name: '',
				file_corner: '',
				src_corner: '',
				corner_name: '',
				params: {
					sort: '',
					name: '',
					price: '',
					pic: '',
					gif: '',
					position: '8',
					status: '1',
					type: '1',
					desc: '',
					on_sale_time: '',
					is_vip: '0',
					is_turntable: '0',
					probability: '',
					is_special: '0',
					stock_num: '',
					week_hot:'',
					is_pool:''
				},
			};
			_this.giftEditorloading = {
				index: '',
				type: '',
				dialogShow: false,
				file_pic: '',
				src_pic: '',
				pic_name: '',
				file_gif: '',
				src_gif: '',
				gif_name: '',
				file_corner: '',
				src_corner: '',
				corner_name: '',
				params: {
					id: '',
					sort: '',
					name: '',
					price: '',
					pic: '',
					gif: '',
					position: '',
					status: '',
					type: '',
					create_time: '',
					on_sale_time: '',
					is_vip: '',
					is_turntable: '',
					probability: '',
					is_special: '',
					stock_num: '',
					week_hot:'',
					is_pool:''
				},
			};
			_this.giftSpecial = {
				dialogShow: false,
				id: '',
				num: '',
				is_dynamic_effect: '',
				num_url: '',
				src_num: '',
				num_file_name: '',
				effect_url: '',
				src_effect: '',
				effect_file_name: '',
				num_name: '',
				num_status: '',
				week_hot:'',
				is_pool:''
			};
			_this.weekData=[
				{num:0,text:'周日',checked:false},
				{num:1,text:'周一',checked:false},
				{num:2,text:'周二',checked:false},
				{num:3,text:'周三',checked:false},
				{num:4,text:'周四',checked:false},
				{num:5,text:'周五',checked:false},
				{num:6,text:'周六',checked:false},
			]
		},
		/*
			downDemo：下载文件
			lookDemo：进入浏览的页面
		*/ 
		downDemo(val) {
			var _this = this;
			window.open(val);
		},
		lookDemo() {
			window.open('http://svga.io/svga-preview.html');
		},
		// 点击查看的svga文件
		lookSvga(val) {
			var _this = this;
			_this.svga.dialogShow=true;
			if(_this.player) {
				_this.parser.load(val, function(videoItem) {
					_this.player.setVideoItem(videoItem);
					_this.player.startAnimation();
				});
			} else {
				setTimeout(function() {
					_this.player = new SVGA.Player('#demoCanvas');
					_this.parser = new SVGA.Parser('#demoCanvas'); // 如果你需要支持 IE6+，那么必须把同样的选择器传给 Parser。
					_this.parser.load(val, function(videoItem) {
						_this.player.setVideoItem(videoItem);
						_this.player.startAnimation();
					});
				}, 500);
			}
		},
	},
	mounted() {
		var _this = this;
		_this.$nextTick(function() {
			_this.tableHeight = baseConfig.lineNumber(tabSearchPageHeight);
			_this.tabSearchHeight = baseConfig.lineNumber(tabSearchHeight);
			_this.getTableNormalGiftData();
			_this.getTableActivityGiftData();
			_this.getTableRoomGiftData();
			_this.getLabelData();
		})
	}
};
</script>

<style lang="css" scoped>
/* 引入的图片展示样式 */
.el-form-item__content{
	overflow:hidden;
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
.el-tab-pane{
	height: 800px;
}
#demoCanvas{
	width: 375px; height: 657px;
	margin: 0 auto;
}
</style>
