<template>
  <div>
    <div class="container">
      <!--      表格上方的搜索部分信息-->
      <div class="handle-box">
        <el-input v-model="query.username" placeholder="用户名" class="handle-input mr10"></el-input>
        <el-button type="primary" :icon="Search" @click="handleSearch">搜索</el-button>
        <el-button type="primary" :icon="Plus" @click="handlerAddUser">新增用户</el-button>
      </div>
      <el-table :data="tableData" class="table" ref="multipleTable">
        <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
        <el-table-column prop="username" label="用户名" width="120"></el-table-column>
        <el-table-column prop="age" label="年龄" width="55"></el-table-column>
        <el-table-column prop="nickname" label="昵称" width="200"></el-table-column>
        <el-table-column prop="mail" label="邮箱" width="200"></el-table-column>
        <el-table-column prop="updateTime" label="更新时间" width="170"></el-table-column>
        <el-table-column prop="userDesc" label="用户描述" width="200"></el-table-column>
        <el-table-column label="操作" align="center" width="290">
          <template #default="scope">
            <el-button text :icon="Edit" @click="handleEdit(scope.$index, scope.row)" v-permiss="15">
              编辑
            </el-button>
            <el-button text :icon="Delete" class="red" @click="handleDelete(scope.$index, scope.row)" v-permiss="16">
              删除
            </el-button>
            <el-button text>
              禁用
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
            background
            layout="total, prev, pager, next"
            :current-page="query.page"
            :page-size="query.size"
            :total="pageTotal"
            @current-change="handlePageChange"
        ></el-pagination>
      </div>
    </div>

    <!-- 编辑弹出框 -->
    <el-dialog title="编辑" v-model="editVisible" width="30%">
      <el-form label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="form.username"></el-input>
        </el-form-item>
        <el-form-item label="昵称">
          <el-input v-model="form.nickname"></el-input>
        </el-form-item>
        <el-form-item label="邮件">
          <el-input v-model="form.mail"></el-input>
        </el-form-item>
        <el-form-item label="用户描述">
          <el-input v-model="form.userDesc" type="textarea" />
        </el-form-item>
      </el-form>
      <template #footer>
				<span class="dialog-footer">
					<el-button @click="editVisible = false">取 消</el-button>
					<el-button type="primary" @click="saveEdit">确 定</el-button>
				</span>
      </template>
    </el-dialog>

    <!-- 新增弹出框 -->
    <el-dialog title="新增用户" v-model=" addVisible " width="30%">
      <el-form label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="昵称">
          <el-input v-model="addForm.nickname"></el-input>
        </el-form-item>
        <el-form-item label="邮件">
          <el-input v-model="addForm.mail"></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
				<span class="dialog-footer">
					<el-button @click="addVisible = false">取 消</el-button>
					<el-button type="primary" @click="saveAddUser">确 定</el-button>
				</span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts" name="basetable">
import {ref, reactive} from 'vue';
import {ElMessage, ElMessageBox} from 'element-plus';
import {handlerUserSearch, handlerUserEdit} from '../../api/manage';
import { Search, Plus, Edit, Delete } from '@element-plus/icons-vue';

interface TableItem {
  id: number;
  username: string;
  age: number;
  mail: string;
  nickname: string;
  userDesc: string;
}

const query = reactive({
  username: '',
  page: 1,
  size: 10
});
const tableData = ref<TableItem[]>([]);
const pageTotal = ref(0);
// 获取表格数据
const getData = () => {
  let queryStr = 'page=' + query.page + '&size=' + query.size;
  if (query.username) {
    queryStr = queryStr + '&username=' + query.username
  }
  handlerUserSearch(queryStr).then(res => {
    if (res.data.code == '200') {
      tableData.value = res.data.data.records;
      pageTotal.value = res.data.data.total || 50;
    }
  });
};
getData();
// 查询操作
const handleSearch = () => {
  query.page = 1;
  query.size = 10;
  getData();
  ElMessage.success(`查询成功`);

};

// 处理用户新增弹窗
const handlerAddUser = () => {
  addVisible.value = true
};
//处理新增后台用户数据
const saveAddUser = () => {
  ElMessageBox.confirm('确定要新增用户吗？', '提示', {
    type: 'warning'
  }).then(() => {
    console.log('点击确认')

  }).catch(() => {
    console.log('点击其他')

  });

}

// 分页导航
const handlePageChange = (val: number) => {
  query.page = val;
  getData();
};

// 删除操作
const handleDelete = (index: number, data: any) => {
  // 二次确认删除
  ElMessageBox.confirm('确定要删除吗？', '提示', {
    type: 'warning'
  }).then(() => {
    console.log(data.id)
    // ElMessage.success('删除成功');
    // tableData.value.splice(index, 1);
  })
      .catch(() => {
      });
};

// 表格编辑时弹窗和保存
const editVisible = ref(false);
// 表格新增的弹窗控制
const addVisible = ref(false);
let form = reactive({
  username: '',
  nickname: '',
  mail: '',
  userDesc: '',
  id: ''
});
let addForm = reactive({
  username: '',
  nickname: '',
  mail: '',
});
let idx: number = -1;
//处理编辑的情况
const handleEdit = (index: number, row: any) => {
  idx = index;
  form.id = row.id
  form.username = row.username;
  form.nickname = row.nickname;
  form.userDesc = row.userDesc;
  form.mail = row.mail;
  editVisible.value = true;
};
// 保存使用调用的方法
const saveEdit = () => {
  //取消对话框修改
  editVisible.value = false;
  const data = {}
  handlerUserEdit(JSON.stringify(form)).then(res => {
    if (res.data.code == '200') {
      ElMessage.success(`修改第 ${idx + 1} 行成功`);
      getData();
    }
  })
};

const saveAdd = () => {
  //取消对话框修改
  addVisible.value = false;
  const data = {}
  handlerUserEdit(JSON.stringify(form)).then(res => {
    if (res.data.code == '200') {
      ElMessage.success(`修改第 ${idx + 1} 行成功`);
      getData();
    }
  })
};


</script>

<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
}

.table {
  width: 100%;
  font-size: 14px;
}

.red {
  color: #F56C6C;
}

.mr10 {
  margin-right: 10px;
}

.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>
