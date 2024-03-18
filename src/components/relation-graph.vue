<template>
  <div>
    <div style="height: 100vh">
      <RelationGraph ref="graphRef" :options="graphOptions" :on-node-click="onNodeClick" :on-line-click="onLineClick">
        <template #node="{node}">
          <div >
            <div class="node-body" @mouseenter="handelMouseenter(node,$event)" @mouseleave="handelMouseleave(node,$event)">
                <div class="node-body-item" :style="[{backgroundColor:(node.data.direction==='IN'?'rgb(255, 96, 96)':'rgb(18, 139, 237)')},{color:'#ffffff'}]">{{node.text}}</div>
               <div class="node-body-item" v-for="(i,index) in node.data.collection" :key="i.id" v-if="index < 3">
                 <span>{{i.nodeName}}</span>
               </div>
              <span @click="detailOpen = true" v-if="node.data.collection" class="node-body-item">查看全部{{node.data.collection.length}}家人员/企业</span>
            </div>
          </div>
        </template>
        <template slot="graph-plug" >
          <div class="item-detail" v-if="detailOpen">
            <div class="detail-header">
              <span>{{listTitle}}</span>
              <span class="close" @click="detailOpen = false">×</span>
            </div>
            <div class="item-search">
              <input  v-model="searchValue" />
              <span style="cursor: pointer" @click="openAll">{{openText}}</span>
            </div>
            <div class="item-list">
              <div v-for="(i,index) in listData1" :key="i.id">
                <div class="item-list-top" @click="openDetail(i)">
                  <div class="item-list-avater">
                    {{i.nodeName[0]}}
                  </div>
                  <div class="item-list-title">
                    <span>
                     {{index + 1}}.{{i.nodeName}}
                    </span>
                  </div>
                  <div class="item-list-open">
                    <img src="../assets/open.png">
                  </div>
                </div>
                <div class="item-list-detail" v-if="i.open">
                  <div v-for="j in i.paths" :key="j.id">
                    <div class="company-1">
                      <span>{{j.nodeName}}</span>
                    </div>
                    <div class="link" v-if="j.operation">
                      <span>{{j.operation}}</span>
                      <span class="link-line"></span>
                      <span>{{j.reason}}</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </template>
      </RelationGraph>
    </div>
  </div>
</template>
<script>
import RelationGraph from 'relation-graph'
import data from "@/assets/data";
export default {
  name: "relation-graph",
  components: {RelationGraph },
  data() {
    return {
      openText:'全部展开',
      detailOpen:false,
      listTitle:'',
      listData:[],
      chrildNode:[],
      lines:[],
      rootNode:{id: data['id'].toString(), text: data['nodeName'],index:0 },
      originalData:data,
      currentCase: '纵向树状图谱',
      isShowCodePanel: false,
      graphOptions: {
        debug: false,
        layouts: [
          {
            layoutName: 'tree',
            from: 'top',
            min_per_width: 500,
            max_per_width: 1200,
            min_per_height: 600,
            max_per_height: 1200

          }
        ],
        allowShowMiniToolBar: false,
        defaultNodeShape: 1,
        defaultLineShape: 4,
        defaultNodeBorderWidth: 0,
        defaultNodeColor: '#ffffff',
        // defaultNodeWidth: 200,
        // defaultNodeHeight: 50,
        defaultJunctionPoint: 'tb',
      },
      searchValue:''
    }
  },
  mounted() {
    this.showGraph()
  },
  computed:{
    listData1(){
      return this.searchValue === ''? this.listData:this.listData.filter(a => a.nodeName.indexOf(this.searchValue) >= 0)
    }
  },
  methods: {
   async showGraph() {

      const jsonData = {
        rootId: '105605',
        nodes: [],
        lines: []
      }

     await  this.transformData(this.originalData)
     await  this.transformLines(this.chrildNode)
     this.$forceUpdate()

     // 计算插入位置（数组的中间位置）
     let  insertIndex = Math.floor(this.chrildNode.length / 2);
    // 使用splice()方法插入元素
    this.chrildNode.splice(insertIndex, 0, this.rootNode);
     jsonData.nodes = this.chrildNode
     jsonData.lines =this.lines

      // 以上数据中的node和link可以参考"Node节点"和"Link关系"中的参数进行配置
      // this.$refs.graphRef.setJsonData(jsonData, (graphInstance) => {
      //   // Called when the relation-graph is completed
      // })

     this.showSeeksGraph(this.$refs.graphRef,jsonData,)
    },
    transformData(data, depth = 1, parent = null){
      for (let key in data) {
        // eslint-disable-next-line
        if (data.hasOwnProperty(key)) {
          if (typeof data[key] === 'object') {
            // 如果当前属性是一个对象
            if (Array.isArray(data[key])) {
              // 如果是数组
              data[key].forEach(item => {
                item.id =  this.generateRandomId(10)
                // 对数组中的每个元素进行处理
                this.chrildNode.push({
                  id: (item.id).toString(),
                  text: item.title,
                  index: depth,
                  data:{
                    collection: item.collection,
                    operation:item.operation,
                    direction:item.direction,
                  },
                  parent: parent ? parent.id : null // 添加父节点引用
                });

                if (Array.isArray(item.children) && item.children.length > 0) {
                  // 如果当前元素有子节点，则递归处理子节点
                  this.transformData({ children: item.children }, depth + 1, item); // 传递父节点引用
                }
              });
            } else {
              // 如果不是数组，递归处理子对象
              this.transformData(data[key], depth + 1, parent);
            }
          }
        }
      }
    },
    transformLines(nodeData){
      nodeData.forEach((item,index) => {
        const line = {
          from: item.id,
          to: '',
          text: item.data.operation,
          lineShape: 4
        };

        if (item.parent === null) {
          // 如果父节点为 null，则根据节点的 index 奇偶性设置 from 和 to
          if (index % 2 === 0) {
            line.from = this.originalData['id'].toString(); // root 节点
            line.to = item.id;
            line.showStartArrow=true
            line.showEndArrow = false
          } else {
            line.from = item.id;
            line.to = this.originalData['id'].toString(); // root 节点
          }
        } else {
          const parentItem = nodeData.find(parent => parent.id === item.parent);
          if (parentItem) {
            if (item.data.direction === 'OUT') {
              line.to = item.id;
              line.from = item.parent;
            } else if (item.data.direction === 'IN') {
              line.to = item.parent;
            }
          }
        }

        this.lines.push(line);
      });
    },
  generateRandomId(length) {
    let result = '';
    const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    const charactersLength = characters.length;
    for (let i = 0; i < length; i++) {
      result += characters.charAt(Math.floor(Math.random() * charactersLength));
    }
    return result;
},
  showSeeksGraph(graphRef, graph_json_data) {
    graphRef.setJsonData(graph_json_data, (graphInstance) => {
      // 这些写上当图谱初始化完成后需要执行的代码
      graphInstance.focusNodeById(this.originalData['id'].toString());

    });
  },

    onNodeClick(nodeObject, $event) {
     if(nodeObject.data.collection){
       this.listData = nodeObject.data.collection
       this.listTitle = nodeObject.text
       console.log('onNodeClick:', this.listData)

       this.listData.forEach(item =>{
         if(item.paths){
           item.open = false
         }
       })
     }
      const allLinks = this.$refs.graphRef.getLinks();
      allLinks.forEach(link => { // 还原所有样式
        link.relations.forEach(line => {
          line.color = "#c0c0c0";
          line.lineWidth = 2;
          line.className = ''
          line.disableDefaultClickEffect = true
        });
      });

      let fromNodeList = []
      let toNodeList = []
      this.getFromNode(allLinks,nodeObject,fromNodeList)
      this.getToNode(allLinks,nodeObject,toNodeList)
      // fromNodeList.forEach(item=>{
      //   console.log(item.fromNode.id + "->"+item.toNode.id)
      // })
      // toNodeList.forEach(item=>{
      //   console.log(item.fromNode.id + "->"+item.toNode.id)
      // })

      toNodeList.forEach(item=>{
        allLinks.filter(link => (link.fromNode === item.fromNode && link.toNode === item.toNode)).forEach(link => {
          console.log(link,'22222')
          link.relations.forEach(line => {
            line.color = "#148ccd"
            line.lineWidth = 3;
            line.className = 'all_height_light'
          });
        });
      })
      fromNodeList.forEach(item=>{
        allLinks.filter(link => (link.fromNode === item.fromNode && link.toNode === item.toNode)).forEach(link => {
          link.relations.forEach(line => {
            line.color = "#148ccd"
            line.lineWidth = 3;
            line.className = 'all_height_light'
          });
        });
      })


      // 让与{nodeObject}相关的所有连线高亮
      // allLinks.filter(link => (link.fromNode === nodeObject || link.toNode === nodeObject)).forEach(link => {
      //   console.log(link.fromNode)
      //   link.relations.forEach(line => {
      //     console.log('line:', line);
      //     line.data.orignColor = line.color;
      //     line.data.orignFontColor = line.fontColor || line.color;
      //     line.data.orignLineWidth = line.lineWidth || 1;
      //     line.lineWidth = 3;
      //   });
      // });
      // // 有时候更改一些属性后，并不能马上同步到视图，这需要以下方法让视图强制根据数据同步到最新
      this.$refs.graphRef.getInstance().dataUpdated();
    },
    onLineClick(lineObject, $event) {
      console.log('onLineClick:', lineObject)
      lineObject['className'] = ''
    },
    handelMouseenter(nodeObject,$event){
      this.onNodeClick(nodeObject,$event)
      //console.log(nodeObject,$event,'111111')
    },
    handelMouseleave(){
      const allLinks = this.$refs.graphRef.getLinks();
      allLinks.forEach(link => { // 还原所有样式
        link.relations.forEach(line => {
          line.color = "#c0c0c0";
          line.lineWidth = 2;
          line.className = ''
          line.disableDefaultClickEffect = true
        });
      });
    },
    getFromNode(allLinks, node, fromNodeList) {
      let linkList = allLinks.filter(link => (link.toNode === node));
      linkList.forEach(link => {
        fromNodeList.push({fromNode: link.fromNode, toNode: node});
      });
    },

    getToNode(allLinks, node, toNodeList) {
      let linkList = allLinks.filter(link => (link.fromNode === node));
      linkList.forEach(link => {
        toNodeList.push({fromNode: node, toNode: link.toNode});
      });
    },
    openDetail(item){
      this.listData.forEach(i =>{
        if(i.id === item.id){
          i.open = !i.open
        }
      })
      this.$forceUpdate()
    },
    openAll(){
      this.listData.forEach(i =>{
          i.open = !i.open
        if(i.open){
          this.openText = '全部收起'
        }else {
          this.openText = '全部展开'
        }
      })
      this.$forceUpdate()
    },
  }
}
</script>

<style  lang="less">
.relation-graph .rel-node-checked{
  box-shadow: none;
}
.relation-graph {
  //height: 300px!important;
}


.root{
  box-shadow: none !important;
  border-radius: 0 !important;
}
.node-body{
  height: auto;
  width: 300px;
  //background-color: red;
  display: flex;
  align-items: self-start;
  flex-direction: column;
  justify-content: center;
  border: 1px solid rgb(153, 153, 153);
  color: rgb(153, 153, 153);
    .node-body-item{
      display: flex;
      align-items: center;
      justify-content: flex-start;
      height: 50px;
      width: calc(100% - 10px);
      border-bottom: 1px solid rgb(153, 153, 153);
      padding-left: 10px;
      &:last-child{
        border-bottom: none;
      }
    }
}
.item-detail{
  height: 600px;
  width: 500px;
  background-color: #42b983;
  position: absolute;
  z-index: 100;
  top: 90px;
  right: 20px;
  box-shadow: rgba(0, 0, 0, 0.2) 0px 2px 4px 0px;
  .detail-header{
    height: 50px;
    width: 100%;
    background-color: #ffffff;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 15px;
    box-sizing: border-box;
    border-bottom: 1px solid #E5E5E5;
    span{
      font-size: 14px;
      color: #128bed;
      font-weight: 700;
    }
    .close{
      color: #128BED;
      font-size: 30px;
      line-height: 52px;
      float: right;
      cursor: pointer;
    }

  }
  .item-search{
    height: 66px;
    background-color: #ffffff;
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 15px;
    color: #666666;
    font-size: 14px;
    border-bottom: 1px solid #E5E5E5;
  }
  .item-list{
    height: 484px;
    width: 100%;
    background-color: #eeeeee;
    box-sizing: border-box;
    overflow-x: auto;
    .item-list-top{
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: flex-start;
      background-color: #ffffff;
      padding: 14px 15px;
      .item-list-avater{
        width: 30px;
        height: 30px;
        display: block;
        border-radius: 3px;
        float: left;
        border: 1px solid #eee;
        -o-object-fit: contain;
        object-fit: contain;
        text-align: center;
        line-height: 30px;
        background: #51a3d8;
        color: #fff;
      }
      .item-list-title{
        margin-left: 15px;
        font-size: 14px;
      }
      .item-list-open{
        margin-left: auto;
      }
    }
    .item-list-detail{

      box-sizing: border-box;
      //height: 90px;
      width: 100%;
      background-color: #F3F9FD;
      display: flex;
      align-items: center;
      justify-content: flex-start;
      padding: 14px 15px;
      .company-1{
        max-width: 90px;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: flex-start;
      }
      .link{
        box-sizing: border-box;
        height: 100%;
        width: 90px;
        //background-color: #42b983;
        padding: 0 10px;
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        line-height: 12px;
        color: #128bed;
        text-align: center;
        font-size: 12px;
        margin-bottom: 7px;

        .link-line{
          width: 100%;
          height: 2px;
          background-color: #ddd;
          margin: 5px 0;
        }
      }
    }

  }
}
</style>
<style scoped lang="less">

::v-deep .my-line-style-1{
  .c-rg-line-checked {
    animation: my-line-anm1 2s linear infinite;
  }
}
::v-deep .my-line-style-2{
  .c-rg-line-checked {
    animation: my-line-anm2 1s infinite;
  }
}
::v-deep .my-line-style-3{
  .c-rg-line-checked {
    animation: my-line-anm3 1s infinite;
  }
}
::v-deep .my-line-style-4{
  .c-rg-line-checked {
    animation: my-line-anm4 1s infinite;
  }
}
::v-deep .relation-graph {
  .all_height_light{
    animation: my-line-anm1 1s linear infinite !important;
  }
}

@keyframes my-line-anm1 {
  0% {
    stroke-dashoffset: 100px;
    stroke-dasharray: 5, 5, 5;
  }
  100% {
    stroke-dasharray: 5, 5, 5;
    stroke-dashoffset: 3px;
  }
}
@keyframes my-line-anm2 {
  from {
    stroke-dashoffset: 0;
    stroke-dasharray: 4,4,4;
  }
  to {
    stroke-dashoffset: 10px;
    stroke-dasharray: 20,20,20;
  }
}
@keyframes my-line-anm3 {
  0% { stroke-opacity: 1; }
  50% { stroke-opacity: 0.2; }
  100% { stroke-opacity: 1; }
}
@keyframes my-line-anm4 {
  0%{
    stroke-dasharray:0,100%;
  }
  100%{
    stroke-dasharray:100%,0;
  }
}
::v-deep .rel-link-peel>g>g{
  transform:  translate(-1000, -1127) rotate(0deg)!important;
}
</style>
