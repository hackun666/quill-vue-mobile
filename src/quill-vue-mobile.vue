<template>
  <div class="editor-wrapper">
    <div class="flex1">
  <div style="position:relative;padding:0 10px">
    <textarea
      class="title"
      id="textarea"
      maxlength="40"
      v-model.trim="title"
      @input="titleChange"
      type="text"
      placeholder="请输入标题"
    ></textarea>
    <span style="position:absolute;right:0;bottom:0;font-size:14px;color:#555" v-if="warning"><span :class="warning?'warning':''">超出限制 {{textLength-40}} 字符</span></span>
  </div>

    <div
      id="editor"
    >
    </div>
    </div>
    <div id="toolbar">
      <div style="flex:1"></div>
      <button
        class="ql-font"
        :class="fontShow?'active':''"
        @click="fontShow=!fontShow"
      >
        <i class="iconfont icon-ziti"></i>
        <div
          class="pop"
          v-show="fontShow"
        >
          <button class="ql-bold">
            <i class="iconfont icon-cuti"></i>
          </button>
          <button class="ql-italic">
            <i class="iconfont icon--xieti"></i>
          </button>
          <button class="ql-header">
            <i class="iconfont icon-biaotizhengwenqiehuan"></i>
          </button>
          <button class="ql-underline">
            <i class="iconfont icon-xiahuaxian"></i>
          </button>
          <button class="ql-strike">
            <i class="iconfont icon-shanchuxian"></i>
          </button>
        </div>
      </button>
      <button
        class="ql-font"
        :class="layoutShow?'active':''"
        @click="layoutShow=!layoutShow"
      >
        <i class="iconfont icon-buju"></i>
        <div
          class="pop"
          v-show="layoutShow"
        >
          <button
            class="ql-list"
            value="ordered"
          >
            <i class="iconfont icon-youxuliebiao"></i>
          </button>
          <button
            class="ql-list"
            value="bullet"
          >
            <i class="iconfont icon-wuxuliebiao"></i>
          </button>
          <button
            class="ql-align"
            value="justify"
          >
            <i class="iconfont icon-zuoduiqi"></i>
          </button>
          <button
            class="ql-align"
            value="right"
          >
            <i class="iconfont icon-youduiqi"></i>
          </button>
          <button
            class="ql-align"
            value="center"
          >
            <i class="iconfont icon-juzhongduiqi"></i>
          </button>
        </div>
      </button>

      <button @click="uploadImg">
        <i class="iconfont icon-tupian"></i>
      </button>
      <button @click="hr">
        <i class="iconfont icon-jianxianxing"></i>
      </button>

      <button
        @click="undo"
        :disabled="!undoBool"
      >
        <i class="iconfont icon-7chexiao"></i>
      </button>
      <button
        @click="redo"
        :disabled="!redoBool"
      >
        <i class="iconfont icon-fanchexiao"></i>
      </button>
    </div>
  </div>
</template>
<script>
import Quill from 'quill'
import 'quill/dist/quill.core.css'
function strlen (str) {
  var len = 0
  for (var i = 0; i < str.length; i++) {
    var c = str.charCodeAt(i)
    // 单字节加1
    if ((c >= 0x0001 && c <= 0x007e) || (c <= 0xff9f && c >= 0xff60)) {
      len++
    } else {
      len += 2
      // len++
    }
  }
  return len
}
/* function makeExpandingArea (el) {
  var timer = null
  // 由于ie8有溢出堆栈问题，故调整了这里
  var setStyle = function (el, auto) {
    if (auto) el.style.height = 'auto'
    el.style.height = el.scrollHeight + 'px'
  }
  var delayedResize = function (el) {
    if (timer) {
      clearTimeout(timer)
      timer = null
    }
    timer = setTimeout(function () {
      setStyle(el)
    }, 200)
  }
  if (el.addEventListener) {
    el.addEventListener('input', function () {
      setStyle(el)
    }, false)
    setStyle(el)
  } else if (el.attachEvent) {
    el.attachEvent('onpropertychange', function () {
      setStyle(el)
    })
    setStyle(el)
  }
  if (window.VBArray && window.addEventListener) { // IE9
    el.attachEvent('onkeydown', function () {
      var key = window.event.keyCode
      if (key === 8 || key === 46) delayedResize(el)
    })
    el.attachEvent('oncut', function () {
      delayedResize(el)
    }) // 处理粘贴
  }
} */

export default {
  name: 'quill-vue-mobile',
  props: {
    // vh: {
    //   type: [Number, String],
    //   default: window.innerHeight
    // },
    // kh: {
    //   type: [Number, String],
    //   default: 0
    // },
    defaultValue: {
      type: Object,
      default: () => {
        return {
          title: '',
          content: ''
        }
      }
    }
  },
  data () {
    return {
      title: this.defaultValue.title,
      content: this.defaultValue.content,
      quill: null,
      undoBool: false,
      redoBool: false,
      fontShow: false,
      layoutShow: false,
      textLength: 0,
      warning: false
    }
  },
  watch: {
    defaultValue: {
      handler (val) {
        this.title = val.title || ''
        this.content = val.content || ''
        this.quill.root.innerHTML = this.content
        this.titleChange()
        this.$emit('content-change', this.content)
      },
      deep: true
    },
    title: {
      handler (val) {
        this.textLength = strlen(val)
        if (this.textLength > 40) {
          this.warning = true
        } else {
          this.warning = false
        }
      }
    }
  },
  methods: {
    titleChange () {
      this.$emit('title-change', this.title)
      this.$emit('input', this.title)
    },
    renderImg (url) {
      const range = this.quill.getSelection(true)
      this.quill.insertEmbed(range.index, 'image', url, Quill.sources.USER)
      this.$nextTick(() => {
        this.quill.insertText(range.index + 1, '\n', Quill.sources.USER)
        this.$nextTick(() => {
          this.quill.setSelection(range.index + 2, Quill.sources.SILENT)
        })
      })
    },
    fn3 () {
      const BlockEmbed = Quill.import('blots/block/embed')

      class DividerBlot extends BlockEmbed { }
      DividerBlot.blotName = 'divider'
      DividerBlot.tagName = 'hr'
      Quill.register(DividerBlot)
      this.quill = new Quill('#editor', {
        placeholder: '请输入正文',
        modules: {
          toolbar: '#toolbar'
        }
      })
      this.$emit('init', this.quill)
      this.quill.root.innerHTML = this.content
      this.quill.on('editor-change', (eventName, ...args) => {
        this.undoBool = this.quill.history.stack.undo.length > 0
        this.redoBool = this.quill.history.stack.redo.length > 0
        this.$emit('content-change', this.quill.container.firstChild.innerHTML)
        this.titleChange()
      })
    },
    undo () {
      this.quill.history.undo()
    },
    redo () {
      this.quill.history.redo()
    },
    hr () {
      const range = this.quill.getSelection(true)
      this.quill.insertText(range.index, '\n', Quill.sources.USER)
      this.quill.insertEmbed(
        range.index + 1,
        'divider',
        true,
        Quill.sources.USER
      )
      this.quill.setSelection(range.index + 2, Quill.sources.SILENT)
    },
    uploadImg () {
      this.$emit('upload-img')
    }
  },
  mounted () {
    // var textarea = document.getElementById('textarea')
    // makeExpandingArea(textarea)
    this.fn3()
    document.addEventListener(
      'click',
      e => {
        const cln = e.toElement.className
        if (
          cln === 'iconfont icon-ziti' ||
          cln === 'iconfont icon-cuti' ||
          cln === 'iconfont icon--xieti' ||
          cln === 'iconfont icon-biaotizhengwenqiehuan' ||
          cln === 'iconfont icon-xiahuaxian' ||
          cln === 'iconfont icon-shanchuxian'
        ) {
          this.fontShow = true
          this.layoutShow = false
          return false
        } else if (
          cln === 'iconfont icon-youxuliebiao' ||
          cln === 'iconfont icon-wuxuliebiao' ||
          cln === 'iconfont icon-zuoduiqi' ||
          cln === 'iconfont icon-youduiqi' ||
          cln === 'iconfont icon-juzhongduiqi' ||
          cln === 'iconfont icon-buju'
        ) {
          this.layoutShow = true
          this.fontShow = false
          return false
        }
        this.fontShow = false
        this.layoutShow = false
      },
      false
    )
  }
}
</script>
<style lang="less">
@import '//at.alicdn.com/t/font_1652649_qu42x05j4s.css';
.ql-active {
  .iconfont {
    color: #3fbdf0;
  }
}
.ql-editor.ql-blank::before{
  font-style: normal;
}
.flex1{
  flex: 1;
  overflow-y: scroll;
  margin-bottom: 40px;
  height: 100%;
}
.warning{
  color: red;
}
.expandingArea.active pre {
  display: block;
  visibility: hidden;
}
.expandingArea.active pre span {
  display: block;
  visibility: hidden;
}
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow: hidden;
  // overflow: hidden;
  // height: 100%;
}
.ql-container {
  overflow-y: hidden !important ;
  // height: calc(100% - 90px);
  font-size: 24px;
  height: auto!important;
}
.editor-wrapper {
  height: 100%;
  display: flex;
  flex-direction: column;
}
.ql-editor {
  padding-bottom: 40px;
}
html,
body,
#app,
.editor-wrapper {
  // height: 100vh;
}
.title {
  color: #333;
  border: none;
  width: 100%;
  border-bottom: 1px solid #eee;
  outline: none;
  padding: 0;
  font-size: 36px;
  overflow: hidden;
  min-height: 40px;
  max-height: 80px;
  height: 80px;
  font-weight: 500;
  height: 100%;
  resize: none;
  line-height: 40px;
}
#toolbar {
  display: flex;
  position: fixed;
  bottom:0;
  width: 100%;
  background: #f9f9f9;
  z-index: 99;
  height: 40px;
}
#editor {
  box-sizing: border-box;
  border: none;
  // flex: 1;
  // height: 100%;
  // padding-bottom: 40px;
  .ql-editor {
    outline: none;
    // overflow-y: scroll;
  }
  .ql-clipboard {
    outline: none;
    height: 0;
  }
  img {
    max-width: 90%;
  }
}
button {
  position: relative;
  background: transparent;
  border: 1px solid #f9f9f9;
  cursor: pointer;
  display: inline-block;
  font-size: 18px;
  padding: 0;
  height: 40px;
  width: 45px;
  text-align: center;
  .iconfont {
    color: #444;
    padding: 10px;
  }
  &.active {
    > .iconfont {
      color: #3fbdf0;
    }
  }
  .pop {
    position: absolute;
    top: -40px;
    background: #f9f9f9;
    display: flex;
    flex-wrap: nowrap;
    box-shadow: 0px 0px 6px rgba(0, 0, 0, 0.3);
    border-radius: 4px;
    overflow: hidden;
    z-index: 9;
  }
}
button[disabled] {
  .iconfont {
    color: #bbb;
  }
}
button:active,
button:focus {
  outline: none;
}
</style>
