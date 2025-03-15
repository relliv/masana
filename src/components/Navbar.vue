<template>
  <div class="navbar">
    <h1>Workspace: {{ workspaceName }}</h1>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import fs from 'fs';
import path from 'path';

const workspaceName = ref('');

onMounted(() => {
  const workspacePath = fs.readFileSync(path.join(window.electronAPI.getPath('userData'), 'workspacePath.txt'), 'utf-8');
  workspaceName.value = path.basename(workspacePath);
});
</script>

<style scoped>
.navbar {
  width: 100%;
  background-color: #333;
  color: white;
  padding: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.navbar h1 {
  font-size: 18px;
}
</style>
