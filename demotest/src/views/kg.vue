<template>
  <div class="about">

    <el-row class="demo-autocomplete">
      <el-col>
        <h1 style="color: #FFFFFF">请输入您想搜索的关键词</h1>
        <!--    autocomplete 是一个可带输入建议的输入框组件，fetch-suggestions 是一个返回输入建议的方法属性，如 querySearch(queryString, cb)，
        在该方法中你可以在你的输入建议数据准备好时通过 cb(data) 返回到 autocomplete 组件中。-->
        <el-autocomplete
            class="inline-input"
            v-model="state1"
            :fetch-suggestions="querySearch"
            placeholder="请输入内容"
            suffix-icon="el-icon-search"
            @select="handleSelect"
            @keyup.enter.native="handleInput()"
        ></el-autocomplete>
      </el-col>
    </el-row>

    <!--    @change="handleSelect"-->
    <div ref="myPage" style="height:calc(100vh - 50px);"
         @click="show()">
      <SeeksRelationGraph
          ref="seeksRelationGraph"
          :options="graphOptions"
          :on-node-click="onNodeClick"
          :on-line-click="onLineClick">
        <div slot="node" slot-scope="{node}">
          <div style="height:64px;line-height: 64px;border-radius: 32px;cursor: pointer;"
               @contextmenu.prevent.stop="showNodeMenus(node, $event)">
            {{ node.text }}
          </div>
        </div>
      </SeeksRelationGraph>
    </div>

    <div v-if="isShowNodeMenuPanel"
         :style="{left: nodeMenuPanelPosition.x + 20+'px', top: nodeMenuPanelPosition.y +370+ 'px' }"
         style="z-index: 999;padding:10px;background-color: #ffffff;border:#eeeeee solid 1px;box-shadow: 0px 0px 8px #cccccc;position: absolute;">
      <div style="line-height: 25px;padding-left: 10px;color: #888888;font-size: 12px;">对这个节点进行操作：</div>
      <div class="c-node-menu-item" @click.stop="doAction1('查看操作')">查看节点信息</div>
      <div class="c-node-menu-item" @click.stop="doAction('更新操作')">编辑节点信息</div>
    </div>

    <div id = "ShowLinkMenu"
         :style="{left: linkMenuPanelPosition.x + 20+'px', top: linkMenuPanelPosition.y +370+ 'px' }"
         style="display: none;z-index: 999;padding:10px;background-color: #ffffff;border:#eeeeee solid 1px;box-shadow: 0px 0px 8px #cccccc;position: absolute;">
      <div style="line-height: 25px;padding-left: 10px;color: #888888;font-size: 12px;">对这条边进行操作：</div>
      <div class="c-node-menu-item" @click.stop="doAction1('查看操作')">查看</div>
      <div class="c-node-menu-item" @click.stop="doAction('更新操作')">编辑</div>
      <div class="c-node-menu-item" @click.stop="doAction2('合并操作')">合并两端节点</div>
    </div>

    <el-dialog title="详细信息" :visible.sync="dialogTableVisible">
      <el-table :data="gridData">
        <el-table-column property="id" label="id号"></el-table-column>
        <el-table-column property="text" label="名称/值"></el-table-column>
      </el-table>
    </el-dialog>

    <el-dialog title="修改信息" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="id号" :label-width="formLabelWidth">
          <el-input v-model="form.id" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="名称" :label-width="formLabelWidth">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog
        title="提示"
        :visible.sync="centerDialogVisible"
        width="30%"
        center>
      <span>节点合并成功！</span>
      <span slot="footer" class="dialog-footer">
    <el-button @click="centerDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="centerDialogVisible = false">确 定</el-button>
  </span>
    </el-dialog>


  </div>
</template>

<script>
import SeeksRelationGraph from 'relation-graph'

export default {
  name: 'Demo',
  components: {SeeksRelationGraph},
  data() {
    return {
      nodes: [],
      links: [],
      models: [],
      currentNodeId: 0,
      currentType: "",
      currentFrom: 0,
      currentTo: 1,
      state1: '',

      rootedId: 2,
      __graph_json_data: {},
      isShowCodePanel: false,
      isShowNodeMenuPanel: false,
      nodeMenuPanelPosition: {x: 0, y: 0},
      linkMenuPanelPosition: {x: 0, y: 0},
      graphOptions: {
        defaultNodeBorderWidth: 0,
        defaultNodeColor: 'rgba(238, 178, 94, 1)',
        allowShowMiniView: true,
        allowSwitchLineShape: true,
        allowSwitchJunctionPoint: true,
        isMoveByParentNode: true,
        defaultLineShape: 1,
        'layouts': [
          {
            'label': '自动布局',
            'layoutName': 'force',
            'layoutClassName': 'seeks-layout-force'
          }
        ],
        defaultJunctionPoint: 'border',
      },

      // gridData: [{
      //   date: '2016-05-02',
      //   name: '王小虎',
      //   address: '上海市普陀区金沙江路 1518 弄'
      // }, {
      //   date: '2016-05-04',
      //   name: '王小虎',
      //   address: '上海市普陀区金沙江路 1518 弄'
      // }, {
      //   date: '2016-05-01',
      //   name: '王小虎',
      //   address: '上海市普陀区金沙江路 1518 弄'
      // }, {
      //   date: '2016-05-03',
      //   name: '王小虎',
      //   address: '上海市普陀区金沙江路 1518 弄'
      // }],
      gridData: [],
      dialogTableVisible: false,
      dialogFormVisible: false,
      centerDialogVisible: false,
      form: {
        id: '',
        name: '',
        region: '',
        date1: '',
        date2: '',
        delivery: false,
        type: [],
        resource: '',
        desc: ''
      },
      formLabelWidth: '120px'
    };
  },
  methods: {
    querySearch(queryString, cb) {
      var nodes = this.nodes;
      var links = this.links;
      var models = [];
      // nodes.forEach(item => {
      //   item.value = item.text;
      // })
      for (var i = 0; i < nodes.length; i++) {
        var vote = {};
        vote.id = nodes[i].id;
        vote.value = nodes[i].text;
        vote.type = "node";
        models.push(vote);
      }
      for (var i = 0; i < links.length; i++) {
        var vote1 = {};
        vote1.id = links[i].id;
        vote1.value = links[i].text;
        vote1.type = "link";
        vote1.to = links[i].to;
        models.push(vote1);
      }
      this.models = models;
      var results = queryString ? models.filter(this.createFilter(queryString)) : models;
      console.log(results)
      // 调用 callback 返回建议列表的数据
      cb(results);
    },
    createFilter(queryString) {
      return (model) => {
        return (model.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
      };
    },
    loadAll() {
      return this.__graph_json_data.nodes;
    },
    loadAll1() {
      var lines = this.__graph_json_data.links
      var i = 1
      lines.forEach(item => {
        item.id = i;
        i++
      })
      return lines;
    },
    show()
    {
      this.isShowNodeMenuPanel =  false;
      //document.getElementById("ShowLinkMenu").style.display="none";
    },
    handleSelect(item) {
      if (item.type === "node")
        this.rootedId = item.id;
      else
        this.rootedId = item.to;
      this.$refs.seeksRelationGraph.focusNodeById(this.rootedId)

    },
    handleInput() {
      var flag = 0;
      console.log(this.state1)
      this.models.forEach(item => {
        if (item.value === this.state1) {
          if (item.type === "node")
            this.rootedId = item.id;
          else
            this.rootedId = item.to;
          this.$refs.seeksRelationGraph.focusNodeById(this.rootedId)
          flag = 1;
        }
      })
      if (flag === 0)
        alert("请输入有效关键词");
    },
    onNodeClick(nodeObject, $event) {
      // this.isShowNodeMenuPanel = true;
      console.log('onNodeClick:', nodeObject)
      // this.isShowLinkMenuPanel = true;
      this.$refs.seeksRelationGraph.focusNodeById(nodeObject.id)
    },
    onLineClick(lineObject, $event) {
      console.log('onLineClick:', lineObject)
      //this.isShowLinkMenuPanel = true;
      document.getElementById("ShowLinkMenu").style.display="inline";
      for (const key in lineObject) {
        console.log("是我啊" + key + ':' + lineObject[key]);
      }
      var _base_position1 = this.$refs.myPage.getBoundingClientRect()
      console.log('showNodeMenus:', $event, _base_position1)
      console.log("888888888888" + this.isShowLinkMenuPanel)
      this.currentFrom = lineObject.fromNode.id
      this.currentTo = lineObject.toNode.id
      this.currentType = "link"
      console.log("888888888888" + this.currentFrom + " " + this.currentTo)
      this.linkMenuPanelPosition.x = $event.clientX - _base_position1.x
      this.linkMenuPanelPosition.y = $event.clientY - _base_position1.y
      console.log("位置是：" + this.linkMenuPanelPosition.x + "  " + this.linkMenuPanelPosition.y)
    },
    showNodeMenus(nodeObject, $event) {
      this.currentNode = nodeObject
      for (var key in nodeObject) {
        console.log("是我啊" + key + ':' + nodeObject[key]);
      }
      this.currentNodeId = nodeObject.id
      this.currentType = "node"
      console.log("888888888888" + this.currentNode.id + " " + this.currentNode.text)
      var _base_position = this.$refs.myPage.getBoundingClientRect()
      console.log('showNodeMenus:', $event, _base_position)
      this.isShowNodeMenuPanel = true
      console.log("888888888888" + this.isShowNodeMenuPanel)
      this.nodeMenuPanelPosition.x = $event.clientX - _base_position.x
      this.nodeMenuPanelPosition.y = $event.clientY - _base_position.y
    },
    doAction(name) {
      this.dialogFormVisible = true
      this.isShowNodeMenuPanel = false
      document.getElementById("ShowLinkMenu").style.display="none";
    },
    doAction1(name) {
      this.dialogTableVisible = true
      console.log("888888888888" + this.currentNodeId)
      this.gridData = []
      if (this.currentType === "node") {
        var nodes = this.nodes
        for (var i = 0; i < nodes.length; i++) {
          {
            if (nodes[i].id === this.currentNodeId) {
              var vote = {};
              vote.id = nodes[i].id;
              vote.text = nodes[i].text;
              this.gridData.push(vote);
              break;
            }
          }
        }
      } else {
        var links = this.links;
        for (var i = 0; i < links.length; i++) {
          {
            if (links[i].from === this.currentFrom && links[i].to === this.currentTo) {
              var vote = {};
              vote.id = links[i].id;
              vote.text = links[i].text;
              this.gridData.push(vote);
              break;
            }
          }
        }
      }
      // this.$notify({
      //   title: '提示',
      //   message: '对节点【' + this.currentNode.text + '】进行了：' + actionName,
      //   type: 'success'
      // })
      this.isShowNodeMenuPanel = false
      document.getElementById("ShowLinkMenu").style.display="none";
    },
    doAction2(name) {
      this.isShowNodeMenuPanel = false
      document.getElementById("ShowLinkMenu").style.display="none";
      this.centerDialogVisible = true
    },
    getColorByRandom() {
      var colorList = ["#FFFF99", "#B5FF91", "#94DBFF", "#FFBAFF", "#FFBD9D", "#C7A3ED", "#CC9898", "#8AC007", "#CCC007", "#FFAD5C"];
      var colorIndex = Math.floor(Math.random() * colorList.length);
      var color = colorList[colorIndex];
      colorList.splice(colorIndex, 1);
      return color;
    },
    showSeeksGraph(query) {
      this.__graph_json_data = {
        rootId: this.rootedId,
        nodes: [
          // 注意：在节点配置信息中，你的自定义属性需要像下面这样放到data标签中，否则数据会丢失
          {id: '1', text: '节点-1', data: {myicon: 'el-icon-star-on'}},
          {id: '2', text: '节点-2', data: {myicon: 'el-icon-setting'}},
          {id: '3', text: '节点-3', data: {myicon: 'el-icon-setting'}},
          {id: '4', text: '节点-4', data: {myicon: 'el-icon-star-on'}},
          {id: '5', text: '节点-5', data: {myicon: 'el-icon-sunny'}},
          {id: '6', text: '节点-6', data: {myicon: 'el-icon-setting'}},
          {id: '7', text: '节点-7', data: {myicon: 'el-icon-setting'}},
          {id: '8', text: '节点-8', data: {myicon: 'el-icon-star-on'}},
          {id: '9', text: '节点-9', data: {myicon: 'el-icon-headset'}},
          {id: '71', text: '节点-71', data: {myicon: 'el-icon-headset'}},
          {id: '72', text: '节点-72', data: {myicon: 'el-icon-s-tools'}},
          {id: '73', text: '节点-73', data: {myicon: 'el-icon-star-on'}},
          {id: '81', text: '节点-81', data: {myicon: 'el-icon-s-promotion'}},
          {id: '82', text: '节点-82', data: {myicon: 'el-icon-s-promotion'}},
          {id: '83', text: '节点-83', data: {myicon: 'el-icon-star-on'}},
          {id: '84', text: '节点-84', data: {myicon: 'el-icon-s-promotion'}},
          {id: '85', text: '节点-85', data: {myicon: 'el-icon-sunny'}},
          {id: '91', text: '节点-91', data: {myicon: 'el-icon-sunny'}},
          {id: '92', text: '节点-82', data: {myicon: 'el-icon-sunny'}},
          {id: '51', text: '节点-51', data: {myicon: 'el-icon-sunny'}},
          {id: '52', text: '节点-52', data: {myicon: 'el-icon-sunny'}},
          {id: '53', text: '节点-53', data: {myicon: 'el-icon-sunny'}},
          {id: '54', text: '节点-54', data: {myicon: 'el-icon-sunny'}},
          {id: '55', text: '节点-55', data: {myicon: 'el-icon-sunny'}}
        ],
        links: [
          {from: '7', to: '71', text: '投资1', id: '1'},
          {from: '7', to: '72', text: '投资2', id: '2'},
          {from: '7', to: '73', text: '投资3', id: '3'},
          {from: '8', to: '81', text: '投资4', id: '4'},
          {from: '8', to: '82', text: '投资5', id: '5'},
          {from: '8', to: '83', text: '投资6', id: '6'},
          {from: '8', to: '84', text: '投资7', id: '7'},
          {from: '8', to: '85', text: '投资8', id: '8'},
          {from: '9', to: '91', text: '投资9', id: '9'},
          {from: '9', to: '92', text: '投资10', id: '10'},
          {from: '5', to: '51', text: '投资11', id: '11'},
          {from: '5', to: '52', text: '投资12', id: '12'},
          {from: '5', to: '53', text: '投资13', id: '13'},
          {from: '5', to: '54', text: '投资14', id: '14'},
          {from: '5', to: '55', text: '投资15', id: '15'},
          {from: '1', to: '2', text: '投资16', id: '16'},
          {from: '3', to: '1', text: '高管', id: '17'},
          {from: '4', to: '2', text: '高管1', id: '18'},
          {from: '6', to: '2', text: '高管2', id: '19'},
          {from: '7', to: '2', text: '高管3', id: '20'},
          {from: '8', to: '2', text: '高管4', id: '21'},
          {from: '9', to: '2', text: '高管5', id: '22'},
          {from: '1', to: '5', text: '投资16', id: '23'}
        ]
      }

      this.__graph_json_data.nodes.forEach(item => {
        item.color = this.getColorByRandom();
      })

      var __graph_json_data = this.__graph_json_data
      // 以上数据中的node和link可以参考"Node节点"和"Link关系"中的参数进行配置
      this.$refs.seeksRelationGraph.setJsonData(__graph_json_data, (seeksRGGraph) => {
        // Called when the relation-graph is completed
      })
    }
  },
  mounted() {
    this.showSeeksGraph()
    this.nodes = this.loadAll();
    this.links = this.loadAll1();
    console.log("2222222222" + this.isShowNodeMenuPanel)
    console.log("2222222222" + this.isShowLinkMenuPanel)
  }
}
</script>
<style>
.el-autocomplete {
  width: 500px;
  height: 200px;
}

.c-node-menu-item {
  line-height: 30px;
  padding-left: 10px;
  cursor: pointer;
  color: #444444;
  font-size: 14px;
  border-top: #efefef solid 1px;
}

.c-node-menu-item:hover {
  background-color: rgba(66, 187, 66, 0.2);
}

.el-dialog {
  line-height: 10px;
}
</style>