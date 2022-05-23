<template>
	<el-dialog title="执行归档" width="500px" :close-on-click-modal="false" v-model="visible" :show-close="false">
		<el-upload
			ref="upload"
			:action="url"
			list-type="picture-card"
			accept=".jpg,.jpeg,.png"
			with-credentials="true"
			:before-upload="beforeUploadHandle"
			:on-success="successHandle"
			:on-remove="removeHandle"
		>
			<i class="el-icon-plus"></i>
		</el-upload>
		<template #footer>
			<span class="dialog-footer">
				<el-button size="medium" @click="cancel()">取消</el-button>
				<el-button type="primary" @click="archive()" size="medium" :disabled="disableBtn">{{ btn }}</el-button>
			</span>
		</template>
	</el-dialog>
</template>

<script>
export default {
	data: function() {
		return {
			visible: false,
			url: this.$baseUrl + 'cos/uploadCosFile?type=archive',
			taskId: '',
			picList: {},
			disableBtn: false,
			btn: '确定'
		};
	},
	methods: {
    init: function(taskId) {
      let that = this;
      that.visible = true;
      that.taskId = taskId;
      that.btn = '执行归档';
      that.disableBtn = false;
      that.$nextTick(() => {
        that.$refs['upload'].clearFiles();
      });
    },
    beforeUploadHandle: function(file) {
      if (file.type !== 'image/jpg' && file.type !== 'image/jpeg' && file.type !== 'image/png') {
        this.$message.error('只支持jpg、png格式的图片！');
        return false;
      }
      return true;
    },
    successHandle: function(resp, file, fileList) {
      if (resp && resp.code === 200) {
        for (let one of fileList) {
          this.picList[one.response.url] = one.response.path;
        }
      } else {
        this.$message.error('图片上传失败');
      }
    },
    removeHandle: function(file, fileList) {
      let that = this;
      let url = file.response.url;
      let path = this.picList[url];
      that.$http('cos/deleteCosFile', 'POST', { pathes: [path] }, true, function(resp) {
        delete that.picList[url];
      });
    },
    cancel: function() {
      let that = this;
      if (Object.keys(that.picList).length > 0) {
        let pathes = Object.values(that.picList);
        that.$http('cos/deleteCosFile', 'POST', { pathes: pathes }, true, function(resp) {
          that.picList = {};
        });
      }
      that.visible = false;
      that.$refs['upload'].clearFiles();
    },
    archive: function() {
      let that = this;
      //禁用归档按钮，防止归档中用户重复点击归档按钮
      that.btn = '正在归档';
      that.disableBtn = true;
      if (Object.keys(that.picList).length == 0) {
        that.$message({
          message: '没有上传归档照片',
          type: 'error',
          duration: 1200
        });
        return;
      }
      let files = [];
      for (let key in that.picList) {
        files.push({
          url: key,
          path: that.picList[key]
        });
      }
      let data = {
        taskId: that.taskId,
        files: JSON.stringify(files)
      };
      that.$http(`approval/archiveTask`, 'POST', data, true, function (resp) {
        that.$message({
          message: '操作成功',
          type: 'success',
          duration: 1200
        });
        that.visible = false;
        that.$emit('refreshDataList');
      });
    },
    expand: function(row, expandedRows) {
      let that = this;
      //判断是否展开了折叠面板
      if (expandedRows.length>0) {
        that.expands = [];  //关闭所有的折叠面板
        if (row) {
          that.expands.push(row.taskId); //展开你想展开的折叠面板
          let data = {
            instanceId: row.processId,
            type: row.type,
            status: row.status
          };
          that.$http('approval/searchApprovalContent', 'POST', data, false, function(resp) {
            let content = resp.content;
            that.content = content;
            //如果返回工作流实例存在files，说明工作流实例绑定了归档文件
            if (content.hasOwnProperty('files')) {
              for (let one of content.files) {
                that.archiveList.push(one.url);
              }
            }
          });
          that.bpmnUrl =
              that.$baseUrl +
              'approval/searchApprovalBpmn' +
              '?instanceId=' +
              row.processId +
              '&time=' +
              new Date().getTime();
          that.bpmnList = [that.bpmnUrl];
        }
      } else {
        that.expands = []; // 默认不展开
      }
    },

  }
};
</script>

<style lang="less"></style>
