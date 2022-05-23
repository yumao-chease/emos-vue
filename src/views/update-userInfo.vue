<template>
	<el-dialog title="个人信息" v-model="visible" width="25%">
		<el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
			<!--<el-form-item label="头像" prop="tel">-->
			<!--	<el-input type="tel" v-model="dataForm.tel" size="medium" clearable />-->
			<!--</el-form-item>-->
			<el-form-item label="电话" prop="tel">
				<el-input v-model="dataForm.tel" size="medium" clearable />
			</el-form-item>
			<el-form-item label="邮箱" prop="email">
				<el-input v-model="dataForm.email" size="medium" clearable />
			</el-form-item>
		</el-form>
		<template #footer>
			<span class="dialog-footer">
				<el-button size="medium" @click="visible = false">取消</el-button>
				<el-button type="primary" size="medium" @click="dataFormSubmit">确定</el-button>
			</span>
		</template>
	</el-dialog>
</template>

<script>
export default {
	data() {
		const validateConfirmPassword = (rule, value, callback) => {
			if (value != this.dataForm.email) {
				callback(new Error('两次输入的密码不一致'));
			} else {
				callback();
			}
		};

		return {
			visible: false,
			dataForm: {
				tel: '',
				email: ''
			},
			dataRule: {
        tel: [{ required: true, pattern: '^1\\d{10}$', message: '电话格式错误' }],
        email: [
          {
            required: true,
            pattern: '^([a-zA-Z]|[0-9])(\\w|\\-)+@[a-zA-Z0-9]+\\.([a-zA-Z]{2,4})$',
            message: '邮箱格式错误'
          }
        ]
			}
		};
	},
	methods: {
		init(){
			this.visible = true;
			this.$nextTick(() => {
				// 清除缓存
				this.$refs['dataForm'].resetFields();
			});
		},
		dataFormSubmit: function(){
			let that = this;
			that.$refs['dataForm'].validate(valid => {
				if(valid){
					let data = {tel: that.dataForm.tel,email: that.dataForm.email};
					that.$http('user/updateUserInfo','POST',data,true,resp => {
						if(resp.rows == 1){
							that.$message({
								message: '修改信息成功',
								type: 'success',
								duration: 1200
							});
						}else{
							that.$message({
								message: '信息修改失败',
								type: 'error',
								duration: 1200
							});
						}
					})
					that.visible = false;
				}
			})
		}
	}
};
</script>

<style></style>
