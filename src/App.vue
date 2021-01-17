<template>
  <div id="app">
    <el-row>
      <el-col :span="24">
        <h1>THBD OSS Media Cloud</h1>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <el-tree
            :props="defaultProps"
            :load="loadNode"
            node-key="path"
            lazy>
          <span class="custom-tree-node" slot-scope="{node}">
            <span>{{ node.label }}</span>
            <span v-if="node.isLeaf">
              <el-button type="text" @click="() => download(node)"> Download </el-button>
            </span>
          </span>
        </el-tree>
      </el-col>
    </el-row>
  </div>
</template>

<script>

import OSS from 'ali-oss'

const client = new OSS({
  region: 'oss-cn-beijing',
  accessKeyId: 'LTAI4G7k2W9gD1qpxWqs5iSw',
  accessKeySecret: '3NZvYcSpe9Z8hoCYkZRH9c5OGPQbpk',
  bucket: 'mayidance',
  secure: true
});

export default {
  name: 'App',
  data() {
    return {
      defaultProps: {
        label: 'name',
        isLeaf: 'leaf',
        id: 'id',
      }
    }
  },
  methods: {
    async loadNode(node, resolve) {
      if (node.level === 0) {
        return resolve([{
          name: 'OSS Root/',
          path: ''
        }]);
      }
      return resolve(await this.listDir(node.key))
    },
    async download(node) {
      window.open('https://mayidance.oss-cn-beijing.aliyuncs.com/' + node.key)
    },
    async listDir(dir) {
      let files = await client.list({
        prefix: dir,
        delimiter: '/'
      })
      let data = []
      if (files.prefixes) {
        files.prefixes.forEach(subDir => {
          if (subDir !== dir) {
            data.push({
              name: subDir,
              leaf: false,
              path: subDir
            })
          }
        })
      }
      if (files.objects) {
        files.objects.forEach(obj => {
          if (obj.name !== dir) {
            data.push({
              name: obj.name.split('/').pop(),
              leaf: true,
              path: obj.name
            })
          }
        })
      }
      return data
    }
  }
}

</script>

<style>

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}

</style>
