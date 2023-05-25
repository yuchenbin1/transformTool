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
        </div>
        <pre class="editor" id="editor2"></pre>
      </div>
    </div>
    <!-- <el-input class="input" v-model="fileInput" type="textarea" :rows="20"></el-input> -->
  </div>
</template>

<script>
import gcoord from 'gcoord'
import ace from 'ace-builds'
const fs = require('fs')
const remote = require('@electron/remote')
// const electron = require('electron')
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
    handleClickButton () {
      console.log(dialog, 'dialog')
      dialog.showOpenDialog({ properties: ['openFile'] }).then(res => {
        console.log(res)
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
          console.log(err, '读取文件内容失败')
        } else {
          console.log(result, 'result')
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
      const data = JSON.parse(JSON.stringify(geojson))
      try {
        gcoord.transform(data, gcoord.WGS84, gcoord.GCJ02)
        editor2.setValue(data)
      } catch (err) {
        dialog.showErrorBox('错误', '转化错误，请检查文件格式是否正确')
      }
    },
    handleClickTransform () {
      const value = editor1.getValue()
      console.log(value, 'value')
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
        filters: [{ name: '转化后的geojson', extensions: ['json'] }],
        message: editor2.getValue()
      }).then(res => {
        console.log(res, 'res')
        if (res && res.filePath) {
          try {
            fs.writeFileSync(res.filePath, editor2.getValue())
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
    }
  }
}
</style>
