<template>
  <div>
    <div style="height: 100vh" :class="['my-line-style-2']">
      <RelationGraph ref="graphRef" :options="graphOptions" :on-node-click="onNodeClick" :on-line-click="onLineClick">
        <template #node="{node}">
          <div >
            <div class="node-body" @mouseenter="handelMouseenter" @mouseleave="handelMouseleave">
                <div class="node-body-item" :style="[{backgroundColor:(node.data.direction==='IN'?'rgb(255, 96, 96)':'rgb(18, 139, 237)')},{color:'#ffffff'}]">{{node.text}}</div>
               <div class="node-body-item" v-for="i in node.data.collection" :key="i.id">
                 {{i.nodeName}}
               </div>
            </div>
            <span class="operation" :style="[{color:(node.data.direction==='IN'?'rgb(255, 96, 96)':'rgb(18, 139, 237)')}]">{{node.data.operation}}</span>
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
  components: { RelationGraph },
  data() {
    return {
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
        defaultJunctionPoint: 'tb'
      },
    }
  },
  mounted() {
    this.showGraph()
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


      console.log(jsonData.nodes,'2222')
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
                    collection: item.collection.length > 4 ? item.collection.slice(0, 3) : item.collection,
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
      // data.forEach((item,index) => {
      //
      //     if (item.parent === null) {
      //       if (item.id !== this.originalData['id'].toString()) {
      //         this.lines.push({
      //           from: (item.id).toString(),
      //           to: (this.originalData['id']).toString(),
      //           // text: item.text,
      //           lineShape: 4
      //         });
      //       }
      //     } else {
      //       this.lines.push({
      //         from: (item.id).toString(),
      //         to: (item.parent).toString(),
      //         // text: item.text,
      //         lineShape: 4
      //       });
      //     }
      // });
      nodeData.forEach(item => {
        const line = {
          from: item.id,
          to: '',
          // text: item.operation,
          lineShape: 4
        };

        if (item.parent === null) {
          line.to = this.originalData['id'].toString();
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
      console.log('onNodeClick:', nodeObject)
    },
    onLineClick(lineObject, $event) {
      console.log('onLineClick:', lineObject)
    },
    handelMouseenter(nodeObject){

      console.log('onNodeClick:', nodeObject)
    },
    handelMouseleave(){

    }
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
//.operation::after {
//  content: '';
//  display: block;
//  width: 0;
//  height: 0;
//  border-left: 10px solid transparent; /* 左边透明 */
//  border-right: 10px solid transparent; /* 右边透明 */
//  border-bottom: 10px solid black; /* 底部为实心 */
//}
::v-deep .my-line-style-2{
  .c-rg-line-checked {
    animation: my-line-anm2 1s infinite;
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
</style>
