<template>
  <div class="app-container home">
    <el-row :gutter="20">
      <el-col :sm="24" :lg="24">

      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :sm="24" :lg="12" style="padding-left: 20px">
        <h2>ChatGPTAI智能问答系统<span>v{{ version }}</span></h2>
        <p>
          全面智能交互，实现高效沟通：ChatGPTAI智能问答系统是一个全新的对话系统，可以在微信小程序、安卓应用程序和网页上使用，为用户提供高效便捷的智能交互服务。基于OpenAI，NewBing和文心一言等大模型开发，ChatGPTAI实现了更准确的语义理解和全面的知识储备。用户与ChatGPTAI之间的沟通更加自然顺畅，解决问题更加高效准确。
        </p>
        <p>
          <el-button
            type="primary"
            size="mini"
            icon="el-icon-cloudy"
            plain
          >
            <a href="https://github.com/lessnkun/ChatGptWechat" target="_blank" style="text-decoration: none">
              源码链接
            </a>

          </el-button
          >
          <el-button
            size="mini"
            icon="el-icon-s-home"
            plain
          >
            <a href="https://github.com/lessnkun/ChatGptWechat" target="_blank" style="text-decoration: none">
              源码链接
            </a>
          </el-button>
        </p>
        <blockquote class="text-warning" style="font-size: 14px">
          <div>
            <el-link
              href="https://note.youdao.com/s/3vzev0xt"
              type="primary"
              target="_blank"
            >2.完整版生产安装教程（点击即可进入）
            </el-link>
          </div>
        </blockquote>
        <div style="margin-top: 20px;color: #000000"></div>

      </el-col>
    </el-row>
    <el-divider/>
    <el-row :gutter="20">
      <el-col :xs="24" :sm="24" :md="12" :lg="8">
        <el-card class="update-log">
          <div slot="header" class="clearfix">
            <span>联系信息</span>
          </div>
          <div class="body">
            <p>
           <div>
            <el-image src="https://i.postimg.cc/65fjvD0v/20230707224402.png"></el-image>
          </div>
            </p>
          </div>
        </el-card>
      </el-col>
      <el-col :xs="24" :sm="24" :md="12" :lg="8">
        <el-card class="update-log">
          <div slot="header" class="clearfix">
            <span>更新日志</span>
          </div>
          <el-collapse accordion>

            <el-collapse-item title="v4.0.0(大更新)">
              <ol>
                <li>角色选择</li>
                <li>优化UI和角色预设</li>
                <li>增加stream流式传输</li>
                <li>集成模型包括gpt3.5（初版大更新gpt4、绘画模型、文心一言等暂时没有加、下个版本就会加入）</li>
                <li>历史对话记录</li>
                <li>我的问询记录</li>
                <li>支持三方接口</li>
                <li>增加会员中心</li>
                <li>展示剩余次数</li>
                <li>增加次数和时间两种计费模式,后台可行性选择</li>
                <li>增加充值（目前仅支持充值卡重置）功能</li>
                <li>分享增加次数功能</li>
                <li>是否计费对话、是否计费询问、公告显示等均可自定义开关</li>
                <li>后台自行添加模型分类、模型名称、设定语等等</li>
                <li>所有后台配置页面化</li>
                <li>用户管理</li>
                <li>在线用户</li>
                <li>登录日志</li>
                <li>后台批量生成激活码</li>
                <li>激活码一键导出</li>
                <li>后台key统一管理</li>
                <li>key自动轮询</li>
                <li>使用的为微信官方违禁词，可自行添加违禁词</li>

              </ol>
            </el-collapse-item>

            <el-collapse-item title="v3.0.0">
              <ol>
                <li>问答类型,一问一答模式</li>
                <li>对话模式,和机器人消息对话</li>
                <li>对对话内容做了敏感词检测</li>
              </ol>
            </el-collapse-item>

        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import md5 from 'js-md5';
import axios from 'axios'
export default {
  name: 'Index',
  data() {
    return {
      // 版本号
      version: '4.0.0',
    }
  },
  created() {

  },
  methods: {
    goTarget(e){
      window.location.href=e;
    }
  }
}
</script>

<style scoped lang="scss">
.home {
  blockquote {
    padding: 10px 20px;
    margin: 0 0 20px;
    font-size: 17.5px;
    border-left: 5px solid #eee;
  }

  hr {
    margin-top: 20px;
    margin-bottom: 20px;
    border: 0;
    border-top: 1px solid #eee;
  }

  .col-item {
    margin-bottom: 20px;
  }

  ul {
    padding: 0;
    margin: 0;
  }

  font-family: "open sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 13px;
  color: #676a6c;
  overflow-x: hidden;

  ul {
    list-style-type: none;
  }

  h4 {
    margin-top: 0px;
  }

  h2 {
    margin-top: 10px;
    font-size: 26px;
    font-weight: 100;
  }

  p {
    margin-top: 10px;

    b {
      font-weight: 700;
    }
  }

  .update-log {
    ol {
      display: block;
      list-style-type: decimal;
      margin-block-start: 1em;
      margin-block-end: 1em;
      margin-inline-start: 0;
      margin-inline-end: 0;
      padding-inline-start: 40px;
    }
  }
}
</style>

