<template>
  <div>
    <div style="margin-bottom: 5px">
      <el-input v-model="nameText" placeholder="请输入物品名" style="width: 200px;"
        @keyup.enter.native="loadPost"/>

      <el-select v-model="storage" placeholder="请选择仓库" style="margin-left: 5px; width: 150px">
        <el-option
          v-for="item in storageData"
          :key="item.id"
          :label="item.name"
          :value="item.id">
        </el-option>
      </el-select>

      <el-select v-model="goodstype" placeholder="请选择分类" style="margin-left: 5px; width: 150px">
        <el-option
          v-for="item in goodstypeData"
          :key="item.id"
          :label="item.name"
          :value="item.id">
        </el-option>
      </el-select>

      <el-button @click="loadPost" size="small" type="primary" icon="el-icon-search" circle style="margin-left: 5px"></el-button>
      <el-button @click="reset" size="small" type="danger" icon="el-icon-delete" circle></el-button>
      <el-button @click="add" size="small" type="primary" icon="el-icon-plus" circle v-if="user.grade==0"></el-button>
    </div>
  <el-table :data="tableData" border>
        <el-table-column prop="id" label="ID" width="50">
        </el-table-column>
        <el-table-column prop="name" label="货物名" width="200">
        </el-table-column>
        <el-table-column prop="storage" label="仓库" width="180" :formatter="formatStorage">
        </el-table-column>
        <el-table-column prop="goodstype" label="分类" width="180" :formatter="formatGoodsType">
        </el-table-column>
        <el-table-column prop="count" label="数量" width="120">
        </el-table-column>
        <el-table-column prop="remark" label="备注" width="340" align="center">
        </el-table-column>
        <el-table-column prop="operate" label="操作" align="center">
          <template slot-scope="scope">
            <el-button size="small" type="primary" @click="mod(scope.row)">编辑</el-button>
            <el-popconfirm title="确定删除吗？" @confirm="del(scope.row.id)" style="margin-left: 5px">
              <el-button size="small" slot="reference" type="danger">删除</el-button>
            </el-popconfirm>
          </template>
        </el-table-column>
  </el-table>

    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="pageNum"
      :page-size="9"
      layout="prev, pager, next, jumper"
      :total="total">
    </el-pagination>

    <el-dialog
      title="提示"
      :visible.sync="centerDialogVisible"
      width="30%"
      center>
      <el-form ref="form" :rules="rules" :model="form" label-width="80px">

        <el-form-item label="物品名" prop="name">
          <el-col :span="19">
            <el-input v-model="form.name"></el-input>
          </el-col>  
        </el-form-item>

        <el-form-item label="仓库" prop="storage">
          <el-col :span="19">
            <el-select v-model="form.storage" placeholder="请选择仓库" style="margin-left: 5px; width: 150px">
              <el-option
                v-for="item in storageData"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </el-col>  
        </el-form-item>

        <el-form-item label="分类" prop="goodstype">
          <el-col :span="19">
            <el-select v-model="form.goodstype" placeholder="请选择分类" style="margin-left: 5px; width: 150px">
              <el-option
                v-for="item in goodstypeData"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </el-col>  
        </el-form-item>

        <!-- <el-form-item label="数量" prop="count">
          <el-col :span="19">
            <el-input v-model="form.count"></el-input>
          </el-col>  
        </el-form-item> -->

        <!-- <el-form-item label="备注" prop="remark">
          <el-col :span="19">
            <el-input type="textarea" v-model="form.remark"></el-input>
          </el-col>  
        </el-form-item> -->

      </el-form>  
      <span slot="footer" class="dialog-footer">
        <el-button @click="centerDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="save">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>

export default {
    name: "GoodsManage",
    data() {
       let checkDuplicate = (rule,value,callback)=>{
         if(this.form.id){
           return callback();
         }
         this.$axios.get(this.$httpUrl+"/goods/findByName?name="+this.form.name).then(res=>res.data).then(res=>{
           if(res.code!=200){
            console.log(res.code+"aaaaaaaaaaAAAAAAAAAAAAAAAAAAAAAAAAA")
             callback()
           }else{
             callback(new Error('该物品名称已经存在'))
           }
         })
       };
      // let checkCount = (rule ,value ,callback) => {
      //   if(value>9999){
      //     callback(new Error('输入数量过大'));
      //   }else{
      //     callback();
      //   }
      // };
      return {
        user: JSON.parse(sessionStorage.getItem('CurUser')),
        goodstypeData:[],
        storageData:[],
        tableData:[],
        pageNum:1,
        pageSize:9,
        total:0,
        nameText:'',
        centerDialogVisible: false,
        storage:'',
        goodstype:'',
        form:{
          id:'',
          name:'',
          storage:'',
          goodstype:'',
          count:'',
          remark:'',
        },
        rules: {
          name: [
            { required: true, message: '请输入物品名', trigger: 'blur' },
            {validator: checkDuplicate,trigger: 'blur'}
          ],
          storage: [
            { required: true, message: '请选择仓库', trigger: 'blur' },
            // {validator: checkDuplicate,trigger: 'blur'}
          ],
          goodstype: [
            { required: true, message: '请选择分类', trigger: 'blur' },
            // {validator: checkDuplicate,trigger: 'blur'}
          ],
          // count:[
          //   {required: true, message: '请输入数量', trigger: 'blur'},
          //   {pattern: /^([1-9][0-9]*){1,4}$/, message: '数量必须为正整数',trigger: 'blur'},
          //   {validator: checkCount, trigger: 'blur'}
          // ]
        }
      }
    },
    methods:{
      formatStorage(row){
        let temp = this.storageData.find(item=>{
          return item.id == row.storage
        })
        return temp && temp.name
      },
      formatGoodsType(row){
        let temp = this.goodstypeData.find(item=>{
          return item.id == row.goodstype
        })
        return temp && temp.name
      },
      resetForm(){
        this.$refs.form.resetFields();
      },
      add(){
        
        this.centerDialogVisible = true;
        this.$nextTick(()=>{
          this.resetForm();
          //reset后form中存在id不为空的问题
          this.form.id = '';
        })
      },
      mod(row){
        this.centerDialogVisible = true;
        this.$nextTick(()=>{
          this.form.id = row.id;
          this.form.name = row.name;
          this.form.storage = row.storage;
          this.form.goodstype = row.goodstype;
          this.form.count = row.count;
          this.form.remark = row.remark;
        })
      },
      del(id){
        this.$axios.get(this.$httpUrl+'/goods/del?id='+id).then(res=>res.data).then(res=>{
            console.log(res)
            if(res.code==200){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.loadPost()
            }else{
              this.$message.error('操作失败');
            } 
        })
      },
      doSave(){
        this.$axios.post(this.$httpUrl+'/goods/add',this.form).then(res=>res.data).then(res=>{
            console.log(res)
            if(res.code==200){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.centerDialogVisible = false;
              this.loadPost()
              this.resetForm()
            }else{
              this.$message.error('操作失败');
            } 
        })
      },
      doMod(){
        this.$axios.post(this.$httpUrl+'/goods/mod',this.form).then(res=>res.data).then(res=>{
            console.log(res)
            if(res.code==200){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.centerDialogVisible = false;
              this.loadPost()
              this.resetForm()
            }else{
              this.$message.error('该物品名称已存在');
            } 
        })
      },
      save(){
        this.$refs.form.validate((valid) => {
          if (valid) {
            if(this.form.id){
              this.doMod();
            }else{
              this.doSave()
            }
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.pageNum=1;
        // this.pageSize=val;
        this.loadPost()
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        // this.pageNum=val;
        this.loadPost()
      },
      reset(){
        this.nameText='',
        this.storage='',
        this.goodstype=''
      },
      loadPost(){
        this.$axios.post(this.$httpUrl+'/goods/getPage',{
          pageNum:this.pageNum,
          pageSize:this.pageSize,
          param:{
            name: this.nameText,
            goodstype: this.goodstype,
            storage:this.storage
          }
          
          }).then(res=>res.data).then(res=>{
          console.log(res)
          if(res.code==200){
            this.tableData=res.data;
            this.total=res.total
          }else{
            alert('获取数据失败')
          }
        })
      },
      loadStorage(){
        this.$axios.get(this.$httpUrl+'/storage/list').then(res=>res.data).then(res=>{
          console.log(res)
          if(res.code==200){
            this.storageData=res.data;
          }else{
            alert('获取数据失败')
          }
        })
      },
      loadGoodsType(){
        this.$axios.get(this.$httpUrl+'/goodstype/list').then(res=>res.data).then(res=>{
          console.log(res)
          if(res.code==200){
            this.goodstypeData=res.data;
          }else{
            alert('获取数据失败')
          }
        })
      }
    },
    beforeMount(){
      this.loadStorage();
      this.loadGoodsType();
      this.loadPost();
    }
}
</script>

<style scoped>

.el-input{
    justify-content: center;
    align-items: center;
}

</style>