<template>
	<div class="grade">
		<el-row class="topBtns">
			<el-col :span="12">
				<el-input style="width:220px;"></el-input>
			</el-col>
			<el-col :span="12" class="content-right">
				<el-button @click="toaddGrade">添加</el-button>
				<el-button @click="toBatchDelete">批量删除</el-button>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
			<!-- {{selectIds}} -->
				<el-table
				    ref="multipleTable"
				    :data="getGradeList"
				    tooltip-effect="dark"
				    style="width: 100%;text-align:center"
				    @selection-change="handleSelectionChange"
					border
					size="mini"
					v-loading="gradeTblloading"

				    >
				    <el-table-column
				      type="selection"
				      prop="id"
				      width="55"
					
				      >
				    </el-table-column>
				    <el-table-column
				      prop="name"
				      label="名称"
				      >
				     
				    </el-table-column>
				    <el-table-column
				      prop="description"
				      label="简介"
				      >
				    </el-table-column>
				    <el-table-column
				      label="操作"
				      show-overflow-tooltip>
				      <template slot-scope="scope">
				      	<el-button type="text" @click="toEditGrade(scope.row)">修改</el-button>
						<el-button type="text" @click="toDeleteGrade(scope.row)">删除</el-button>
				      </template>
				    </el-table-column>
				  </el-table>
			

			</el-col>
		</el-row>
			
			<el-dialog :title="gradeDialog.title" :visible.sync="gradeDialog.visibile" @close="closeDialog">
			{{gradeDialog.form}}
			  <el-form :model="gradeDialog.form" label-width="100px" size="small" :rules="rules" ref="gradeDialogForm">
			    <el-form-item label="年级名称" prop="name">
			      <el-input v-model="gradeDialog.form.name" auto-complete="off"></el-input>
			    </el-form-item>
			    <el-form-item label="年级简介" prop="description">
			      <el-input type="textarea" v-model="gradeDialog.form.description"></el-input>
			    </el-form-item>
			  </el-form>
			  <div slot="footer" class="dialog-footer">
			    <el-button @click="gradeDialog.visibile = false">取 消</el-button>
			    <el-button type="primary" @click="addGrade">确 定</el-button>
			  </div>
			</el-dialog>
		

			

	</div>
</template>


<script>
	import {mapGetters,mapActions} from 'vuex'
	export default{
		data(){
			return{
				gradeDialog:{
					visibile:false,
					title:'',
					form:{
						id:'',
						name:'',
						description:''
					}
				},
				rules:{
					name:[
					{required:true,message:'请输入年级名称',trigger:'blur'}
					],
					description:[
						{required:true,message:'请输入年级简介',trigger:'blur'}
					]
				},
				gradeTblloading:false,
				selectIds:[]
			}
		},
		computed:{
			// getGradeList(){
			// 	return [{id:1,name:'webui',description:'啦啦啦'},{id:2,name:'javaee',description:'呜呜呜'}];
			// }
			...mapGetters(['getGradeList'])
		},
		created(){
			this.loadGradeList();
		},
		methods:{
			...mapActions(['findGradeList',
						'saveOrUpdateGrade',
						'deleteGrade',
						'batchDeleteGrade'
				]),
		   loadGradeList(){
				this.gradeTblloading=true;
				this.findGradeList().then(()=>{
					this.gradeTblloading=false;
					
				}).catch(()=>{
					this.$notify.error({
				          title: '错误',
				          message: '数据加载异常'
				        });
				});
			},
			handleSelectionChange(val) {
				// console.log(val);
		        this.selectIds = val;
		      },
		    toEditGrade(row){
		    	this.gradeDialog.title="修改年级";
		    	this.gradeDialog.form=row;
		    	this.gradeDialog.visibile=true;
		    },
		    toaddGrade(){
		    	
		    	this.gradeDialog.title="添加年级";
		    	this.gradeDialog.visibile=true;
		    	// console.log(this.gradeDialog.visibile);
		    },
		    toDeleteGrade(row){
		    	this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        }).then(() => {
		        	this.deleteGrade(row.id).then(()=>{
		        		this.$notify({
		        			title:'成功',
		        			message:'删除成功',
		        			type:'success'
		        		});
		        		this.loadGradeList();
		        	}).catch((error)=>{
		        		this.$notify({
		        			title:'错误',
		        			message:error.message,
		        			type:'error'
		        		});
		        	});
		          
		        }).catch(() => {
		            this.$notify({
		            	title:'错误',
		            	message:'取消删除',
		            	type:'error'
		            });    
		        });
		    	
		    },
		    toBatchDelete(){
		    	this.$confirm('此操作将永久删除文件，是否继续','提示',{
		    		confirmButtonText:'确定',
		    		cancelButtonText:'取消',
		    		type:'warning'
		    	}).then(()=>{
		    		let ids=this.selectIds.map((item)=>{
		    			return item.id;
		    		});
		    		this.batchDeleteGrade(ids.join()).then(()=>{
		    			this.$notify({
		    				title:'成功',
		    				message:'删除成功',
		    				type:'success'
		    			});
		    			this.loadGradeList();
		    		}).catch((error)=>{
		    			this.$notify({
		    				title:'失败',
		    				message:'删除失败',
		    				type:'error'
		    			});
		    		});


		    	}).catch(()=>{
		    		this.$notify({
		    			title:'错误',
		    			message:'取消删除',
		    			type:'error'
		    		});
		    	});
		    },
		   addGrade(){
		   
		   		//1.对用户输入的数据进行校验
		   		//2.如果校验成功，获得数据，提交数据
		   		//3.清空模态框中的额数据
		   		//4.关闭模态框
		   		 this.$refs['gradeDialogForm'].validate((valid) => {
		          if (valid) {
		          	let grade=this.gradeDialog.form;
		            this.saveOrUpdateGrade(grade).then(()=>{
		            	this.gradeDialog.visibile=false;
		            	this.$notify({
				          title: '成功',
				          message: '添加成功',
				          type: 'success'
				        });
				        this.loadGradeList();
		            }).catch(()=>{
		            	this.$notify.error({
				          title: '错误',
				          message: '添加失败'
				        });
						            });

		          } else {
		            console.log('error submit!!');
		            return false;
		          }
		        });
		   		
		   		
		   		
		   },
		   closeDialog(){

		   	this.$refs['gradeDialogForm'].resetFields();
		   	this.gradeDialog.form={
		   		id:'',
		   		name:'',
		   		description:''
		   	}
		   },


		   // handlerClose(done){
		   // 	alert("aaa");
		   // 	done();
		   // }

		}
	}
</script>


<style scoped>
	.topBtns{
		margin-bottom: .5em;
	}
</style>