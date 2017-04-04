<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 50,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* 创意来自 https://github.com/jirengu-inc/animating-resume
* 大家好，我是杨君良
* 过不久就要毕业啦，你是不是也在准备简历呀。
* 那么，我也来写一份属于自己的简历吧！
*/

/* 首先给所有元素加上过渡效果 */
* {
    -webkit-transition: all 1s;
    transition: all 1s;
}
/* 白色背景太单调了，我们来点背景 */
html {
    color: rgb(222,222,222); 
    background: rgb(63,82,99); 
    font-size: 1.25em;
}
/* 文字离边框太近了 */
.styleEditor {
    padding: .5em;
    border: 1px solid;
    background: rgb(48, 48, 48);
    margin: .5em;
    overflow: auto;
    -webkit-transition: 1s;
    transition: 1s;
    width: 45vw; height: 85vh;
    box-shadow: -4px 4px 2px 0 rgba(0,0,0,0.3);
}

/* 鼠标移动到上面会发光哦 */
.styleEditor:hover{
    box-shadow: 0px 0px 40px 5px rgba(255,255,255,0.4);
}

/* 代码高亮一下吧 */
.token.selector{ color: #dd7d2e; }
.token.property{ color: #f62f2f; }
.token.punctuation{ color: yellow; }
.token.function{ color: #73aaff; }

/* 加点 3D 效果呗 */
html{
    -webkit-perspective: 1000px;
    perspective: 1000px;
}
.styleEditor {
    position: absolute; top: 0; 
    -webkit-transform: translateX(95%);
    transform-origin: right;
    -webkit-transform: rotateY(-10deg) translateX(105%);
    transform: rotateY(-10deg) translateX(105%);
    z-index:999;
}

/* 接下来我给自己准备一个编辑器 */
.resumeEditor{
    position: absolute; left: 0; top: 0;
    padding: .5em;  margin: .5em;
    width: 45vw; height: 85vh; 
    border: 1px solid;
    background: rgb(48, 48, 48); color: white;
    overflow: auto;
    transition: 1s;
    transform-origin: left;
    -webkit-transform: rotateY(10deg);
    transform: rotateY(10deg);
    box-shadow: -4px 4px 2px 0 rgba(0,0,0,0.3);
}
.resumeEditor:hover{
    box-shadow: 0px 0px 40px 5px rgba(255,255,255,0.4);
}
/* 好了，我开始写简历了 */


`,
          `
/* 这个简历好像差点什么
 * 对了，这是 Markdown 格式的，我需要变成对 HR 更友好的格式
 * 简单，用marked翻译成 HTML 就行了
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.resumeEditor{
    padding: 2em;
}
.resumeEditor h1{
    text-align: center;
}
.resumeEditor h2{
    display: inline-block;
    border-bottom: 1px solid;
    margin: 1em 0 .5em;
}
.resumeEditor ul{
    list-style: none;
    margin-left: 2em;
}
.resumeEditor ul> li::before{
    content: '•';
    margin-right: 1em;
}
/* 调整一下超链接的颜色 */
a{
    color: white;
}

/* 到这我的简历就写完了，是不是还不错呢？ */
`],
        currentMarkdown: '',
        fullMarkdown: `
杨君良
====

基本信息
----

* 男，22岁，信息管理与信息系统专业本科生，现在南京。

个人经历
----

* 大一：加入团学组织，学习，打杂
* 大二：开始参加数学建模竞赛，转专业参加江苏省大创，帮导师搞科研
* 大三：放弃考研，自学前端相关知识，希望能从事相关工作
* 大四：额。。还没有大四。。（噗嗤）

技能证书
----

* 前端基本知识：HTML+CSS+JS
* 库/框架相关：JQuery、Vue...
* 本科生普通证书：计算机二级/三级...

链接
----

* [GitHub个人主页](https://github.com/JJJJokeryang)

`
      }
    },

    //非data
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              console.log(prevChar)
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
  
  html {
    min-height: 100vh;
  }
  
  * {
    -webkit-transition: all .3s;
    transition: all .3s;
  }
</style>
