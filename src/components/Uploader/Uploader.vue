<template>
  <div :class="[prefixCls + '-upload']" :id="'upload-' + uploadId">
    <div v-if="uploadType == 'click'">
      <label>
        <input
          type="file"
          :name="name"
          :accept="accept"
          :id="uploadId"
          :multiple="multiple"
          @change="onChange($event)" />
        <slot>
          <span :class="[prefixCls + '-btn', prefixCls + '-btn-tertiary']">点击上传</span>
        </slot>
      </label>
      <div :class="[prefixCls + '-upload-list']">
        <div :class="[prefixCls + '-upload-item']" v-for="(file, index) in fileList">
          <div :class="[prefixCls + '-upload-item-info', current == index && (prefixCls + '-active')]"
               @mouseover="filemouseover(index)"
               @mouseout="filemouseout">
            <icon type="doc"
                  :class="[prefixCls + '-upload-file-icon']"
                  size="12"></icon>
            <span>{{file.name}}</span>
            <icon type="close"
                  :class="[prefixCls + '-upload-del-info']"
                  size="12"
                  @click="delExistFile(index)"></icon>
          </div>
        </div>
        <div :class="[prefixCls + '-upload-item']" v-for="(file, index) in uploadList">
          <div :class="[prefixCls + '-upload-item-info', current == index && (prefixCls + '-active')]"
               @mouseover="filemouseover(index)"
               @mouseout="filemouseout">
            <icon type="doc"
                  :class="[prefixCls + '-upload-file-icon']"
                  size="12"></icon>
            <span>{{file.name}}</span>
            <icon type="close"
                  :class="[prefixCls + '-upload-del-info']"
                  size="12"
                  @click="delFile(index)"></icon>
          </div>
          <div :class="[prefixCls + '-upload-item-progress', progress[index] == '100%' && (prefixCls + '-hide')]">
            <div :class="[prefixCls + '-upload-progress', prefixCls + '-upload-progress-line', prefixCls + '-upload-progress-status-success']">
              <div :class="[prefixCls + '-upload-progress-inner']">
                <div :class="[prefixCls + '-upload-progress-bg']" :style="{width: progress[index]}"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-if="uploadType == 'drag'"
         :class="[prefixCls + '-upload-drag']">
      <div :class="[prefixCls + '-upload-drag-container', dragover && (prefixCls + '-is-dragover')]">
        <input type="file"
               :name="name"
               :id="uploadId"
               :accept="accept"
               :multiple="multiple"
               @change="onChange($event)" />
        <label :for="uploadId"
               :class="[prefixCls + '-upload-drag-area']">
          <p :class="[prefixCls + '-upload-drag-icon']"></p>
          <span v-if="advanceDrag">点击或将文件拖拽到此区域上传</span>
          <span v-if="!advanceDrag">当前环境不支持拖拽上传，请点此上传</span>
          <p v-if="fileList.length > 0" v-for="(file, index) in selectFiles">{{file}}</p>
        </label>
      </div>
      <div :class="[prefixCls + '-upload-list']">
        <div :class="[prefixCls + '-upload-item']" v-for="(file, index) in fileList">
          <div :class="[prefixCls + '-upload-item-info', current == index && (prefixCls + '-active')]"
               @mouseover="filemouseover(index)"
               @mouseout="filemouseout">
            <icon type="doc"
                  :class="[prefixCls + '-upload-file-icon']"
                  size="12"></icon>
            <span>{{file.name}}</span>
            <icon type="close"
                  class="[prefixCls + '-upload-del-info']"
                  size="12"
                  @click="delExistFile(index)"></icon>
          </div>
        </div>
        <div :class="[prefixCls + '-upload-item']" v-for="(file, index) in uploadList">
          <div :class="[prefixCls + '-upload-item-info', current == index && (prefixCls + '-active')]"
               @mouseover="filemouseover(index)"
               @mouseout="filemouseout">
            <icon type="doc"
                  :class="[prefixCls + '-upload-file-icon']"
                  size="12"></icon>
            <span>{{file.name}}</span>
            <icon type="close"
                  :class="[prefixCls + '-upload-del-info']"
                  size="12"
                  @click="delFile(index)"></icon>
          </div>
          <div :class="[prefixCls + '-upload-item-progress', progress[index] == '100%' && (prefixCls + '-hide')]">
            <div :class="[prefixCls + '-upload-progress', prefixCls + '-upload-progress-line', prefixCls + '-upload-progress-status-success']">
              <div :class="[prefixCls + '-upload-progress-inner']">
                <div :class="[prefixCls + '-upload-progress-bg']" :style="{width: progress[index]}"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  import Icon from '../Icon'
  import Message from '../Message'
  export default {
    name: 'Uploader',
    props: {
      id: {
        type: String,
        default: ''
      },
      name: {
        type: String,
        default: 'files'
      },
      uploadType: {
        type: String,
        default: 'click' // drag拖拽上传，click点击上传
      },
      accept: {
        type: String,
        default: ''
      },
      url: {
        type: String,
        default: ''
      },
      multiple: {
        type: Boolean,
        default: true
      },
      fileList: {
        default: null
      },
      maxlength: Number,
      prefixCls: {
        type: String,
        default: 'atui'
      }
    },
    data () {
      return {
        value: '',
        uploadId: this.id || 'upload' + (new Date()).getTime(),
        model: null,
        current: -1,
        percent: 0,
        xhr: 'FormData' in window,
        uploadList: [],
        progress: [],
        dragover: false
      }
    },
    computed: {
      advanceDrag () {
        const div = document.createElement('div')
        return (('draggable' in div) || ('ondragstart' in div && 'ondrop' in div)) && 'FormData' in window && 'FileReader' in window
      }
    },
    components: {
      Icon,
      Message
    },
    mounted () {
      this._input = document.querySelector('#' + this.uploadId)
      this.$el = document.querySelector('#upload-' + this.uploadId)

      this.advanceDrag && this.addDragEvt()
    },
    beforeDestroy () {
      let events = ['drag', 'dragstart', 'dragend', 'dragleave', 'drop', 'dragover', 'dragenter']
      events.forEach((event) => {
        this.$el.removeEventListener(event, () => this._eventHandler())
      })
    },
    methods: {
      onChange (e) {
        let files = e.target.files

        if (files) {
          for (let i in files) {
            if (typeof (files[i]) === 'object' && files[i].name) {
              this.progress.push('0%')
              this.uploadList.push(files[i])
            }
          }
        } else {
          // 只会取上传的第一个，只能上传一个
          this.progress = ['0%']
          this.uploadList = [{name: this._input.value.replace(/^.*\\/, '')}]
        }

        if (this.maxlength && this.fileList.length + this.uploadList.length > this.maxlength) {
          this._input.value = ''
          this.uploadList = []
          this.showMessage('超过上传数量限制，请先删除再进行上传')
        } else {
          this.submitForm()
        }
      },

      /**
       * 上传后自动提交
       */
      submitForm () {
        if (this.uploadList.length > 0) {
          if (this.url) {
            if (this.xhr) {
              this.xhrUpload()
            } else {
              this.iframeUpload()
            }
          }
        }
      },

      xhrUpload () {
        let self = this
        let data = new window.FormData()
        let i = 0
        let len = this.uploadList.length

        for (i = 0; i < len; i++) {
          (function (i, file) {
            if (file.type.match(self.accept)) {
              data.append(self.name, file, file.name)

              let xhr = new window.XMLHttpRequest()

              xhr.open('post', self.url, true)

              xhr.onload = () => {
                self.parseResponse(xhr.responseText, i)
              }

              xhr.upload.onprogress = (e) => {
                const loaded = e.loaded ? e.loaded : 0
                const total = e.total ? e.total : 1
                self.progress[i] = parseInt((loaded / total) * 100, 10) + '%'
              }

              xhr.onerror = () => {
                self.setError('上传失败了！')
              }

              xhr.send(data)
            }
          })(i, this.uploadList[i])
        }
      },

      /**
       * body中插入form和iframe进行上传
       */
      iframeUpload () {
        let i = 0
        let len = this.uploadList.length
        if (this.testSameOrigin(this.url)) {
          for (i = 0; i < len; i++) {
            let iframeName = 'uploadiframe-' + i + '-' + new Date().getTime()
            let iframe = document.createElement('iframe')
            let form = document.createElement('form')
            let input = document.createElement('input')

            input.setAttribute('type', 'file')
            input.setAttribute('value', this.uploadList[i].name)
            iframe.setAttribute('name', iframeName)
            iframe.style.display = 'none'
            form.setAttribute('method', 'post')
            form.setAttribute('action', this.url)
            form.setAttribute('target', iframeName)
            form.setAttribute('data-index', i)

            document.body.appendChild(form)
            form.appendChild(iframe)
            form.appendChild(input)

            iframe.addEventListener('load', () => {
              this.parseResponse(iframe.contentDocument.body.innerHTML, form.getAttribute('data-id'))
              document.body.removeChild(form)
            })
            form.submit()
          }
        } else {
          this.setError('iframe不支持跨域请求')
        }
      },

      /**
       * 测试上传地址与当前页面地址是否同域
       */
      testSameOrigin (url) {
        const loc = window.location
        const a = document.createElement('a')
        a.href = url
        return a.hostname === loc.hostname &&
               a.port === loc.port &&
               a.protocol === loc.protocol
      },

      /**
       * 处理响应结果
       */
      parseResponse (response, index) {
        let data = null

        if (!response) {
          this.setError('服务器没有响应', index)
        } else {
          try {
            data = JSON.parse(response)
          } catch (e) {
            this.setError('服务器响应数据格式有问题', index)
          }

          if (data) {
            if (data.success) {
              this.model = data.data
              this.$dispatch('completed::file-upload', {
                model: this.model,
                file: this.uploadList[index]
              })
            } else if (data.error) {
              this.setError(data.error, index)
            }
          }
        }
      },

      /**
       * 设置错误提示
       */
      setError (message, index) {
        this.errorMessage = message
        this.$dispatch('completed::file-upload', {
          error: this.errorMessage,
          file: index && this.uploadList[index] || null
        })

        index > -1 && this.uploadList.splice(index, 1)
      },

      showMessage (msg) {
        Message.success(msg)
      },

      filemouseover (index) {
        this.current = index
      },

      filemouseout () {
        this.current = -1
      },

      delFile (index) {
        this.$dispatch('delete::file-upload', {
          file: this.uploadList[index]
        })
        this.uploadList.splice(index, 1)
      },

      delExistFile (index) {
        this.$dispatch('delete::file-upload', {
          file: this.fileList[index]
        })
        this.fileList.splice(index, 1)
      },

      /**
       * 添加drag事件
       */
      addDragEvt () {
        let events = ['drag', 'dragstart', 'dragend', 'dragleave', 'drop', 'dragover', 'dragenter']

        events.forEach((event) => {
          this.$el.addEventListener(event, (e) => this.dragHandler(e))
        })
      },

      /**
       * drag事件处理
       */
      dragHandler (e) {
        let self = this
        e.preventDefault()
        e.stopPropagation()

        if (e.type === 'dragover' || e.type === 'dragenter') {
          self.dragover = true
        }

        if (e.type === 'dragend' || e.type === 'dragleave' || e.type === 'drop') {
          self.dragover = false
          if (e.type === 'drop') {
            let files = e.dataTransfer.files || {}
            for (let i in files) {
              if (files[i] && files[i].name) {
                self.progress.push('0%')
                self.uploadList.push(files[i])
              }
            }
            self.submitForm()
          }
        }
      }
    }
  }
  </script>
