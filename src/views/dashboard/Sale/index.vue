<template>
    <el-card class="box-card" style="margin:10px 0px">
        <div slot="header" class="clearfix">
            <!--  @tab-click="handleClick" -->
            <!-- 头部左侧内容 -->
            <el-tabs class="tab" v-model="activeName">
                <el-tab-pane label="销售额" name="sale"></el-tab-pane>
                <el-tab-pane label="访问量" name="visit"></el-tab-pane>
            </el-tabs>
            <!-- 头部右侧内容 -->
            <div class="right">
                <span @click="setDay">本日</span>
                <span @click="setWeek">本周</span>
                <span @click="setMonth">本月</span>
                <span @click="setYear">本年</span>
                <el-date-picker
                class="date"
                v-model="date"
                type="daterange"
                range-separator="-"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                size="mini"
                value-format="yyyy-MM-dd">
                </el-date-picker>
            </div>
        </div>
        <div>
            <el-row :gutter="10">
                <el-col :span="18">
                    <div class="charts" ref="charts"></div>
                </el-col>
                <el-col :span="6">
                    <h3>门店{{title}}排名</h3>
                    <ul>
                        <li>
                            <span class="rindex">0</span>
                            <span class="rtext">肯德基</span>
                            <span class="rvalue">123456</span>
                        </li>
                        <li>
                            <span class="rindex">1</span>
                            <span class="rtext">肯德基</span>
                            <span class="rvalue">123456</span>
                        </li>
                        <li>
                            <span class="rindex">2</span>
                            <span class="rtext">肯德基</span>
                            <span class="rvalue">123456</span>
                        </li>
                        <li>
                            <span class="rindex1">3</span>
                            <span class="rtext">肯德基</span>
                            <span class="rvalue">123456</span>
                        </li>
                        <li>
                            <span class="rindex1">4</span>
                            <span class="rtext">肯德基</span>
                            <span class="rvalue">123456</span>
                        </li>
                        <li>
                            <span class="rindex1">5</span>
                            <span class="rtext">肯德基</span>
                            <span class="rvalue">123456</span>
                        </li>
                        <li>
                            <span class="rindex1">6</span>
                            <span class="rtext">肯德基</span>
                            <span class="rvalue">123456</span>
                        </li>
                    </ul>
                </el-col>
            </el-row>
        </div>
</el-card>
</template>

<script>
//引入echarts
import * as echarts from 'echarts'
import * as dayjs from 'dayjs'
import {mapState} from 'vuex'
export default {
    name: '',

    data() {
        return {
            activeName:"sale",
            mycharts:null,
            date:[]
        };
    },

    mounted() {
        //初始化echarts实例
        this.mycharts = echarts.init(this.$refs.charts)
        //配置数据
        this.mycharts.setOption({
            title:{
                text:'销售额趋势'
            },
            tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            }
            },
            grid: {
              left: '3%',
              right: '4%',
              bottom: '3%',
              containLabel: true
            },
            xAxis: [
              {
                type: 'category',
                data: [],
                axisTick: {
                  alignWithLabel: true
                }
              }
            ],
            yAxis: [
              {
                type: 'value',
              }
            ],
            series: [
              {
                name: 'Direct',
                type: 'bar',
                barWidth: '60%',
                data: []
              }
            ]
        })
    },

    methods: {
        setDay(){
            const day = dayjs().format('YYYY-MM-DD')
            this.date=[day,day]
        },
        setWeek(){
            const start = dayjs().day(0).format('YYYY-MM-DD')
            const end=dayjs().day(6).format('YYYY-MM-DD')
            this.date=[start,end]
        },
        setMonth(){
            const start=dayjs().startOf('month').format('YYYY-MM-DD')
            const end=dayjs().endOf('month').format('YYYY-MM-DD')
            this.date=[start,end]
        },
        setYear(){
            const start=dayjs().startOf('year').format('YYYY-MM-DD')
            const end=dayjs().endOf('year').format('YYYY-MM-DD')
            this.date=[start,end]
        }   
    },

    computed:{
        //计算属性-标题
        title(){
            return this.activeName=='sale'?'销售额':'访问量'
        },
        ...mapState({
            listState:state=>state.home.list
        })
    },

    watch:{
        title(){
            //重新修改图表的配置数据
            this.mycharts.setOption({
                title:{
                    text:this.title+'趋势'
                },
                xAxis:{
                    data:this.title=='销售额'?this.listState.orderFullYearAxis:this.listState.userFullYearAxis
                },
                series:[
                    {
                        name: 'Direct',
                        type: 'bar',
                        barWidth: '60%',
                        data: this.title=='销售额'?this.listState.orderFullYear:this.listState.userFullYear
                    }
                ]
            })
        },
        listState(){
            this.mycharts.setOption({
            title:{
                text:'销售额趋势'
            },
            tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            }
            },
            grid: {
              left: '3%',
              right: '4%',
              bottom: '3%',
              containLabel: true
            },
            xAxis: [
              {
                type: 'category',
                data: this.listState.orderFullYearAxis,
                axisTick: {
                  alignWithLabel: true
                }
              }
            ],
            yAxis: [
              {
                type: 'value',
                axisTick: {
                  alignWithLabel: true
                }
              }
            ],
            series: [
              {
                name: 'Direct',
                type: 'bar',
                barWidth: '60%',
                data: this.listState.orderFullYear
              }
            ]
        })
        }
    }
};
</script>

<style lang="scss">
.el-card__header{
    border-bottom:none
}
.clearfix{
    position:relative;
    display: flex;
    justify-content: space-between;
}
.tab{
    width:100%
}
.right{
    position: absolute;
    right:0px;
}
.date{
    width:200px;
    margin:0px 20px;
}
.right span{
    margin:0px 10px
}
.charts{
    width:100%;
    height:300px
}
ul{
    list-style: none;
    width:100%;
    height:300px;
    padding:0px
}
ul li{
    height:8%;
    margin:10px 0px
}
.rindex{
    float:left;
    width:18px;
    height:18px;
    border-radius:50%;
    background:black;
    color:white;
    text-align:center;
}
.rtext{
    margin-left:20px
}
.rvalue{
    float:right
}
.rindex1{
    margin-left:5px
}
</style>