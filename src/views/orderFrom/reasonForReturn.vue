<script lang="ts" setup>
import {onMounted, reactive, ref} from 'vue'
import {
	addOrderSalesReturn,
	orderSalesReturn,
	removeOrderSalesReturn,
	changeSalesReturn,
	editSalesReturn
} from "@/api/orderApi";
import {useLoginStore} from "@/stores/loginStore"
import {ElMessage} from "element-plus";

let pageSize = ref(5)
let total = ref(10)
let currentPage = ref(1)
const isShow = ref(false)
// 添加商品数据
const form = reactive({
	title: '',
	open: false,
})

const editTitle = ref('')

// 对话框开关
const isOpenMask = ref(false)
const editOpenMask = ref(false)
let data = reactive<Array<any>>([{}])

let tableData = reactive<Array<any>>([{}])

// 改变状态的params数据
let obj = reactive<object>({
	token: useLoginStore().get(),
	id: ''
})

const list = () => {
	orderSalesReturn().then(res => {
		console.log(res)
		// @ts-ignore
		data = res.data
		data.forEach(item => {
			item.time = TimestampToDate(item.create_at)
			item.value = item.isAvailable === 1
		})
		total.value = data.length
		isShow.value = true
		handleCurrentChange(1)
	}).catch(err => {
		console.log(err)
	})
}

onMounted(() => {
	list()
})

// 添加退货原因
const addReason = () => {

	let data = {
		token: useLoginStore().get(),
		title: '',
		isAvaila: 0
	}
	data.title = form.title
	data.isAvaila = form.open ? 1 : 0
	if (form.title.trim() == '') {
		ElMessage({
			message: "请输入退货原因",
			type: 'error'
		})
		return
	}
	addOrderSalesReturn({...data}).then(res => {
		isOpenMask.value = false
		console.log(res)
		isShow.value = false
		list()
		ElMessage({
			// @ts-ignore
			message: res.msg,
			type: 'success',
		})
	}).catch(err => {
		console.log(err)
	})
}

// 删除数据
const removeReason = (value) => {
	obj.id = value
	removeOrderSalesReturn({...obj}).then((res: any) => {
		let type = res.msg != '删除成功' ? 'error' : 'success'
		ElMessage({
			// @ts-ignore
			message: res.msg,
			type: type,
		})
		list()
	}).catch(err => {
		console.log(err)
	})
}

// 退货原因状态改变
const changeSales = (value) => {
	obj.id = value
	changeSalesReturn({...obj}).then(res => {
		console.log(res)
		ElMessage({
			// @ts-ignore
			message: res.msg,
			type: 'success',
		})
	}).catch(err => {
		console.log(err)
	})
}

let edidId = ref('')
const OpenMask = (value, title) => {
	edidId.value = value
	editOpenMask.value = true
	editTitle.value = title
}
// 编辑退货原因
const editReason = () => {
	let obj = reactive<object>({
		token: useLoginStore().get(),
		name: editTitle,
		id: edidId
	})
	editSalesReturn({...obj}).then(res => {
		console.log(res)
		ElMessage({
			// @ts-ignore
			message: res.msg,
			type: 'success',
		})
		list()
		editOpenMask.value = false
		isShow.value = false
		editTitle.value = ''
	}).catch(err => {
		console.log(err)
	})

}

// 时间戳转换函数
function TimestampToDate(Timestamp: number) {
	let date = new Date(Timestamp * 1000);
	let y = date.getFullYear();
	let m = date.getMonth() + 1;
	let d = date.getDate();
	return y + "-" + (m < 10 ? "0" + m : m) + "-" + (d < 10 ? "0" + d : d) + " " + date.toTimeString().substr(0, 8);
}

const handleCurrentChange = (val: Number) => {
	tableData = [...data.slice((currentPage.value - 1) * pageSize.value, currentPage.value * pageSize.value)]
}

const handleSizeChange = (val: Number) => {
	tableData = [...data.slice((currentPage.value - 1) * pageSize.value, currentPage.value * pageSize.value)]
	handleCurrentChange(1)
}
</script>

<template>
	<div class="box">
		<!-- 面包屑导航栏	-->
		<el-breadcrumb class="breadcrrumb" separator="/">
			<el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item :to="{path:'/orderlist'}">订单</el-breadcrumb-item>
			<el-breadcrumb-item>退货原因</el-breadcrumb-item>
		</el-breadcrumb>
		<el-divider/>
		<el-dialog v-model="isOpenMask" title="添加你的商品">
			<el-form :model="form">
				<el-form-item label="原因类型" label-width="100px">
					<el-input v-model="form.title" autocomplete="off"/>
				</el-form-item>
				<el-form-item label="是否启用" label-width="100px">
					<el-switch v-model="form.open"/>
				</el-form-item>
			</el-form>
			<template #footer>
      <span class="dialog-footer">
        <el-button @click="isOpenMask = false">取消</el-button>
        <el-button type="primary" @click="addReason">
          确定
        </el-button>
      </span>
			</template>
		</el-dialog>
		<el-dialog v-model="editOpenMask" title="编辑你的商品">
			<el-form-item label="原因类型" label-width="100px">
				<el-input v-model="editTitle" autocomplete="off"/>
			</el-form-item>
			<template #footer>
      <span class="dialog-footer">
        <el-button @click="isOpenMask = false">取消</el-button>
        <el-button type="primary" @click="editReason">确定</el-button>
      </span>
			</template>
		</el-dialog>
		<!-- 数据列表	-->
		<div class="dataList">
			<div>
				<el-icon>
					<List/>
				</el-icon>
				<span>&nbsp;数据列表</span>
			</div>
			<el-button style="width: 80px" @click="isOpenMask = true">添加</el-button>

		</div>
		<!-- 数据	-->
		<div v-if="isShow" class="data-box">
			<el-table :data="tableData" border max-height="580" style="width: 100%; text-align: center;">
				<el-table-column align="center" fixed prop="date" type="selection" width="50"/>
				<el-table-column align="center" label="编号" prop="_id" width="60"/>
				<el-table-column align="center" label="原因类型" prop="name" width="200"/>
				<el-table-column align="center" label="排序" prop="isAvailable" width="170"/>
				<el-table-column align="center" label="是否可用" prop="value">
					<template #default="scope">
						<el-switch v-model="scope.row.value" @change="changeSales(scope.row._id)"/>
					</template>
				</el-table-column>
				<el-table-column align="center" label="添加时间" prop="time"/>
				<el-table-column align="center" label="操作" width="220">
					<template #default="scope">
						<el-button @click="OpenMask(scope.row._id,scope.row.name)">编辑</el-button>

						<el-button @click="removeReason(scope.row._id)">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
		</div>
		<div class="pagination">
			<el-pagination
				v-model:current-page="currentPage"
				v-model:page-size="pageSize"
				:page-sizes="[5, 10, 20, 100]"
				:total="total"
				background
				class="el-pagination" layout="total, sizes, prev, pager, next"
				@current-change="handleCurrentChange"
				@size-change="handleSizeChange"
			/>
		</div>
	</div>
</template>

<style lang="scss" scoped>
.data-box {
	margin-top: 20px;
}

.pagination {
	position: absolute;
	right: 20px;
	bottom: 20px;
	margin-top: 10px;
}

.dataList {
	width: calc(100% - 20px);
	padding: 20px 10px;
	display: flex;
	justify-content: space-between;
	align-items: center;
	border: 1px solid #eee;
	margin: 20px 0;
}
</style>