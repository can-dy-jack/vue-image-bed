<template>
  <div class="home">
    <form class="box">
        <h2>上传图片</h2>
        <fieldset class="img-loader-control">
          <legend>上传图片</legend>
          <div class="img-show" id="img-preview">
            <div class="info">点击下面按钮上传图片</div>
          </div>
          <div class="img-loader">
            <span><input name="file" type="file" id="img-input" multiple accept=".jpg,.jpeg,.png,.tif,.webp,.bmp"></span>
            <input type="button" value="删除">
            <input type="button" value="上传" @click="pushImg">
            <label for="img-input">选择图片</label>
          </div>
          <div class="access-token-input">
            <input type="text" placeholder="输入access-token" v-model="access_token" />
            <input type="text" placeholder="输入用户名" v-model="username" />
            <input type="text" placeholder="输入仓库名" v-model="userrepo" />
            <!-- <input type="text" placeholder="路径" v-model="path" /> -->
          </div>
        </fieldset>
    </form>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: 'HomeView',
  components: {
  },
  data(){
    return {
      fileTypes: ['image/jpeg','image/png'],
      username: '',
      userrepo: '',
      files: [],
      access_token: '',
    }
  },
  computed:{
    name: function(){
      return this.username.trim();
    },
    repo: function(){
      return this.userrepo.trim();
    },
    auth: function(){
      return this.access_token.trim();
    }
  },
  methods: {
    uploadImg: function(){
      let self = this;
      const input = document.querySelector('#img-input');
      const preview = document.querySelector('#img-preview');
      // 监测文件上传 - input值改变
      input.addEventListener('change',updateImageDisplay);
      function updateImageDisplay(){
        // 清空之前留下的 preview 图片
        while(preview.firstChild){
          preview.removeChild(preview.firstChild);
        }
        // 获取包含所有已选择图片的 FileList 对象，
        const curImages = input.files;
        self.files = curImages;
        // 检查是否没有图片被选择
        if(curImages.length === 0){
          const previewInfo = document.createElement('div');
          previewInfo.className = "info";
          previewInfo.textContent = "未选择图片！";
          previewInfo.style.color = 'red';
          preview.appendChild(previewInfo);
        } else {
          // 如果正确选择图片
          for(const file of curImages){
            const imgItem = document.createElement('div');
            imgItem.className = "img-show-item";
            const text = document.createElement("div");
            text.className = "text";
            const imgBox = document.createElement("div");
            imgBox.className = "img-box"
            const img = document.createElement("img");
            if(self.fileTypes.includes(file.type)){
              const name = document.createElement("div");
              const size = document.createElement("div");
              size.textContent = `(${self.returnFileSize(file.size)})`;
              imgBox.appendChild(img);
              img.src = URL.createObjectURL(file);
              name.textContent = file.name;
              name.className = "text-name";
              text.appendChild(name);
              text.appendChild(size);
            } else {
              text.textContent = `File name ${file.name}: Not a valid file type. Update your selection.`;
            }
            imgItem.appendChild(imgBox);
            imgItem.appendChild(text);
            preview.appendChild(imgItem);
          }
        }
      }
    },
    returnFileSize: function(number) {
      if(number < 1024) {
        return number + 'bytes';
      } else if(number >= 1024 && number < 1048576) {
        return (number/1024).toFixed(1) + 'KB';
      } else if(number >= 1048576) {
        return (number/1048576).toFixed(1) + 'MB';
      }
    },
    pushImg: function(){
      let self = this;
      let reader = new FileReader();
      let xhr = new XMLHttpRequest();
      xhr.withCredentials = true;
      xhr.addEventListener("readystatechange", function() {
        if(this.readyState === 4) {
          console.log(this.responseText);
        }
      });

      xhr.open("PUT", "https://api.github.com/repos/can-dy-jack/vue-img/contents/img");
      xhr.setRequestHeader("accept", "application/vnd.github.v3+json");
      xhr.setRequestHeader("Authorization", "token " + self.access_token);
      xhr.setRequestHeader("Content-Type", "application/json");

      reader.onload = function(evt) {
        xhr.send({
          "message": "img-test1",
          "content": evt.target.result
        });
      };
      reader.readAsBinaryString(self.files[0]);
    }
  },
  mounted() {
    this.uploadImg();
  }
}
</script>

<style lang="scss">
  .home {
    height: 90vh;
    padding-top: 20px;
    .box {
      width: 80%;
      margin: 10px auto;
      h2 {
        margin: 20px 0;
        border-bottom: 1px solid #42b983;
      }
      .img-loader-control {
        border: 1px solid #42b983;
        padding: 10px;
        .img-show {
          min-height: 220px;
          width: 100%;
          background: #fafafa;
          border: 1px dashed #42b983;
          display: flex;
          flex-wrap: wrap;
          justify-content: space-evenly;
          padding: 10px;
          box-sizing: border-box;
          .info {
            width: 100%;
            text-align: center;
            padding-top: 100px;
          }
          .img-show-item {
            height: 220px;
            width: 200px;
            margin: 10px;
            box-shadow: 0 4px 12px rgba(0,0,0,.14);
            display: flex;
            flex-direction: column;
            background: #fff;
            .img-box {
              height: 170px;
              flex: 1;
              margin: 10px auto;
              img {
                max-width: 180px;
                max-height: 150px;
              }
            }
            .text {
              height: 50px;
              box-sizing: border-box;
              width: 200px;
              padding: 5px;
              text-align: center;
              .text-name {
                width: 190px;
                white-space:nowrap; 
                overflow: hidden;
                text-overflow: ellipsis;
              }
            }
            img {
              max-width: 180px;
            }
          }
        }
        .img-loader {
          display: flex;
          justify-content: flex-end;
          padding: 5px 0;
          &>input {
            cursor: pointer;
            border: none;outline: none;
            background: #42b983;
            color: white;
            width: 80px;
            height: 30px;
            line-height: 30px;
            text-align: center;
            transition: all 0.25s ease;
            &:hover {
              background: green;
            }
            &:active {
              box-shadow: 0 0 0 2px rgba(66, 185, 131, .5);
            }
          }
          input:disabled {
            cursor: default;
            &:hover {
              background: #42b983;
            }
            &:active {
              box-shadow: none;
            }
          }
          span {
            // input[type="file"] {
            // }
            flex: 1;
            height: 30px;
            line-height: 30px;
          }
          label {
            cursor: pointer;
            background: #42b983;
            color: white;
            z-index: 10;
            width: 80px;
            height: 30px;
            line-height: 30px;
            text-align: center;
            transition: all 0.25s ease;
            &:hover {
              background: green;
            }
            &:active {
              box-shadow: 0 0 0 2px rgba(66, 185, 131, .5);
            }
          }
        }
      }
    }
  }
  @media screen and (max-width: 800px){
    .box {
      width: 90%;
    }
  }
  .access-token-input {
    display: flex;
    box-sizing: border-box;
    padding: 4px 0;
    input {
      flex: 1;
      margin-right: 3px;
      padding: 4px 6px;
      font-size: 16px;
      outline: none;
      border: 1px dashed #42b983;
      transition: all 0.25s ease;
      &::placeholder{
        color: #aaa;
      }
      &:focus{
        border-style: solid;
        box-shadow: 0 0 0 2px rgba(19, 214, 94, 0.3);
      }
    }
  }
</style>
