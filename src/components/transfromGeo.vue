<template>
  <div class="transformGeo">

    <div class="button-container">
      <div class="button" @click="handleClickButton">读取文件</div>
      <div class="button" @click="handleClickTransform">转化</div>
      <div class="button" @click="handleClickSave">下载</div>
      <div class="button" @click="handleClickClear">清空</div>
    </div>
    <div class="editor-container">
      <div class="pre-container">
        <div class="pre-title">当前选择的文件数据</div>
        <div class="select-container">
          <el-select v-model="value1" placeholder="请选择坐标系">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
          <div class="pre-button" @click="handleFormat(1)">json格式化</div>
        </div>
        <pre class="editor" id="editor1"></pre>
      </div>
      <div class="pre-container">
        <div class="pre-title">转化后的文件数据</div>
        <div class="select-container">
          <el-select v-model="value2" placeholder="请选择坐标系">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
          <div class="pre-button" @click="handleFormat(2)">json格式化</div>
        </div>
        <pre class="editor" id="editor2"></pre>
      </div>
    </div>
    <!-- <el-input class="input" v-model="fileInput" type="textarea" :rows="20"></el-input> -->
  </div>
</template>

<script>
// import gcoord from 'gcoord'
const gcoord = require('gcoord').default
// import ace from 'ace-builds'
const ace = require('ace-builds').default
const fs = require('fs')
const remote = require('@electron/remote')
const { dialog } = remote

let editor1 = null
let editor2 = null

export default {
  data () {
    return {
      value1: '',
      value2: '',
      options: [
        {
          label: 'WGS-84坐标系',
          value: 'WGS84'
        },
        {
          label: 'GCJ-02坐标系',
          value: 'GCJ02'
        },
        {
          label: 'BD-09坐标系',
          value: 'BD09'
        },
        {
          label: 'BD-09米制坐标',
          value: 'BD09MC'
        },
        {
          label: '墨卡托投影(EPSG3857)',
          value: 'WebMercator'
        }
      ]
    }
  },
  mounted () {
    editor1 = ace.edit('editor1')
    editor2 = ace.edit('editor2')
  },
  methods: {
    handleFormat (num) {
      if (num === 1 && editor1.getValue()) {
        editor1.setValue(JSON.stringify(JSON.parse(editor1.getValue()), null, 2))
      } else if (num === 2 && editor2.getValue()) {
        editor2.setValue(JSON.stringify(JSON.parse(editor2.getValue()), null, 2))
      }
    },
    handleClickButton () {
      dialog.showOpenDialog({ properties: ['openFile'] }).then(res => {
        // 读取的文件路径赋值
        // filePath.value = res.filePaths[0];
        if (res && res.filePaths[0]) {
          this.readFileContent(res.filePaths[0])
        } else {
          dialog.showErrorBox('错误', '未获取文件')
        }
      })
    },
    readFileContent (filePath) {
      fs.readFile(filePath, { encoding: 'utf-8' }, (err, result) => {
        if (err) {
          dialog.showErrorBox(err, '读取文件内容失败')
        } else {
          editor1.setValue(result)
          // this.transform(result)
          // this.fileContent.value = result;
        }
      })
    },
    transform (geojson) {
      if (!this.value1) {
        dialog.showMessageBox({
          title: '请选择当前文件坐标系',
          message: '请选择当前文件坐标系'
        })
        return
      } else if (!this.value2) {
        dialog.showMessageBox({
          title: '请选择转换成文件的坐标系',
          message: '请选择转换成文件的坐标系'
        })
        return
      }
      try {
        const data = JSON.parse(geojson.toString())
        console.log(data, 'data')
        gcoord.transform(data, gcoord[this.value1], gcoord[this.value2])
        console.log(data, 'data')

        console.log(JSON.stringify(data))

        editor2.setValue(JSON.stringify(data))
      } catch (err) {
        dialog.showErrorBox('错误', '转化错误，请检查文件格式是否正确')
      }
    },
    handleClickTransform () {
      const value = editor1.getValue()
      this.transform(value)
    },
    handleClickClear () {
      editor1.setValue('')
      editor2.setValue('')
    },
    handleClickSave () {
      dialog.showSaveDialog({
        title: '保存转化后的文件',
        properties: ['showHiddenFiles'],
        filters: [{ name: 'geojson', extensions: ['json'] }],
        message: editor2.getValue()
      }).then(res => {
        if (res && res.filePath) {
          try {
            fs.writeFileSync(res.filePath, editor2.getValue())
            dialog.showMessageBox({
              title: '下载成功',
              message: '下载成功'
            })
          } catch (err) {
            console.error(err)
          }
        } else {
          dialog.showErrorBox('错误', '保存路径获取失败')
        }
      })
    }
  }
}
</script>

<style lang="scss">
.transformGeo {
  .button-container{
    display: flex;
    justify-content: center;
    align-items: center;
    .button {
      width:80px;
      height:30px;
      line-height: 30px;
      margin: 0 10px;
      border:1px solid #CCC;
      background: #CCC;
      text-align: center;
      border-radius: 4px;
      cursor: pointer;
    }
  }
  .editor-container{
    display: flex;
    justify-content: center;
    margin:0 20px;
    .pre-container{
      .pre-title{
        font-size:18px;
      }
      .select-container{
        margin:10px 0;
      }
      .editor{
        width:500px;
        height:600px;
      }
      .pre-button{
        width: 100px;
        height:30px;
        line-height:30px;
        border-radius: 4px;
        border:1px solid #ccc;
        margin:10px auto;
        cursor: pointer;
      }
    }
  }
}
</style>
