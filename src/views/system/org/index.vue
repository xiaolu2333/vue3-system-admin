<template>
  <el-form :model="queryParams" ref="queryForm" :inline="true">
    <el-form-item label="机构名称" prop="name">
      <el-input
          v-model="queryParams.name"
          placeholder="请输入机构名称"
          clearable
          style="width: 200px"
          @change="handleQueryNameOptionsChange"
      />
    </el-form-item>
    <el-form-item>
      <el-button
          type="primary"
          icon="Search"
          @click="handleQuery"
      >搜索
      </el-button>
      <el-button
          icon="Refresh"
          @click="resetQuery"
      >重置
      </el-button>
    </el-form-item>
  </el-form>

  <el-card>
    <el-button
        type="primary"
        plain
        icon="Plus"
        @click="handleAdd"
    >新增
    </el-button>
    <el-table
        :data="orgTree"
        :loading="loading"
        :row-style="rowStyle"
        row-key="id"
        border
        default-expand-all
        :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
        @cell-click="handleNodeClick"
    >
      <el-table-column prop="name" label="Name"/>
      <el-table-column prop="id" label="Id"/>
      <el-table-column prop="parentId" label="ParentId"/>
      <el-table-column prop="desc" label="Desc"/>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="success" @click="handleUpdate(scope.row)">修改</el-button>
          <!--          <el-button v-if="scope.row.parentId != 0" link type="danger" @click="handleDelete(scope.row)">删除</el-button>-->
        </template>
      </el-table-column>
    </el-table>

    <!-- 新增或编辑机构对话框 -->
    <el-dialog v-model="dialogVisible" :title="orgFormTitle" draggable>
      <el-form :model="orgForm" ref="mainForm" size="large">
        <el-form-item label="name" prop="name">
          <el-input v-model="orgForm.name"/>
        </el-form-item>
        <el-form-item label="code" prop="code">
          <el-input v-model="orgForm.code"/>
        </el-form-item>
        <el-form-item label="status" prop="status">
          <el-select v-model="orgForm.status" placeholder="请选择状态">
            <el-option label="启用" value="1"/>
            <el-option label="禁用" value="0"/>
          </el-select>
        </el-form-item>
        <el-form-item label="desc" prop="desc">
          <el-input v-model="orgForm.desc"/>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button type="primary" @click="submitForm">Submit</el-button>
          <el-button @click="dialogVisible = false">Cancel</el-button>
        </span>
      </template>
    </el-dialog>
  </el-card>

</template>
<script setup>
import {getCurrentInstance, onMounted, reactive, toRefs} from "vue";
import {ElMessage} from "element-plus";

import {GetOrgTreeByPost, SaveOrg} from "@/api/system/org";

const {proxy} = getCurrentInstance();

const state = reactive({
  loading: false,
  parentNode: {},
  currentNode: {},
  orgTree: [],
//   orgTree: [
//   {
//     id: 1,
//     name: '机构1',
//     code: 'JG1',
//     status: 0,
//     desc: "1111111111",
//     parentId: "root",
//     children: [
//       {
//         id: 2,
//         name: '机构2',
//         code: 'JG2',
//         status: 1,
//         desc: "1111111111",
//         parentId: 1,
//         children: [
//           {
//             id: 3,
//             name: '机构3',
//             code: 'JG3',
//             status: 0,
//             desc: "1111111111",
//             parentId: 2,
//             children: [
//               {
//                 id: 4,
//                 name: '机构4',
//                 code: 'JG4',
//                 status: 1,
//                 desc: "1111111111",
//                 parentId: 3,
//                 children: []
//               },
//               {
//                 id: 5,
//                 name: '机构5',
//                 code: 'JG5',
//                 status: 0,
//                 desc: "1111111111",
//                 parentId: 3,
//                 children: []
//               }
//             ]
//           },
//           {
//             id: 6,
//             name: '机构6',
//             code: 'JG6',
//             status: 1,
//             desc: "1111111111",
//             parentId: 2,
//             children: []
//           }
//         ]
//       },
//       {
//         id: 7,
//         name: '机构7',
//         code: 'JG7',
//         status: 0,
//         desc: "1111111111",
//         parentId: 1,
//         children: [
//           {
//             id: 8,
//             name: '机构8',
//             code: 'JG8',
//             status: 1,
//             desc: "1111111111",
//             parentId: 7,
//             children: []
//           }
//         ]
//       }
//     ]
//   },
//   {
//     id: 9,
//     name: '机构9',
//     code: 'JG9',
//     status: 0,
//     desc: "1111111111",
//     parentId: "root",
//     children: [
//       {
//         id: 10,
//         name: '机构10',
//         code: 'JG10',
//         status: 1,
//         desc: "1111111111",
//         parentId: 9,
//         children: []
//       }
//     ]
//   },
//   {
//     id: 11,
//     name: '机构11',
//     code: 'JG11',
//     status: 1,
//     desc: "1111111111",
//     parentId: "root",
//     children: []
//   },
// ],
  queryParams: {},
  dialogVisible: false,
  orgForm: {},
  orgFormTitle: "",
})

const {
  loading,
  orgTree,
  parentNode,
  currentNode,
  queryParams,
  dialogVisible,
  orgForm,
  orgFormTitle
} = toRefs(state);

/**
 * 查询机构
 * */
function handleQuery() {
  GetOrgTreeByPost().then(response => {
    orgTree.value = response.data.data;
  })
}

function resetQuery() {
  console.log("resetQuery");
}

/**
 * 重置暂存数据
 * */
function resetForm() {
  console.log("resetForm");
  state.orgForm = {};
  state.parentNode = {};
  state.currentNode = {};
}

/**
 * 新增机构
 * */
function handleAdd(val) {
  // console.log("新增机构:", val);
  dialogVisible.value = true;
  orgFormTitle.value = "新增机构";

  // 填充基础表单
  orgForm.value = {
    // 随机整数ID
    id: Math.floor(Math.random() * 1000000),
    parentId: currentNode.value?.id ? currentNode.value.id : "root",
  }
  console.log("新增机构基础表单:", orgForm.value);
}

/**
 * 修改机构
 * */
function handleUpdate(row) {
  console.log("handleUpdate", row);
  // 去除children属性
  delete row.children;
  orgForm.value = row;

  dialogVisible.value = true;
  orgFormTitle.value = "修改机构";

  console.log("新增机构基础表单:", orgForm.value);
  // 然后将当前行数据赋值给表单
  orgForm.value = row;
}

/**
 * 提交表单
 * */
function submitForm() {
  // orgForm
  console.log("orgForm:", orgForm);
  SaveOrg(orgForm.value).then(response => {
    console.log("response:", response);
    if (response.data.code === 200) {
      ElMessage.success("操作成功！");
      dialogVisible.value = false;
      handleQuery();
      resetForm();
    } else {
      ElMessage.error(response.data.msg);
      resetForm();
    }
  })
}

// 机构名称输入框 change事件
function handleQueryNameOptionsChange() {
  console.log("handleQueryNameOptionsChange");
}

// 节点点击事件
function handleNodeClick(row) {
  currentNode.value = row;
  parentNode.value = getParentNode(state.orgTree, currentNode.value.id);
  console.log("当前节点:", currentNode.value);
  console.log("父节点:", parentNode.value);
}

// 高亮被点击行
function rowStyle({row}) {
  if (currentNode.value === row) {
    return 'background: #85bff9; color: #aa2626; cursor: pointer';
  }
}

// 这里还有问题：后半部分的节点会被遍历到，但是不会被赋值给parentObj，导致parentObj为null
// 获取被点击行的父级节点
function getParentNode(treeData, id) {
  let parentObj = null;

  function traverse(treeNode, targetId) {
    if (treeNode.id === targetId) {
      return;
    }

    if (treeNode.children && treeNode.children.length > 0) {
      for (let i = 0; i < treeNode.children.length; i++) {
        let child = treeNode.children[i];

        if (child.id === targetId) {
          parentObj = treeNode;
          return;
        }

        traverse(child, targetId);
      }
    }
  }

  for (let i = 0; i < treeData.length; i++) {
    traverse(treeData[i], id);

    if (parentNode) {
      break;
    }
  }

  if (!parentObj) {
    parentObj = {
      id: "root",
      name: "根节点",
      code: "root",
      status: 1,
      desc: "根节点",
      parentId: "root",
    }
    return parentObj;
  }
  return parentObj;
}

// 测试
function test() {
  // 根据id获取父级对象
  function findParentNode(tree, id) {
    let parentNode = null;

    function traverse(node, targetId) {
      if (node.id === targetId) {
        return;
      }

      if (node.children && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          let child = node.children[i];

          if (child.id === targetId) {
            parentNode = node;
            return;
          }

          traverse(child, targetId);
        }
      }
    }

    for (let i = 0; i < tree.length; i++) {
      traverse(tree[i], id);

      if (parentNode) {
        break;
      }
    }

    return parentNode;
  }

  const data = [
    {
      id: 1,
      name: "A省环保局",
      parentId: "root",
      orderNum: 1,
      children: [
        {
          id: 2,
          name: "A市环保局",
          parentId: 1,
          orderNum: 1,
          children: [
            {
              id: 3,
              name: "a1县环保局",
              parentId: 2,
              orderNum: 1,
              children: [
                {
                  id: 4,
                  name: "a1乡环保局",
                  parentId: 3,
                  orderNum: 1,
                  children: []
                },
                {
                  id: 11,
                  name: "a1乡环保局2",
                  parentId: 3,
                  orderNum: 2,
                  children: []
                }
              ],
            },
            {
              id: 5,
              name: "a2县环保局",
              parentId: 2,
              orderNum: 2,
              children: []
            }
          ]
        },
        {
          id: 6,
          name: "B市环保局",
          parentId: 1,
          orderNum: 2,
          children: [
            {
              id: 7,
              name: "b1县环保局",
              parentId: 6,
              orderNum: 1,
              children: []
            }
          ]
        }
      ]
    },
    {
      id: 8,
      name: "B省环保局",
      parentId: "root",
      orderNum: 2,
      children: [
        {
          id: 9,
          name: "C市环保局",
          parentId: 8,
          orderNum: 1,
          children: [
            {
              id: 12,
              name: "c1县环保局",
              parentId: 9,
              orderNum: 1,
              children: []
            }
          ]
        }
      ]
    },
    {
      id: 10,
      name: "C省环保局",
      parentId: "root",
      orderNum: 3,
      children: []
    }
  ];

  const parentNode = findParentNode(data, 12);
  console.log(parentNode);
}

onMounted(() => {
  test();
  handleQuery();
  console.log("proxy", proxy);
});
</script>

<style scoped>
.current {
  background-color: #409EFF;
  color: #e6f7ff;
}
</style>