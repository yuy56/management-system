<template>
    <div>
        <el-card style="margin:20px 0px">
           <CategorySelect @getCategoryId="getCategoryId" :show="scene!=0"></CategorySelect>
        </el-card>
        <el-card>
            <!-- 底部这里将来是有三部分进行切换 -->
            <div v-show="scene==0">
                <!-- 展示spu列表的结构 -->
                <el-button type="primary" icon="el-icon-plus" :disabled="!category3Id" @click="addSpu">添加spu</el-button>
                <el-table style="width:100%" border :data="records">
                    <el-table-column type="index" label="序号" width="80" align="center"></el-table-column>
                    <el-table-column prop="spuName" label="SPU名称" width="width" ></el-table-column>
                    <el-table-column prop="description" label="SPU描述" width="width" ></el-table-column>
                    <el-table-column prop="prop" label="操作" width="width" >
                        <template slot-scope="{row}">
                            <!-- 这里按钮将来用hintButton替换 -->
                            <hint-button type="success" icon="el-icon-plus" size="mini" title="添加sku" @click="addSku(row)"></hint-button>
                            <hint-button type="warning" icon="el-icon-edit" size="mini" title="修改spu" @click="updateSpu(row)"></hint-button>
                            <hint-button type="info" icon="el-icon-info" size="mini" title="查看当前spu全部sku列表" @click="handler(row)"></hint-button>
                            <el-popconfirm title="确定删除吗？" @onConfirm="deleteSpu(row)">
                                <hint-button slot="reference" type="danger" icon="el-icon-delete" size="mini" title="删除spu" ></hint-button>
                            </el-popconfirm>
                        </template>
                    </el-table-column>
                </el-table>
                <el-pagination 
                style="text-align:center"
                :current-page="page"
                :page-sizes="[3,5,10]"
                :page-size="limit"
                layout="prev,pager,next,jumper,->,sizes,total"
                @current-change="getSpuList"
                @size-change="handleSizeChange"
                :total="total">
                </el-pagination>
            </div>
            <SpuForm v-show="scene==1" @changeScene="changeScene" ref="spu">
            </SpuForm>
            <SkuForm v-show="scene==2" ref="sku" @changeScenes="changeScenes">
            </SkuForm>
        </el-card>
        <el-dialog :title="`${spu.spuName}的sku列表`" :visible.sync="dialogTableVisible" :before-close="close">
            <el-table :data="skuList" style="width:100%" border v-loading="loading">
                <el-table-column property="skuName" label="名称" width="width"></el-table-column>
                <el-table-column property="price" label="价格" width="width"></el-table-column>
                <el-table-column property="weight" label="重量" width="width"></el-table-column>
                <el-table-column label="默认图片" width="width">
                    <template slot-scope="{row}">
                        <img :src="row.skuDefaultImg" alt="" style="width:100px;height:100px">
                    </template>
                </el-table-column>
            </el-table>
        </el-dialog>
    </div>
    <!-- 底部这里将来是有三部分进行切换 -->
</template>

<script>
import SpuForm from './SpuForm'
import SkuForm from './SkuForm'
export default {
    name: 'Spu',
    components:{
        SpuForm,
        SkuForm
    },
    data() {
        return {
            //分别是分类的id
            category1Id:'',
            category2Id:'',
            category3Id:'',
            page:1,//分页器当前第几页
            limit:3,//每一页需要展示多少条数据
            records:[],//spu列表的数据
            total:0,//分页器一共需要展示数据的条数
            scene:0,//0代表展示spu列表数据，1添加spu｜修改spu，2添加sku
            dialogTableVisible:false,//控制对话框的显示与隐藏
            spu:{},
            skuList:[],//存储的是sku列表的数据
            loading:true,
        };
    },

    mounted() {
        
    },

    methods: {
        //三级联动的自定义事件，可以把子组件相应的id传递给父母
        getCategoryId({categoryId,level}){
            //categoryId：获取到一二三级分类的id,level：为了区分是几级id
            if(level==1){
                this.category1Id=categoryId
                //清除2、3级分类的id
                this.category2Id=''
                this.category3Id=''
            }else if(level==2){
                this.category2Id=categoryId
                //清除3级id
                this.category3Id=''
            }else{
                this.category3Id=categoryId
                //获取spu列表数据进行展示
                this.getSpuList()
            }
        },
        //获取spu列表数据的方法
        async getSpuList(pages=1){
            this.page=pages
            const {page,limit,category3Id} = this
            //携带三个参数：page 第几页，limit 每一页需要展示多少条数据
            let result = await this.$API.spu.reqSpuList(page,limit,category3Id)
            if(result.code==200){
                this.total=result.data.total
                this.records=result.data.records
            }
        },
        // handleCurrentChange(page){
        //     this.page=page
        //     this.getSpuList()
        // },
        //当分页器的某一个展示数据条数发生变化的回调
        handleSizeChange(limit){
            //修改参数
            this.limit=limit
            //再发请求
            this.getSpuList()
        },
        //添加spu按钮的回调
        addSpu(){
            this.scene=1
            //通知子组件souform发请求---两个
            this.$refs.spu.addSpuData(this.category3Id)
        },
        //修改某一个spu
        updateSpu(row){
            this.scene=1
            //获取子组件spuform
            this.$refs.spu.initSpuData(row)
        },
        //自定义事件回调(spuform)
        changeScene({scene,flag}){
            //子组件通知父组件切换场景，需要再次获取spu列表的数据进行展示
            this.scene=scene
            if(flag=="修改"){
                this.getSpuList(this.page)
            }else{
                this.getSpuList()
            }
        },
        //删除spu的回调
        async deleteSpu(row){
            let result = await this.$API.spu.reqDeleteSpu(row.id)
            if(result.code==200){
                this.$message({type:'success',message:'删除成功'})
                //代表spu个数大于1删除的时候停留在当前页，如果spu个数小于1:回到上一页
                this.getSpuList(this.records.length>1?this.page:this.page-1)
            }
        },
        //添加sku按钮的回调
        addSku(row){
            //切换场景为2
            this.scene=2
            //父组件调用子组件的方法，让子组件发请求
            this.$refs.sku.getData(this.category1Id,this.category2Id,row)
        },
        //skuForm通知父组件修改场景
        changeScenes(scene){
            this.scene=scene
        },
        //查看sku的按钮的回调
        async handler(spu){
            //点击这个按钮的时候，对话框是可见的
            this.dialogTableVisible=true
            //保存spu信息
            this.spu=spu
            //获取sku列表的数据进行展示
            let result= await this.$API.spu.reqSkuList(spu.id)
            if(result.code==200){
                this.skuList=result.data
                //loading隐藏
                this.loading=false
            }
        },
        //关闭对话框的回调
        close(done){
            //loading属性再次变为真
            this.loading=true
            //清除sku列表的数据
            this.skuList=[]
            done()
        }
    },
};
</script>

<style lang="scss" scoped>

</style>