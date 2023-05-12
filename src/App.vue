<template>
  <div>
    <button @click="isShow = true">select</button>
    <x-select v-model="isShow" :options="options"></x-select>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import XSelect from '@/components/select/XSelect.vue'

const isShow = ref(false)
const options = ref([
  {
    label: 'javascript',
    onSelect() {
      console.log('javascript')
    }
  },
  {
    label: 'java',
    onSelect() {
      console.log('java')
    }
  },
  {
    label: 'c++',
    onSelect() {
      console.log('c++')
    }
  },
  {
    label: 'golang',
    onSelect() {
      console.log('golang')
    }
  },
  {
    label: 'clang',
    onSelect() {
      console.log('clang')
    }
  },
  {
    label: 'python',
    onSelect() {
      console.log('python')
    }
  },
  {
    label: 'php',
    onSelect() {
      console.log('php')
    }
  },
  {
    label: 'c#',
    onSelect() {
      console.log('c#')
    }
  },
  {
    label: 'typescript',
    onSelect() {
      console.log('typescript')
    }
  },
  {
    label: 'shell',
    onSelect() {
      console.log('shell')
    }
  },
  {
    label: 'css',
    onSelect() {
      console.log('css')
    }
  },
  {
    label: 'html',
    onSelect() {
      console.log('html')
    }
  },
  {
    label: 'json',
    onSelect() {
      console.log('json')
    }
  },
  {
    label: 'jsx',
    onSelect() {
      console.log('jsx')
    }
  },
  {
    label: 'tsx',
    onSelect() {
      console.log('tsx')
    }
  },
  {
    label: 'git',
    onSelect() {
      console.log('git')
    }
  },
  {
    label: 'diff',
    onSelect() {
      console.log('diff')
    }
  },
  {
    label: 'lua',
    onSelect() {
      console.log('lua')
    }
  },
  {
    label: 'node',
    onSelect() {
      console.log('node')
    }
  },
  {
    label: 'ruby',
    onSelect() {
      console.log('ruby')
    }
  }
])
</script>

<style lang="scss" scoped></style>
