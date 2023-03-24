<template>
  <view class="content">
    <image class="logo" src="/static/logo.png"></image>
    <view class="text-area">
      <!-- <text class="title">{{title}}</text> -->
    </view>
    <!-- <u-button style="margin-bottom: 30px" type="primary" text="按钮" @click="create"></u-button> -->
    <u-upload
      :fileList="fileList1"
      @afterRead="afterRead"
      @delete="deletePic"
      name="1"
      multiple
      :maxCount="10"
    ></u-upload>
    </u-upload>
  </view>
</template>

<script>
	export default {
		data() {
			return {
				fileList1: [],
			}
		},
		methods:{
			// 删除图片
			deletePic(event) {
				this[`fileList${event.name}`].splice(event.index, 1)
			},
			// 新增图片
			async afterRead(event) {
				// 当设置 multiple 为 true 时, file 为数组格式，否则为对象格式
				let lists = [].concat(event.file)
				let fileListLen = this[`fileList${event.name}`].length
				lists.map((item) => {
					this[`fileList${event.name}`].push({
						...item,
						status: 'uploading',
						message: '上传中'
					})
				})
				for (let i = 0; i < lists.length; i++) {
					const result = await this.uploadFilePromise(lists[i].url)
					let item = this[`fileList${event.name}`][fileListLen]
					this[`fileList${event.name}`].splice(fileListLen, 1, Object.assign(item, {
						status: 'success',
						message: '',
						url: result
					}))
					fileListLen++
				}
			},
			uploadFilePromise(url) {
				return new Promise((resolve, reject) => {
					let a = uni.uploadFile({
						url: 'http://www.mjorkj.top:8088/user/updateImages', // 仅为示例，非真实的接口地址
						// url: 'http://pro.yanhuiyun.suiyiyun.cn/api/v1/common/Common/normalfileupload?type=imgs', // 仅为示例，非真实的接口地址
						filePath: url,
						name: 'filedata',
						// header: {
						// 	'Content-Type': 'multipart/form-data'
						// },
						// formData: {
						// 	filedata: url
						// },
						success: (res) => {
							console.log("参数",url);
							console.log('上传成功',res);
							setTimeout(() => {
								resolve(res.data.data)
							}, 1000)
						}
					});
				})
			},
		}

	}
</script>

<style>
  .album {
    @include flex;
    align-items: flex-start;

    &__avatar {
      background-color: $u-bg-color;
      padding: 5px;
      border-radius: 3px;
    }

    &__content {
      margin-left: 10px;
      flex: 1;
    }
  }

  .content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .logo {
    height: 200rpx;
    width: 200rpx;
    margin-top: 200rpx;
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 50rpx;
  }

  .text-area {
    display: flex;
    justify-content: center;
  }

  .title {
    font-size: 36rpx;
    color: #8f8f94;
  }
</style>
