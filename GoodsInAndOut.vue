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
      <el-button @click="inGoods" size="small" type="primary">入库</el-button>
      <el-button @click="outGoods" size="small" type="danger">出库</el-button>
    </div>
  <el-table :data="tableData" border highlight-current-row @current-change="selectCurrentChange">
        <el-table-column prop="id" label="ID" width="50">
        </el-table-column>
        <el-table-column prop="name" label="货物名" width="200">
        </el-table-column>
        <el-table-column prop="storage" label="仓库" width="200" :formatter="formatStorage">
        </el-table-column>
        <el-table-column prop="goodstype" label="分类" width="200" :formatter="formatGoodsType">
        </el-table-column>
        <el-table-column prop="count" label="数量" width="200">
        </el-table-column>
        <el-table-column prop="remark" label="备注" align="center">
        </el-table-column>
  </el-table>
<!-- 入库表单 -->
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
      :visible.sync="inDialogVisible"
      width="30%"
      center>
      <el-form ref="formIn" :rules="rulesIn" :model="formIn" label-width="80px">

        <el-form-item label="物品名">
          <el-col :span="19">
            <el-input v-model="formIn.goodsname" readonly></el-input>
          </el-col>  
        </el-form-item>

        <el-form-item label="数量" prop="count">
          <el-col :span="19">
            <el-input v-model="formIn.count"></el-input>
          </el-col>  
        </el-form-item>

        <!-- <el-form-item label="备注" prop="remark">
          <el-col :span="19">
            <el-input type="textarea" v-model="formIn.remark"></el-input>
          </el-col>  
        </el-form-item> -->

      </el-form>  
      <span slot="footer" class="dialog-footer">
        <el-button @click="inDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="insave">确 定</el-button>
      </span>
    </el-dialog>

<!-- 出库表单 -->
    <el-dialog
      title="提示"
      :visible.sync="outDialogVisible"
      width="30%"
      center>
      <el-form ref="formOut" :rules="rulesOut" :model="formOut" label-width="80px">

        <el-form-item label="物品名">
          <el-col :span="19">
            <el-input v-model="formOut.goodsname" readonly></el-input>
          </el-col>  
        </el-form-item>

        <el-form-item label="数量" prop="count">
          <el-col :span="19">
            <el-input v-model="formOut.count"></el-input>
          </el-col>  
        </el-form-item>

        <!-- <el-form-item label="备注" prop="remark">
          <el-col :span="19">
            <el-input type="textarea" v-model="formOut.remark"></el-input>
          </el-col>  
        </el-form-item> -->

      </el-form>  
      <span slot="footer" class="dialog-footer">
        <el-button @click="outDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="outsave">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>

export default {
    name: "GoodsManage",
    data() {
      let checkCount = (rule ,value ,callback) => {
        if(value>this.countMax){
          callback(new Error('输入数量过大'));
        }else{
          callback();
        }
      };
      return {
        user: JSON.parse(sessionStorage.getItem('CurUser')),
        goodstypeData:[],
        storageData:[],
        tableData:[],
        pageNum:1,
        pageSize:9,
        total:0,
        nameText:'',
        storage:'',
        goodstype:'',
        inDialogVisible: false,
        outDialogVisible: false,
        currentRow:{},
        countMax:0,
        formIn:{
            goods:'',
            goodsname:'',
            count:'',
            userid:'',
            remark:''
        },
        formOut:{
            goods:'',
            goodsname:'',
            count:'',
            userid:'',
            remark:''
        },
        rulesIn: {
          count:[
            {required: true, message: '请输入数量', trigger: 'blur'},
            {pattern: /^([1-9][0-9]*){1,4}$/, message: '数量必须为正整数',trigger: 'blur'},
          ]
        },
        rulesOut: {
          count:[
            {required: true, message: '请输入数量', trigger: 'blur'},
            {pattern: /^([1-9][0-9]*){1,4}$/, message: '数量必须为正整数',trigger: 'blur'},
            {validator: checkCount, trigger: 'blur'}
          ]
        }
      }
    },
    methods:{
      selectCurrentChange(val){
          this.currentRow=val;
      },
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
      resetInForm(){
        this.$refs.formIn.resetFields();
      },
      resetOutForm(){
        this.$refs.formOut.resetFields();
      },
      inGoods(){
        if(!this.currentRow.id){
            alert('请选择记录');
            return;
        }
        this.inDialogVisible = true;
        this.$nextTick(()=>{
          this.resetInForm();
        })
        this.formIn.goodsname = this.currentRow.name;
        this.formIn.goods = this.currentRow.id;
        this.formIn.userid = this.user.id
      },
      doinGoods(){
        this.$axios.post(this.$httpUrl+'/record/insave',this.formIn).then(res=>res.data).then(res=>{
            console.log(res)
            if(res.code==200){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.inDialogVisible = false;
              this.loadPost()
              this.resetInForm()
            }else{
              this.$message.error('操作失败');
            } 
        })
      },
      outGoods(){
        if(!this.currentRow.id){
            alert('请选择记录');
            return;
        }
        this.outDialogVisible = true;
        this.$nextTick(()=>{
          this.resetOutForm();
        })
        this.formOut.goodsname = this.currentRow.name;
        this.formOut.goods = this.currentRow.id;
        this.formOut.userid = this.user.id
        this.countMax = this.currentRow.count;
      },
      dooutGoods(){
        this.$axios.post(this.$httpUrl+'/record/outsave',this.formOut).then(res=>res.data).then(res=>{
            console.log(res)
            if(res.code==200){
              this.$message({
                message: '操作成功',
                type: 'success'
              });
              this.outDialogVisible = false;
              this.loadPost()
              this.resetOutForm()
            }else{
              this.$message.error('操作失败');
            } 
        })
      },
      outsave(){
        this.$refs.formOut.validate((valid) => {
          if (valid) {
            this.dooutGoods();
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      insave(){
        this.$refs.formIn.validate((valid) => {
          if (valid) {
            this.doinGoods();
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
            this.total=res.total;
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