<template>
  <div class="chat_content">
    <div class="slider">
      <div class="header">
        <div class="title">
          ChatGPTAI智能助手
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import md5 from 'js-md5';
import axios from 'axios'
export default {
  name: 'Chat',
  data() {
    return {

    }
  },
  created() {

  },
  methods: {
  }
}
</script>

<style>
:root {
  --white: #fff;
  --black: #303030;
  --gray: #fafafa;
  --primary: #1d93ab;
  --second: #e7f8ff;
  --hover-color: #f3f3f3;
  --bar-color: rgba(0,0,0,.1);
  --theme-color: var(--gray);
  --shadow: 50px 50px 100px 10px rgba(0,0,0,.1);
  --card-shadow: 0px 2px 4px 0px rgba(0,0,0,.05);
  --border-in-light: 1px solid #dedede;
  --window-width: 90vw;
  --window-height: 90vh;
  --sidebar-width: 300px;
  --window-content-width: calc(100% - var(--sidebar-width));
  --message-max-width: 80%;
  --full-height: 100%;
}
.chat_content{
  border: var(--border-in-light);
  border-radius: 20px;
  box-shadow: var(--shadow);
  color: var(--black);
  background-color: var(--white);
  min-width: 600px;
  min-height: 480px;
  max-width: 900px;
  display: flex;
  overflow: hidden;
  box-sizing: border-box;
  width: var(--window-width);
  height: var(--window-height);
}
.chat_content .slider{
  top: 0;
  width: var(--sidebar-width);
  box-sizing: border-box;
  padding: 20px;
  background-color: var(--second);
  display: flex;
  flex-direction: column;
  box-shadow: inset -2px 0 2px 0 rgb(0 0 0 / 5%);
}

.chat_content .slider .header {
  position: relative;
  padding-top: 20px;
  padding-bottom: 20px;
}
.slider .header  .title{
  font-size: 20px;
  font-weight: 700;
}
</style>

